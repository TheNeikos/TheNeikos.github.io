---
date: 2015-09-30T17:14:43Z
title: Zabbix - Monitoring
url: /2015/09/30/Zabbix_-_Monitoring/
---

Well, it's been a long time, but here is another blog post from my side.

Today I am going to talk about Zabbix, it's a pretty great monitoring
Application that is easy enough to install and get running. I am not sure
regarding security when it is openly accessible, so if you need that do your
research first!

### How it works

The basic setup is simple, it's a glorified key-value storage.
You basically just create 'Hosts' that are then polled for data.
This data is saved for the configured time (usually a year) and can be displayed
in various forms (including your standard gnuplot graphs).

This data is then continously analyzed and matched against 'Triggers'.
Triggers themselves are simple equations that get triggered should the
expression be true. This can be anything, from a hard coded limit, to a sudden
jump in values (for example CPU usage jumping without a reason).

### Why I think it is worth to use it in simple situations at least

It is easy to setup (just a few commands on Centos 7), it is rather quick
(although not pretty), and it *works* which is an important property.


[Zabbix](http://www.zabbix.com/)
