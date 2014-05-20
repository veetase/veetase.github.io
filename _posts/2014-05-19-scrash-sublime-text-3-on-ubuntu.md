---
comments: true
date: 2014-05-19 14:19:11
layout: post
slug: scrash-sublime-text-3-on-ubuntu
title: Scrash Sublime-Text3 On Ubuntu
categories:
- tools
tags:
- sublime-text
---
Edit the sublime in /opt/sublime with bless:
{% highlight html linenos %} {% raw %} 
    
    On 64 bit linux, replace:
        85 C0 0F 94 C0 84 C0 88 05 E3 26 4E 00 75 41 48
    to:
        90 90 90 90 90 84 C0 88 05 E3 26 4E 00 75 41 48
    On 32 bit linux, replace:
        0F 94 C0 84 C0 A2 68 CE 4F
    to:
        90 90 90 90 90 A2 68 CE 4F
{% endraw %} {% endhighlight %}
