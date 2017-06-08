# Sometimes, Rewriting is a Good Idea
#### Posted June 6th, 15:47 PM

### Background
There are plenty enough horror-stories being told about rewriting software that I feel compelled to share a recent success from personal experience. This particular rewrite raised the odds further by switching language from Golang to the vastly more complex monster that is modern C++ for a 7-kloc networked, encrypted application with a custom database. On the other hand, the project is still very young and you might well consider the previous version an ext-/pensive prototype. The rewrite may be found [here](https://github.com/andreas-gone-wild/snackis), while the previous implementation has found a new home [here](https://github.com/andreas-gone-wild/snackis-golang).

![enigma](images/enigma.jpg?raw=true)

### The Project
[Snackis](https://github.com/andreas-gone-wild/snackis) is aiming for something that might be described as a post-modern enigma-device; a simple and convenient one-stop shop for most secure communication needs. It provides a curses-based UI on top of [libsodium](https://github.com/jedisct1/libsodium) and supports using any regular email-account as transport. Peers, encryption keys, history and settings are stored locally in an encrypted database. Non-UI functionality is exposed as a static library for easy reuse. At present, Snackis supports key-exchange via invites/accepts and encryption/decryption of arbitrary data for a specific peer. Coming up around the corner is threaded group-conversations; followed by events, wikis, shared resource scheduling and more built on the same platform.

### Motivation
The main motivator for me was the overall lack of stability because of Golangs forced, opaque M*N threading strategy in combination with C-extensions; even the relatively simple case of sending and fetching email while dealing with a curses-UI was enough to trigger Heisenbugs. The best explanation I can come up with is that most Golang-software doesn't use C-extensions. Nothings sets my mind searching for alternative solutions like Heisenbugs hiding in core language features.

### Team
I am mostly a one-man team these days, which means that no effort was wasted on cat-herding. Pulling a trick like this is probably best avoided in large teams because of the overhead and lack of integrity; on the other hand, reaching such a decision in a democratic process is highly unlikely. I have been coding C++ since 1996, so I know my way around; and I have several years of experience from Golang; switching to an unknown language is a different story for another day.

### Time
Time is difficult to quantify when you're unemployed and have a 1-yo child running around your legs for most of the day. I've been working on the design for quite some time, which means that many decisions were more or less settled before I started implementing in Golang. I spent around 3 months in Golang; to date, I've spent around one third of that on rewriting. While there are still bits and pieces left to deal with to reach full feature parity; the new version is significantly improved in every other aspect.

### Conclusion
The current version clocks in at 5 kloc, 2 kloc less than the original. While C++ is without a doubt more rigorous and complex; just getting rid of Golang's in-your-face, paleo-style error-handling was a major win. Add generics and extensive standard-library support for collections/algorithms/more, and that's even more code that I didn't have to write. From my perspective, C++ is clearly a superior language for dealing with software this complex, given enough experience with the language; but the only proof I have to offer is what you see. C++ feels tighter, it lets me move forward faster with confidence; whereas Golang felt increasingly squishy, spongy and unreliable as the project grew in depth.

Until next time; be well,<br/>
A