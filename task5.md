# Task 5

Now that the encrypted filesystem is open, I mounted it in QEMU to check it out. Given that the theme is RE, I decided to close QEMU and mount `part.enc` on my server, to copy the `agent` and `dropper` executable onto a shared folder to use Ghidra.

I spent ages upon ages (a total of probably 10 hours) working on trying to understand and decipher the foreign language that Ghidra was presenting me, trying to decompile the two seemingly most important exes. Many hours were spent staring at and wandering the screen, to no avail. There was no way I was doing this correctly.

And so, I took a step back. I left Ghidra, and went back to my QEMU roots. I decided to play around with the `dropper` file a bit more - turns out, if you pass it an invalid file as an argument, it loads (what I assume are) defaults and tries to connect to the server and process the files. I found this out while trying to RE via GDB (had to set up an ARM64 Ubuntu VM in UTM for this...). Funnily enough, since it's able to run with invalid arguments, it's also able to print out error messages. And boy, print it did.

`1970/01/01 06:50:13 Failed to insert into MongoDB:server selection error: server selection timeout, current topology: { Type: Unknown, Servers: [{ Addr: 100.107.209.17:27017, Type: Unknown, Last error: dial tcp 100.107.209.17:27017: connect: network is unreachable }, ] }`

And so, we have our IP.