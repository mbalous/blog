# Sometimes, Rewriting is a Good Idea
#### Posted June 6th, 15:47 PM

![enigma](images/enigma.jpg?raw=true)

### Background
There are plenty enough horror-tales being told about rewriting software that I feel compelled to share a recent success-story from personal experience. This particular rewrite raised the odds further by switching language from Golang to the vastly more complex monster that is modern C++ for a 7-kloc networked, encrypted application with a custom database. On the other hand, the project is still very young and you might very well consider the previous version an ext-/pensive prototype. The rewrite may be found [here](https://github.com/andreas-gone-wild/snackis), while the previous implementation has found a new home [here](https://github.com/andreas-gone-wild/snackis-golang).

### The Project
Snackis is aiming for something that might be described as a post-modern enigma-device; a simple but convenient one-stop shop for all your secure communication needs. It provides a curses-based UI on top of [libsodium](https://github.com/jedisct1/libsodium) and supports using any regular email-account as transport. Peers, encryption keys, history and settings are stored locally in an encrypted database. Non-UI functionality is exposed as a static library for easy reuse. At present, Snackis supports key-exchange via invites/accepts and encryption/decryption of arbitrary data for a specific peer. Coming up around the corner is threaded group-conversations; followed by events, wikis, shared resource scheduling and more built on the same platform.

### Time
Time is difficult to quantify when you're unemployed and have a 1-yo child running around your legs for most of the day. I've been working on the design for quite some time, which means that many decisions were more or less settled even xbefore the Golang-implementation. I spent around 3 months on the Golang-implementation. To date, I've spent around one third on rewriting. While there are still bits and pieces left to deal with; the new version is significantly improved in every other aspect.

### Team
I am mostly a one-man team these days, working full-time; which means that no effort was wasted on herding cats and spreading information. Pulling a trick like this is probably best avoided in large teams because of the overhead and lack of integrity. I have been coding C++ since 1996, so I know my way around; and I have several years of experience from Golang; switching to an unknown language is a different story for another day.

### Result
The current version clocks in at 5 kloc, 2 kloc less than the original. While C++ is without a doubt more rigorous and complex, just getting rid of Golang's in-your-face paleo-flavored error-handling was a major win. Add generics and first-class standard-library support for collections/algorithms and more, and that's even more code that I didn't have to write. The Golang-executable is 6MB and requires shared libraries for the curses-UI, while a non-debug C++-executable is 1MB with everything statically linked. From my perspective, C++ is clearly a superior language for this kind of software given enough experience; but the only proof I have to offer is what you see.

Until next time; be well,<br/>
A