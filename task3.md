# Task 3

Boy, was this one a doozy. Setting up Docker and QEMU both weren't really an issue: I had already had Docker experience and a full Docker setup on my home server, so I just used the existing daemon alongside my other containers. QEMU was straightforward since the README.md detailed the commands needed.

Where things got rough was finding the actual decryption key. It took me longer than I'd like to admit to find the thing, but once I found the keyaddr environment variable and printed out the memory, I was golden- or so I thought. The key that printed out using `md 467a1000` (don't quote me on the address) wasn't accepted as an answer- correct format, wrong key. After ages of wracking my brain, looking for a different key, trying it in Ghidra, etc., I caved in and submitted a help request:

> I feel like I've explored everything in U-boot. I've gone through each command, I've seen the keyaddr environment variable, I've found and attempted to submit the key in memory (correct format wrong key), I've looked into the device_tree file, edited that file, looked through different memory addresses of interest. I just don't know where to go from here, it's been a good few days stuck on this now.

I went to sleep, feeling defeated, and woke up to this wonderful message (shoutout NSA Danny):

> When dumping memory, think about how bytes are laid out in memory.

Holy smokes. After reading the message I knew exactly what the issue was. I wrote out the same key in a reversed byte order, only to find that that, too, was wrong. Turns out that the order of 4 byte chunks was correct, but the bytes within those chunks was in the wrong order. I realized this after figuring out the documentation of `md - memory display`. The documentation states something along the lines of `md [.b .w ...`. I didn't realize that the space wasn't supposed to be there, and I was supposed to be running `md.b`. Wow.

I ran `md.b` with the value of the `keyaddr` environment variable, got rid of the spaces inbetween bytes, and submitted. Lo and behold, success.