A Month of Lisp
===============

This is the first programming language I'm going to familiarize myself with as
part of my programming language literacy series. If you've missed my
introductory post on this topic, you might want to read it first by following
this [link][1].

  [1]: https://github.com/alco/blog/blob/master/posts/2012-03-01-prog-lang-literacy.md

## Short History ##

Lisp is a very interesting language with rich history and a long-living
community. Ever since I have first encountered it (which was during my days at
the university, they had an XLISP implementation installed) I couldn't ever
forget it and occasionally used it write some basic programs. Conceived by John
McCarthy in 1958, Lisp was the second high-level programming language that is
still in common use today (preceded by Fortran). It was the first dynamic
language and the first language to implement the concept of automatic memory
management known as garbage collection. The [Wikipedia article][2] says that it
has also pioneered tree data structures and the self-hosting compiler among
other things.

A number of features are unique to Lisp and have been for a long time since its
inception. Although it has never enjoyed such widespread adoption as modern
mainstream languages like C, C++, Java, C#, Python, Ruby, etc., it has always
had a solid user base comprised of hackers, researchers, enthusiasts, and
industry professionals.

Here's a short list of some of the most prominent features of Lisp.

* **Lisp is a homoiconic language**. Both code and data are represented in the
  same way in Lisp. This simple concept opens possibilities to do things that
  other languages would have hard times implementing (if at all): meta-circular
  evaluator, powerful macro system, embedded languages, programs that write
  other programs, etc.

* **First-class closures**. Not unique to Lisp these days, this one is a
  fundamental building block of Lisp. In fact, with closures alone one could
  build the _cons cell_ data type which in turn would make it possible to build
  lists and trees, thus making them basically behave like built-in list and
  tree types.

* **Dynamic runtime environment**. Every modern dynamic language has a REPL,
  but it seems to me that it is mostly used for the purposes of learning and
  quick hacking.  Lisp was the first language to come with a REPL: you could
  launch your program and modify its behavior at runtime with ease. Unheard of!

  [2]: http://en.wikipedia.org/wiki/Lisp_(programming_language)

## Family of Languages ##

Lisp is not a single language, it is a family of languages. Over the course of
its history literally dozens of dialects and still more different implementations
have been developed, many of which have found their place in certain fields of
computing. Among the most prominent ones (which I've heard of) are Scheme,
Common Lisp, Emacs Lisp, plus a few younger ones like Clojure (2007), Nu
(2007), and Arc (2008). Also, if I recall correctly, Gimp uses a Scheme dialect
as its scripting language (which some of its users propose to replace with
Python :)).


## Resources ##

This last section of the post contains a list of online resources which I'm
going to be using actively over the course of this month. I hope you have found
this post useful. Let me know if you'd like to add or correct anything. Let us
have an enjoyable and productive month this March. Good luck and have a nice
day!

* History of Lisp
    ([link](http://www-formal.stanford.edu/jmc/history/lisp/lisp.html))

* Structure and Interpretation of Computer Programs
    ([book](http://mitpress.mit.edu/sicp/full-text/book/book.html) and
     [video lectures](http://groups.csail.mit.edu/mac/classes/6.001/abelson-sussman-lectures/))
  This one is a fundamental work on programming in general. I highly recommend
  at least watching the lectures, they provide many bright ideas and insights
  into the theory of software design and the ways of solving practical
  problems. Don't be pushed back by the awkward Scheme syntax, look at what's
  really important — the content. Also, don't be deceived by the book's
  publishing date. As modern languages slowly adopt features that Lisp had
  since its inception (for more than half a century!), many of the problems
  presented in the book remain relevant for the modern-day industry.

* Practical Common Lisp
    ([link](http://www.gigamonkeys.com/book/))

* Clojure - Functional Programming for the JVM
    ([link](http://java.ociweb.com/mark/clojure/article.html))
  A comprehensive and well-written introduction to Clojure.

* Wikibooks on Clojure
    ([one](http://en.wikibooks.org/wiki/Learning_Clojure) and
     [two](http://en.wikibooks.org/wiki/Clojure_Programming))

* Learn Clojure website
    ([link](http://learn-clojure.com/))

* _On Lisp_ by Paul Graham
    ([link](http://en.wikipedia.org/wiki/On_Lisp))

* 4clojure website
    ([link](http://www.4clojure.com/))
  An extensive set of problems with multiple levels of difficulty. You can view
  other users' solutions for the problems you have solved yourself.

* ClojureDocs
    ([link](http://clojuredocs.org/))

  This is the reference of the Clojure language and its user-contributed
  libraries. The only one you'll ever need.

_This list has been updated on Tuesday March 13, 2012._

---
Tags: lisp, proglang, learning

Date: Thu Mar  1 17:52:05 EET 2012
