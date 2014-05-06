---
comments: true
date: 2013-07-08 15:00:00
layout: post
slug: jekyll-setup-vps

title: Jekyll Setup VPS
summary: "Jekyll Setup Part III: Virtual Private Server"
image: 'how-to-jekyll/wp_github_jekyll.png'
tags:
- jekyll 
- blog
- server
---

## Motivation:

If we're already using Jekyll, then why should we even go to the trouble
of hosting it ourselves? I mean, we could just get it hosted for **free** on
Github.  That's pretty crazy simple, follow the
[simple instructions](https://help.github.com/articles/using-jekyll-with-pages)
and boom, you're blog is up!  You can even pay a paltry sum and get it to
resolve you custom domain and then get private github repos to boot!

This is a pretty sweet deal, however, since github is being so incredibly
generous with their server space, they do require something in exchange.  You
can't really use any of the Jekyll Ruby plugins due to security concerns.  The
only way to add functionality is through the Liquid templating language, which
can be pretty tricky.  Since I'm more interested in functionality, like tags,
categories, sitemap generation, etc. I'm quite motivated to setup the blog to
run on my own virtual private server.

## Server Prerequisites

To host a Jekyll blog is a little tricky, but all things considered, it's
pretty easy.  I'm going to assume that the interested reader is running
something like Ubuntu 12.04 LTS or later with Apache up and rolling.

### Git

Git installation is pretty much a piece of cake.

{% highlight console %}
sudo apt-get install git git-base git-core git-svn gitk git-gui
{% endhighlight %}

That should pretty much get `git` on your system, no problems.  What we'd really
like is to have an automated method of deploying our content online.
Fortunately for us, this can easily be accomplished using a simple recieve hook
that is part of the git repo that we will soon create.  There are other methods
to accomplish this task, but I'll outline the one that I found conceptually
the simplest to implement.


#### Setup A Deployment Account

We'll setup an account that will be used only for the sole purpose of updating

our git repo on our server.  This technique can be used to host many different
repos on the same server, and in theory, could allow many users to push and
pull changes to the repo.  Since it's using the same user account, some might
be hesitant, but have no fear, it's likely that you'll be the only one that
needs to update your *personal* blog, right?  :-)

We'll give our user the name of `deployer` to be consistent with some of the
other documentation for Jekyll, but feel free to change the name to whatever
you choose.

{% highlight console %}
sudo groupadd git-data
sudo useradd -g git-data -G www-data --shell /usr/bin/git-shell deployer
{% endhighlight %}

