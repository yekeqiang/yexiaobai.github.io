---
comments: true
date: 2013-07-05 14:30:00
layout: post
slug: jekyll-setup-pt-render
title: Jekyll Rendering
summary: "Jekyll Setup Part II: Rendering"
image: 'how-to-jekyll/logo-2x.png'
tags:
- jekyll 
- blog
---

Now that you have a fully operational blogging station, you'll
need to decide what kind of rendering you want to use.  Jekyll
comes with some sweet stock rendering using Maruku.  If you want somthing more
sophisticated, that will cost you another `gem`.  For example, 
[Garry Welding](http://in-the-attic.com) prefers the `rdiscount` gem which
can be easily installed via:

{% highlight console %}
gem install rdiscount
{% endhighlight %}

If you get a complaint about permissions or needing to be root, you may want
to review my [last post]({{ page.previous.url }}) how to install ruby and gems
for Jekyll.

Then to enable RDiscount, head on over to your `_config.yml` file and set the
`markdown` field.

{% highlight yaml %}
markdown: rdiscount
{% endhighlight %}

Personally, I'd rather have the option to use LaTex, pronounced, say it with
me now Lay-Tek.  Maruku has the ability to render LaTex to PNG files using
`blahtex` and `dvips`.  Of course, `dvips` must be on your path.  To install
`blahtex` on your Ubuntu machine with something like version 11ish or greater
all one has to do is:

{% highlight console %}
sudo apt-get install blahtexml
{% endhighlight %}

I didn't notice the **ml** on the end and wound up getting set up for an old version of the `blahtexml` from an Ubuntu PPA.  Totally unessecary, but only cost me a few minutes.

*But*, if you go with Maruku, then your posts may not be rendered as HTML5
complaint.  So it looks like I need another path to LaTex rendering...

