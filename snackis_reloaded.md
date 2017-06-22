# Snackis Reloaded
#### Posted Jun 22nd, 22:00 PM

### Snackis
[Snackis](https://github.com/andreas-gone-wild/snackis) is aiming for something that might be described as a post-modern enigma-device; a tool that covers most secure communication needs using any regular email-account as transport. Peers, encryption keys, messages and and settings are stored locally, encrypted using a master password. At present; Snackis supports key-exchange via invites/accepts, encryption/decryption of arbitrary data for specific peers and feeds/posts.

### Language Issues
The first bump in the road since [last post](https://github.com/andreas-gone-wild/blog/blob/master/introducing_snackis.md) was stability issues in the Go-stack that I had assembled for the project. In the end, I decided that Snackis would be better off without a complex language-runtime; and [ported](https://github.com/andreas-gone-wild/blog/blob/master/rewriting_good_idea.md) the entire codebase to C++.

### New UI
Next up was the user interface. I first decided to go with NCurses for an initial UI, leaving more energy to deal with core functionality.  I am old enough to fondly remember the instant interfaces of yesterday, the kind where the computer was mostly waiting for you; and a text-based interface seemed like the right approach to get there. This was my deepest dive into NCurses yet, what finally pushed me over the edge was realizing I would have to implement my own buffering to gain support for scrolling. Tempting, but not thanks. It boggles the mind that this is still the apex of open/kind of-portable solutions for text-interfaces. So I went back to the drawing-board and somehow came upon the idea of implementing the same UI in GTK+, changing as little as possible. And it came out pretty well if you ask me. The result feels somewhat like a cross between a Lisp REPL and an iPhone, hacker-friendly yet still simple. Of all the UI toolkits I've used over the years, GTK+ definitely offers one of the most pleasant rides. People get all worked up over the object-oriented C-approach; but I've found that it's trivial to step out of the box and create the abstractions I need without hooking into the object-system, which turns it into a non-issue. And it support styling via CSS these days, which reduces the amount of boring code further.

![setup example](images/setup.png?raw=true)

### Beating Email
The new UI provided enough support to finally cross the line where Snackis does something other than encryption better than regular email. Keeping track of and searching for discussions and/or individual posts is now much more convenient than any email client could hope to offer without providing the same kind of structural support. In my mind, that's where most solutions for encrypting email loose their way; in implementing full specifications rather than solving specific problems; resulting in over-engineered solutions with steep learning curves and complex interfaces.

![post example](images/post.png?raw=true)

### What's Next
Once the code settles down a bit from all the drama lately, I'm planning on implementing signed reciepts for the next iteration. Always knowing for sure if the person in the other end has received a message or not blurries the line between email and chat enough to make Snackis usable in more real-time scenarios. 

![feed search example](images/feed_search.png?raw=true)

Until next time; be well,<br/>
A