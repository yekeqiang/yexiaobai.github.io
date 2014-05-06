---
comments: true
date: 2013-07-16 15:00:00
layout: post
slug: git-pretty-log

title: Let's Git Pretty
summary: "Pretty Command Line Git Log"
image: 'git-pretty/git-pretty.jpeg'
tags:
- git
- stackoverflow
---

Ever find yourself tired of going to `gitk` everytime you do a commit?
What about when you're working on your server, no `gitk` happening there.
Most of the time I find that I just want to make sure I'm on the right
branch.  I know this is right on `git status`, but I like to make double
extra sure, seeing as `git` commits are like love, they're forever.

Whilest trolling along the world wide web looking for a solution, I stumbled
on this great stackoverflow [question](http://stackoverflow.com/questions/1057564/pretty-git-branch-graphs).  How can you not upvote that answer? Talk about
**really** impressive.

{% highlight yaml %}
[alias]
  lg1 = log --graph --abbrev-commit --decorate --date=relative --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
  lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all
  lg = !"git lg1"
{% endhighlight %}


