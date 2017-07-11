# The Mkayloc Methology
#### Posted Jul 11th, 23:36 PM

I've been writing software for 32 years, 15 years professionally. During that time I've seen plenty of methologies fail to deliver on their promises. It turns out that herding cats is tricky business. It's not that most of us don't want to cooperate, or write good software; but rather that the creative process is chaotic by definition. Which means that any attempt to nail it down will inevitably turn into an obstacle sooner or later. The extreme and agile movements got some things right, but they all fell into the trap of dictating the process; despite the fact that their pioneers would hate someone dictating theirs as much as anyone.<br/><br/>

Lately, I've mostly been working by myself on my magnum opus, [Snackis](https://github.com/andreas-gone-wild/snackis); a 3-month old, 7kloc C++ codebase with a custom encrypted DB, GUI and network layer. Since I'm a one man shop, I can't afford to drag duplicated or unused code around; which means that I have to be very disciplined about not piling on features or skipping on cleaning up and compressing existing code. To help me stay focused and prevent side-tracking too far, without killing the creative spirit; I've decided to impose a fixed kloc-limit.<br/><br/>

While seemingly naive, this simple decision has had a decidedly positive impact on my design process and code quality. I will allow the code to grow while exploring new features, and then spend as much time and effort as needed on compressing and organizing existing code to get back below the limit. Sometimes the way to get there is to remove a feature that isn't pulling it's weight, at other times several features may be refactored into a more effective abstraction. Technical debt that usually get shuffled to the end of the priority list is now a non-optional, challenging part of the design process for new features.<br/><br/>

It is said that a picture is worth more than a thousand words. The [graph](https://github.com/andreas-gone-wild/snackis/graphs/code-frequency) for [Snackis](https://github.com/andreas-gone-wild/snackis) clearly shows what a living, breathing code base looks like.<br/><br/>

![graph](images/mkayloc.png?raw=true)

Until next time; be well,<br/>
A