# Task 6

After reading the description of the challenge, I knew that the first hurdle would be figuring out how exactly using an SSH jumpbox works. Through some research, I found that the goal is to setup tunneling through SSH to allow connections to be forwarded through the jump host - so I found a source on [how to do that](https://www.ssh.com/academy/ssh/tunneling-example) and got that running.

`ssh -i ~/.ssh/jumpbox.key -L 27017:100.107.209.17:27017 user@external-support.bluehorizonmobile.com -N`

I knew that MongoDB was involved through reading the CBC Discord in between challenges, so I made a quick search of `connect to mongo database over ssh tunnel` and discovered the holy grail. [A guide on how to connect to a Mongo database over ssh tunneling.](https://gist.github.com/umidjons/ec7a60415c452787f2666ef6761bbf69) Who'd have thunk?

### This was unfortunately the end for me, as finals season was coming in and I had to focus on that, with a family trip around the corner. The tasks were a great fun and I'm thrilled to take another hack at Codebreaker next year!