This should create a new user account with the initial login group of
`git-data`.  Notice we set the shell of this user to be a `git-shell`.  This
shell only allows a few commands and will help keep our server secure.  When we
create the user it will also ask for a password.  Be sure to choose a strong
password and one you'll remember.  I tend to think the intersection of the
set of strong passwords and ones you'll remember are equivalent to the empty
set, so I would recommend looking into password software like
[password safe](http://passwordsafe.sourceforge.net/) or [KeePass](http://keepass.info).

### Ruby

As we discussed on a [earlier post]({{ page.previous.url }}) , Ruby should not
be installed using `apt-get` as the debian package is painfully old and just
causes a lot of problems.  Before, we installed Ruby on our development machine
in our `home` directory.  This is not such a good plan when hosted on a server where
different accounts need access to the same Ruby and gem set.  So, we'll have to
go after a different plan of attack.

First step is to install the [Ruby Version Manager](http://rvm.io), which will help
make our lives far easier.

{% highlight console %}
sudo  bash << \
	(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer )
source /usr/local/rvm/scripts/rvm
which rvm
	/usr/local/rvm/bin/rvm
{% endhighlight %}

So now we have a Ruby Version Manager from `/usr/local/rvm/bin`, which is system wide
accessible, perfect!

Now, let's install jekyll and rdiscount to render the blog:

{% highlight console %}
sudo gem install jekyll
sudo gem install rdiscount
{% endhighlight %}

With any luck, we now should be ready to serve up a jekyll blog of our very own!

### Repo Location

Now, let's create a location where our git repos will be stored.  I've run
across recommendations to place it in `/opt/git/`, after looking into this
option I've decided I like it!

{% highlight console %}
# create the directory
sudo mkdir -p /opt/git
# make it so that the directory and all subdirectories
# will be part of the git-data group
sudo chgrp git-data /opt/git/
sudo chmod g+s /opt/git/
{% endhighlight %}

We can now create a bare repository in our `git` directory to use for our blog.

{% highlight console %}
sudo cd /opt/git
sudo git init --bare yourblog.github.com
{% endhighlight %}

I'm assuming you're going to open source your website and mirror it in github as
well, but it's not a requirement.  :-)  Now, let's change permissions on the new
repo.

{% highlight console %}
sudo chown -R deployer:git-data .
{% endhighlight %}

But that's not all!  We can also modify the bare repo to automagically update our blog
site.  How neat is that!  :0)

{% highlight console %}
sudo vi yourblog.github.com/hooks/post-recieve
{% endhighlight %}

Let's fill out the recieve hook:

{% highlight bash %}

#!/bin/bash

# initialize the ruby version manager
source /usr/local/rvm/scripts/rvm

# assuming you put your blog in /var/www/blog
GIT_REPO=/opt/git/yourblog.github.com.git
TMP_GIT_CLONE=$HOME/tmp/yourblog.github.com
PUBLIC_WWW=/var/www/blog/yourblog.github.com

git clone $GIT_REPO $TMP_GIT_CLONE
jekyll build -s $TMP_GIT_CLONE -d $PUBLIC_WWW
rm -Rf $TMP_GIT_CLONE
exit

{% endhighlight %}

This will automagically update your blog and serve it up, as long as you update
your Apache configuration to allow a link to `/var/www/blog`.

### Push Up Your Blog

It's time for our blog to go live!  We can just start with any old blog, like the default
one that comes with jekyll, and push that to our new repository.

{% highlight console %}
# on local machine for testing
jekyll new testBlog
cd testBlog
git init .
git add .
git commit -m "testing blog"
git remote add deployer deployer@yourdomainname.com:/opt/git/yourblog.github.com
git push deployer master
{% endhighlight %}

We should get a prompt that tells us to enter our password and then see the beauty that is
a repo push.  If you don't want to horse around with entering your password everytime, you
can follow the forthcoming instructions. which have mostly been lifted from the
[jekyll homepage](http://jekyllrb.com/docs/deployment-methods/).

### Authorized Users

Let’s walk through setting up SSH access on the server side. In this example,
you’ll use the `authorized_keys` method for authenticating your users.
Create a .ssh directory for that user.

{% highlight console %}
sudo cd /home/deployer
sudo mkdir .ssh
sudo chown deployer:git-data .ssh
sudo chmod 755 .ssh
sudo touch .ssh/authorized_keys
sudo chown deployer:git-data .ssh/authorized_keys
sudo chmod 600 .ssh/authorized_keys
{% endhighlight %}

The last bit of permissions is explained in the Unix Stack Exchange question, 
[problems with public key authentication](http://unix.stackexchange.com/questions/36540/why-am-i-still-getting-a-password-prompt-with-ssh-with-public-key-authentication)

> Make sure the permissions on the ~/.ssh directory and its contents are
> proper. When I first set up my ssh key auth, I didn't have the ~/.ssh
> folder properly set up, and it yelled at me.

* Your home directory `~` and your `~/.ssh` directory on the remote machine
  must be writable only by you: `rwx------` and `rwxr-xr-x` are fine, 
  but `rwxrwx---` is no good, even if you are the only user in your group
  (if you prefer numeric modes: `700` or `755`, not `775`).
* Your private key file (on the local machine) must be readable and
  writable only by you: `rw-------`, i.e. `600`.
* Your `~/.ssh/authorized_keys` file (on the remote machine) must be
  readable (at least `400`), but you'll need it to be also writable
  (`600`) if you will add any more keys to it.


Next, you need to add your SSH public keys to the `authorized_keys` file for each
account you want to have authorized access.  Do the next bit on your development machine.

{% highlight console %}
cd ~/.ssh
ssh-keygen -f id_rsa.yourkeys -C '' -N '' -t rsa -q 
sudo chmod 600 id_rsa.yourkeys
cat id_rsa.yourkeys.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
dAv8JggJICUvax2T9va5 gsg-keypair
{% endhighlight %}

Now, get the public key up to the server and just append them to your
`authorized_keys` file on the server, logged in as a
[sudoer](http://www.sudo.ws/sudoers.man.html):

{% highlight console %}
sudo cp /home/deploy/.ssh/authorized_keys .
sudo chmod 777 authorized_keys
cat id_rsa.yourkeys.pub >> authorized_keys
sudo chmod 600 authorized_keys
sudo cp authorized_keys /home/deploy/.ssh/authorized_keys
{% endhighlight %}

### Let's test

Now, mosey on over to your blog web page and you should be greeted by a warm default blog page.
