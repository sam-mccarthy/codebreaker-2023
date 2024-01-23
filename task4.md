This task was, thankfully, really straightforward. I went into QEMU and looked through the file system, eventually running into `/opt`, where `mount_part` (a bash script for mounting LUKS filesystems via cryptsetup) and `part.enc` (a LUKS filesystem file) are located. I looked at `mount_part`, hacked away at cryptsetup for a bit, and eventually extracted the challenge files on my desktop to chip at `part.enc` using hashcat.

After learning a bit of how hashcat is used, and playing with the settings (mostly trying to squeeze performance, since it seemed slow), I let it run overnight. To my dismay, it didn't pop, and didn't seem like it was going to.

After thinking on it for a bit, I decided to check `mount_part` again, in case I missed something- and boy did I miss something. I had been using a standard, direct bruteforce method, when the password needed was *clearly* in SHA1 format:

```bash
DATA="${NAME}${ID:0:3}"

...

echo -n $DATA | openssl sha1 | awk '{print $NF}' | cryptsetup open $ENC_PARTITION part
```

After finding out that the password was a SHA1 encoded string consisting of a hostname and a 0-3 digit ID, I had another idea in my toolbox. I wrote a program in C# to generate a list of hashes, which I aptly named `rockme.txt`, and threw that into hashcat instead.

Unfortunately, that didn't pop either- but I quickly realized that I should be including alphabetical characters as well, so I generated another file and threw that into hashcat. I checked `results.txt`, and immediately threw the hash into Codebreaker. It worked.

It turned out to be the lowercase SHA1 hash of `spicybin09a`.

Here's the hashcat command I finished with and the program I wrote to generate the list:
```bash
./hashcat.exe -m 14600 -a 0 -w 4 -O -o results.txt part.dd rockme.txt
```

```csharp
using System.Collections.Concurrent;
using System.Security.Cryptography;
using System.Text;

string ComputeSHA1(string input)
{
    using var sha1 = SHA1.Create();
    var bytes = Encoding.UTF8.GetBytes(input);
    var hash = sha1.ComputeHash(bytes);
    
    return Convert.ToHexString(hash);
}

string chrs = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
Console.WriteLine("Beginning hash process");

ConcurrentQueue<string> hashes = new ConcurrentQueue<string>();
Parallel.For(0, chrs.Length, (x) =>
{
    string regular = $"spicybin{chrs[x]}";
    string rHash = ComputeSHA1(regular);
    
    hashes.Enqueue(rHash);
    hashes.Enqueue(rHash.ToLower());
    for (int i = 0; i < chrs.Length; i++)
    {
        regular = $"spicybin{chrs[x]}{chrs[i]}";
        rHash = ComputeSHA1(regular);
    
        hashes.Enqueue(rHash);
        hashes.Enqueue(rHash.ToLower());
        for (int j = 0; j < chrs.Length; j++)
        {
            regular = $"spicybin{chrs[x]}{chrs[i]}{chrs[j]}";
            rHash = ComputeSHA1(regular);
    
            hashes.Enqueue(rHash);
            hashes.Enqueue(rHash.ToLower());
        }
    }
});

Console.WriteLine("Hashing finished. Writing");

using(FileStream fs = new FileStream("rockme.txt", FileMode.OpenOrCreate, FileAccess.Write))
using (StreamWriter sw = new StreamWriter(fs))
{
    string? hash;
    while (!hashes.IsEmpty)
    {
        while (!hashes.TryDequeue(out hash)) ;
        sw.WriteLine(hash);
    }
}```