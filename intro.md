
With ==Make== you can establish a library of shortcuts for frequently used commands. Names of such shortcuts represent their semantics, not implementation:

`make deploy`, `make logs`, `make feature resubscription`

Makefile is stored in the project repository and becomes a live development workflow documentation shared between the whole team.

- Hey, come on! Make is some ancient tool from the past! 

- I remember I used it to compile some libraries from sources two years ago? It can be used for something else?

- Hold on, I am not a C/C++ developer to compile stuff. I already use npm/rake/maven. Why should I care about one more tool?

This is how I thought too. The reality is that:

1. ==Make== plays the role of the universal glue between many technologies we have to use in the modern (web-)development.
2. It is not a replacement for your native tools for building and compiling stuff. 
3. It is a way to take out all the shell scripts from your yarn/rake/maven because they look unnatural there and cause a lot of overhead

Less than 25% of developers use Makefile as a library of shortcuts:

[pic]

If your one of the rest 75%, you're about to learn the approach, which can save you a huge amount of time in the future.

Make is good, but its official documentation is too detailed and only readable if you're sysadmin thinking in a mix of C and shell scripts. It contains way too many details - way too far from the needs of a modern developer.

Because of that, I created the mini-reference describing only features useful for modern developers.

7500 lines → 500 lines

But you don't need to read them.

How so? What should I do then? 

1. To understand the main of modern Make usage, watch this 5 minutes video.
2. If you want to try it, download the Make mini-reference for modern developers.
3. If you want to organize the whole development workflow using Make, then an extended version of the reference with examples and advanced tricks is what you need.
