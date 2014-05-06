---
comments: true
date: 2013-07-04 14:30:00
layout: post
slug: finally-found-jekyll
title: Finally Found Jekyll
summary: "The blog engine I've been looking for"
image: 'generic-software/generic-software-screen.jpg'
tags:
- jekyll 
- blog
---

This will likely be the first post in a series about the Jekyll, blog-aware,
static site generator in Ruby.  

I've been looking for a blogging vehicle for quite some time now.  Nothing has
been able to fit my needs.  Fortunately for me I just found
[Jekyll](http://jekyllrb.com/)!  This is **exactly** what I've needed!
Ultimately, I've been looking for a platform that would fit the following:

1. Use **Markdown** to write all post content.
2. Minimal, preferably no, design required at initial deployment. 
3. Concentrate on content, not blog plugin management or web framework.

Jekyll hits the nail on the head in all these categories.  I was amazed to find that it had just what I needed.

A friend of mine that does a lot of Linux/Embedded Linux consulting
[Cliff Brake](http://bec-systems.com/site/) first pointed me in the way of
[blacksmith](http://blog.nodejitsu.com/introducing-blacksmith). He's such an incredible engineer
that I took his advice hook line and sinker.


I went off installing [node.js](http://nodejs.org/) on my laptop and VPS to
start the ball rolling.  I found that it did not appear to run as advertised
on my machine.  Knowing that it was clearly something on my end since it would
appear that many people are using `blacksmith` without any difficulty I started
to look at where I had gone wrong.  It seems that `node` prefers to be
installed in `/usr/local/` and not by root.  Undaunted, I began to correct my
errors and continued on the `node.js` course.


One of the features I really was impressed with is the whole idea of 
"statically generated" websites on the server end.  This allows the server
to serve up static files like it's 1995 baby!  Screaming fast HTML on demand.

This makes a lot of sense to me, likely because I've spent a lot of time
working with [Simulink](http://mathworks.com) and its code generation.  When
integrating existing C source code with Simulink generated code one often uses
Target Language Compiler, or TLC code to allow Simulink and C to play
together.  The TLC acts like a preprocessor on steroids, as my colleage puts
it, creating marvelously simple interfaces and fast execution.  Only code that
is required for an application is generated.  Many conditional checks are
avoided because the decision of what should be in the code is made at compile
time, *not* run time.  This is a great benefit, one which is made possible
through a feature rich preprocessor, if I may use that term, like TLC.  Seems
to me that C is missing that feature, maybe it can be addressed someday.
Thus, I desired the same kind of decision making at "compile time" for a
website, knowing that performance is a critical factor in success for any web
based application.


However, I could not seem to get `blacksmith` under control, so I turned to its
cousin, [wintersmith](http://wintersmith.io/).  Again, I found it was not
as easy to run as I first had hoped.  I was beginning to become discouraged and
feared I would return to the Wordpress disaster that I had left behind.  Then, 
I called to remembrance the complete failure that I had with Wordpress and kept
searching, until I found Jekyll.  I need to get some feedback from Cliff his
recomendation, I'm certain it was a good one and ultimately did lead me out
of the Wordpress wasteland.  However, my Wordpress woes were not a function
of the actual Wordpress engine, but of how I prefer to work and deliver
content.  It appears to be orthogonal to how Wordpress authors operate.  I'd
rather put some text in an editor and dress it up with Markdown that tangle
with some WYSIWYG Content Management System, to each his own!

Jekyll, as far as I'm concerned, is extremely easy to run and administer.
Another huge benefit is that [Github](http://www.github.com) has opened up a
service where a user may have their Jekyll blog hosted free on github.  The
content of each blog is often then open sourced in github.  Now the content
can be forked, cloned, etc.  This is the solution for me!  I search around and
found a very well designed site by [Garry Welding](http://in-the-attic.com).
I cloned it, made some minor customizations and now have this blog! 
Outstanding!  Of course, you can find the source at
[github](http://www.github.com/macduff).  I would recommend any interested
persons follow [his blog post](http://in-the-attic/2013/01/04/building-a-blog-using-jekyll-bootstrap-and-github-pages-a-beginners-guide) to get it
straight from the source.  In one fell swoop you get tag support(not standard
with Jekyll), [bootstrap](http://twitter.github.io/bootstrap/) integration, and a host of
other nicities and eye candy.  Enjoy!

