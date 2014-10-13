---
layout: post
date: 2014-10-13 23:56:37
title: The Need for Documentation
---
I have started working on an
[open source project](https://github.com/waysome/waysome) which is going to
implement the wayland protocol. I find this awesome as I am doing this as part
of my University project of the fourth semester. I am doing it merely a semester
earlier (because I wanted to).

Now, to the subject of the actual blog post: **Documentation**! To be honest,
I always dismissed documentation as being second rank (and sometimes still do)
despite learning everyday how annoying it is to dive into a library/software
that is not properly documented.

In this case it is `libwayland`. I really appreciate the effort that have gone
into the project so far, but getting started in developing a compositor for it
is sooooooooo annoying! First of all, there is no real code documentation.
Sure, there exist documents that lie out how it is built up. But just because
now you know component *X* and *Y* exist doesn't mean you know how to use them
correctly.

Then there is the other problem child. `Weston` itself. Just look at it's
[source](http://cgit.freedesktop.org/wayland/weston/tree/src/compositor.c).
You cannot expect anyone to seriously read that through and get a good idea of
what is going on. It's _4805_ lines of code! When I first saw that I thought I
got the wrong sources. But no, it seems that this is a monolithic c file.

I do recognize though that they have splitted up *some* parts, but it's not
getting really better there. But do not get me wrong, I am not expecting Weston
to be a representation of a sample compositor. But as it is made by the same
people who have designed wayland I am a bit disappointed how the code looks
like. (Severly infact)

This has not stopped me from trying to find out how to get wayland do your
bidding. A few resources have been helpful (you can check them out in the wiki
of the associated object) and by now we manage to communicate with a desktop
client.
We've only been at it for 2 weeks, but it's shaping up nicely in my opinion.
