A Theoretical Guide to Code Reuse
=================================

Is cross-platform development even possible?

It depends on how you define "cross-platform". We need to break it down. There exist several approaches that are predominant today.

### 1. Write once, run anywhere ###

This kind of portability is available today, but only to some extent. "Anywhere" really only means multiple operating systems within one category of computing devices, e.g. the 3 major desktop OSes or all mobile OSes. I can only name game development as one truly cross-platform industry to which this approach is the most applicable.

A few concrete examples in this category:

* [Titanium](http://www.appcelerator.com/titanium/) – a framework and development environment that uses JavaScript and targets _iOS, Android, and BlackBerry, as well as hybrid and HTML5_. It is free and open source, with optional paid services.
* [Xamarin](http://xamarin.com/tour) – a platform based on C# and Mono that lets you target multiple mobile and desktop OSes with single code base. Provides free price tear with limits on the app size.
* A multitude of free and paid game frameworks: Corona SDK (Lua, 2D, mobile), [Moai](http://getmoai.com/wiki/index.php?title=Moai_Hosts) (Lua, open-source, mobile and desktop), [Haxe](http://haxe.org/) (open source), [Monkey](http://www.monkeycoder.co.nz/Monkey/about.php) (proprietary, mobile and desktop), [Marmalade](http://www.madewithmarmalade.com/marmaladesdk/supported-platforms) (C++, mobile, desktop, and smart tv).
* C++ or any dynamic language + Qt/GTK/wxWidgets combo provides cross-platform development across the three major desktop OSes: GNU Linux, OS X, Windows. Neither is objectively better than the other one, so the choice here is mostly driven by preference.

Programming languages that are used in this approach are: C++, Java, C#, JavaScript, Lua. Haxe and Monkey are programming languages of their own that are transpiled to other languages before running.

### 2. Using one language on all target platforms ###

Imagine writing a mobile app in C++. You'll be able to share most of the program's logic between platforms (in some cases this would even include local data store, network code, model code for your views, etc.), but you'll have to write a separate GUI code for each one. Even if you decide to use C++ for GUI, you will need a separate wrapper for whatever language is used on each respective platform (which is a tough task to even try to implement).

Language choices in this category are similar to the previous one. You generally want to go with this approach to cross-platformability if you want to have more flexibility and don't like to be locked into any particular framework.

Technologies of interest:

* [emscripten](https://github.com/kripken/emscripten) let's you compile C++ code base to JavaScript, enabling access to the browser for games and other kinds of software. I would not choose it as a base technology for building new stuff though unless I had a sizeable and mature collection of libraries (and an experienced team of C++ fans) to speed up the development process.
* JavaScript – apart from being the "language of the web", it is also used as a scripting language by some cross-platform frameworks. With [Node.js](http://nodejs.org/) it is also possible to build server-side software.
* Ruby – one of the popular dynamic languages widely used in the web development community. With things like [MobiRuby](http://mobiruby.org/) it becomes possible to target mobile devices as well, but it's still under development.

### 3. One language/framework for the server, another one for all clients ###

_(when the division into server and client parts is applicable)_

A prevalent approach to developing web-based apps today because, strictly speaking, the server (backend) and the client (browser, desktop, or mobile) are different kinds of apps that are usually developed by separate teams.

But in practice, some code related to the communication infrastructure or even parts of business logic could be shared between the server and the clients. And so it becomes tempting to use one language for both parties in this case (see previous section for examples).


## Are you choosing a technology based on product needs? ##

If you choose one language to master, then you kind of have to use it for any kind of software development you do. So the task of picking the right technology will boil down to finding a cross-platform framework and possibly an implementation of the language for all the platforms you'd like to support.

But what if we look at this from a different angle. In our trade, there are trade-offs everywhere. So to make an informed decision about any of the numerous existing languages and frameworks out there, we need to see what kind of technologies make more sense for our product (or for our problem domain, if you like to think in broader terms).

With that said, it is important to first answer the question "what are the goals that I'm pursuing in building my next product?" before seeking a solution for the problem of finding a language that would target the most platforms. There is no silver bullet, so I would suggest keeping the focus on the product and expanding to more platforms as time and resources permit. In the end, no matter which technology you choose, the success of your product will mostly depend on the qualities and experience of your developer team.


## A tough choice ##

The idea for writing this post was inspired by this [tweet](https://twitter.com/nivertech/status/381812150668230657):

> **@nivertech**
>
> Is there a programming language, which can target (via different profiles):
> - native (x86, ARM x 32/64)
> - JS (+ asm.js)
> - JVM
> - EVM/BEAM
>
> ?

I find it difficult to imagine a language that could produce equally efficient JVM byte code and BEAM code at the same time. It would have to be a functional languages (or something with strong functional influence) and built-in concurrency model similar to what BEAM provides.

But more importantly than finding such language (which does not exist today) is the question of "what are your goals?" Perhaps, there is another solution to achieve them if you take some time to analyze what it is exactly that you won't build and what of audiences you're trying to reach. If you simply want to choose a single language for life, I personally would not bet on that being possible. Each language has its own inherent properties that make it more suitable for certain problem domains or certain styles of development.

Choose Lua if you want to have the widest platform coverage possible and if you don't mind writing missing libraries for your problem domain. It is simple but potent language, it is a popular choice for game frameworks, it can run [in the browser](http://kripken.github.io/lua.vm.js/lua.vm.js.html), it can even run JavaScript itself (see the question "What's running JavaScript under the hood?" on [this page](http://www.dragoninnovation.com/projects/22-tessel)) and [JVM](http://cowlark.com/luje/doc/stable/doc/index.wiki) :).

Choose C/C++ for the same reasons if you already have a large code base written in it.

Choose JavaScript if you don't mind being locked into the browser or one of the available cross-platform frameworks that support it.

Choosing any other language will get you through 90% of software development needs in the whole industry. The remaining 10% can be covered by writing separate modules or subsystems in specialized languages/toolkits.


---
Tags: cross-platform, framework, proglang

Date: Mon Oct  7 14:28:03 EEST 2013
