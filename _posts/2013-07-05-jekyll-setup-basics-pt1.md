---
comments: true
date: 2013-07-05 14:30:00
layout: post
slug: jekyll-setup-ruby
title: Jekyll Setup Ruby
summary: "Jekyll Setup Part I: The Basics"
image: 'how-to-jekyll/Rubyprogramminglanguagelogo2008.png'
tags:
- jekyll 
- blog
---

Jekyll setup is advertised as being as simple as:

{% highlight console %}
~ $ gem install jekyll
~ $ jekyll new my-awesome-site
~ $ cd my-awesome-site
~/my-awesome-site $ jekyll serve
# => Now browse to http://localhost:4000
{% endhighlight %}

It's not *quite* that easy, but it's close...real close. :-)

Jekyll is, and I quote:

>  Jekyll is a simple, blog-aware, static site generator.

So true.  For those who are puristic at heart, and love the Keep It Simple
philosophy, these words are like music to the ear.  Finally, a way to keep
a web page up to date using only a text editor, if one is so inclined.  Okay,
enough of the dribble, let's get to the setup basics.

Jekyll is written in [Ruby](http://www.ruby-lang.org/en/) a highly object oriented
language which likely needs no introduction.  (However, if it does, I recommend
"Programming Ruby, The Pragmatic Progammers' Guide.")  So, to run Jekyll you need ruby on your machine.  I use Ubuntu as my primary OS so I needed Ruby on my
laptop and VPS to begin using Jekyll.

First, let's discuss the development machine, my laptop, we'll come back to my
VPS at on another post.  I searched around and found that installing the debian
package of ruby was not recommended, as it was old and apparently not well
maintained.  This seems odd to me, seeing that ruby is pretty cutting edge
these days, as well as being **extremely** popular if
[stackoverflow](http://www.stackoverflow.com) is any indication.  I found
some instructions online that recommended **not** installing ruby into a more
traditional `/bin/` location but rather in the user's home directory.  This
sounded familiar to the [last post]({{ page.previous.url }}) about the
`node.js` requirements so I was a little hesitant, but found that the ruby
solution is much less intrusive. I had read that the best plan of attack for
`node.js` is to `chown` the `/usr/local` directory to either your user account
or some other user account that will be in your group.  This just didn't ring
true to me, so I shyed away from it, more because I feared hosing my system
accidentally.

I found that ruby was already on my system, so I removed it completely with
no deliterious effects.

{% highlight console %}
# I like to use dpkg to examine my system, but go back to
# aptitude to do the installation and removal if at all possible
sudo dpkg --list | grep gem
# read through the packages installed that match 'gems' and remove
# the ruby ones
sudo apt-get remove rubygems
# read through the packages installed that match 'ruby' and remove
# one at a time, just for a little more fine grained control
sudo dpkg --list | grep ruby
sudo apt-get remove ruby
sudo apt-get remove ruby1.8
sudo apt-get remove ruby1.8-dev
sudo apt-get remove libruby1.8
{% endhighlight %}

This gave me a machine clean of old, unwanted ruby installs.  Then to install
ruby the "right" way, I used:

{% highlight console %}
# get the ruby virtual machine via curl(who'd a thunk it?)
curl -L get.rvm.io | bash -s stable --auto
# make a backup of your .bashrc file
# this is due to a difference in Ubuntu releases .profile
# and .bash_profile behaviour  The ultimate goal is to get
# ruby initialized correctly for every terminal instance.
# Since there are subtle differences in .profile and .bash_profile
# I found it easiest to just add the ruby initilization lines
# to the end of my bashrc
cp .bashrc bash.bashrc
cat .bash_profile >> .bashrc
{% endhighlight %}

That worked!  So now I can close my terminal and start a new one.  Then test
for the right Ruby Virtual Machine install.

{% highlight console %}
which rvm
		/home/macduff/.rvm/bin/rvm
{% endhighlight %}

Perfetto!  Now, get the rvm requirement and install any packages via `apt-get`.

{% highlight console %}
# this will run an automated process to download and install
# some required software, it may complain about not being root,
# but that can be ignored for now
rvm requirements
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev
curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev
sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake
libtool bison  subversion pkg-config
{% endhighlight %}

I think I had all of the above, but thought I would repeat it just for
completeness.  Now, just setup `rvm` to behave and test its output.

{% highlight console %}
rvm install 1.9.3
rvm use 1.9.3
ruby -v
	 ruby 1.9.3p448 (2013-06-27 revision 41675) [i686-linux]
# note the build number and set it as default
rvm --default use 1.9.3-p448
which gem
	 /home/macduff/.rvm/rubies/ruby-1.9.3-p448/bin/gem
{% endhighlight %}

Magnifico!  Now we're ready to install Jekyll.

{% highlight console %}
gem install jekyll
jekyll new myblog
cd myblog
jekyll serve # => Now browse to http://localhost:4000
{% endhighlight %}

If all goes well, you'll bring up a plain, but nice default Jekyll blog
all ready for you to start blogging!

