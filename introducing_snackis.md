# Introducing Snackis
#### Posted May 9th, 2:50 PM

![invite email example](images/invite_email.png?raw=true)

### Snackis
[Snackis](https://github.com/andreas-gone-wild/snackis) is the latest and by far most substantial iteration of an idea I've been working on in one form or other over the last 30 years. A distributed and secure social network that uses existing email infrastructure as transport, cuts enough corners to make the experience seemless and adds more elaborate abstractions on top. [Snackis](https://github.com/andreas-gone-wild/snackis) stores all data locally in encrypted form and comes with a convenient text-based interface and first-class support for offline use. It is completely self-contained and may be put on a USB-stick for instant secure messaging wherever you are. All functionality offered by the UI is also available through the Golang API.

![post example](images/post.png?raw=true)

### Status
This is intended as a preview; the functionality needed for a minimum viable program is operational, but plenty of testing remains before I'm willing to commit to any kind of compatibility. Minimum viable in this context means support for invites and group-/private discussions. [Snackis](https://github.com/andreas-gone-wild/snackis) uses Python3's mature email libraries and great care has been taken to not interfere with the normal email flow; all messages sent from [Snackis](https://github.com/andreas-gone-wild/snackis) are tagged with ```___SNACKIS___``` and processed messages are moved to a folder with the same name to keep the normal inbox tidy. You may find preliminary documentation [here](https://github.com/andreas-gone-wild/snackis).

### Roadmap
The code needs some time to settle down after a month-long mad rush to get basic functionality in place. I have a few ideas that have been waiting for this kind of platform; distributed scheduling of shared resources to name one. What I can promise will never happen is monetization strategies, behavioural profiling, ads, binary blobs or any other kind of intrusions into your personal integrity. This one is by the people, for the people; and if you're anything like me, you're ready for something new.

### Support
If you have a few coins to spare, [this is your chance](https://www.paypal.me/c4life) to help Snackis move forward faster. $500/month would allow me to work full-time on the project. Thank you.

Until next time; be well,<br/>
A