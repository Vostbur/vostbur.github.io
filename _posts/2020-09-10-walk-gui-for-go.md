---
layout: post
title: Native Windows GUI with Golang and walk
---

I recently wrote a [post](https://vostbur.github.io/post-2020-09-05/) about my attempt to make a Windows desktop app to track changes in a selected folder. It looks like I found the right library for my task. This is a [lxn/walk](https://github.com/lxn/walk).

I haven't figured out everything I need. For example, I had to change some source code to add a scroll bar to the TextEdit widget *(the win.WS_VSCROLL flag has been added to func NewTextEditWithStyle when creating the widget)*.

For simple Win GUI applications, I think this library is the best choice.

Here's what I got

![](/images/go-walk-watchdog.PNG)

The source code and instructions for compilation are [here](https://github.com/Vostbur/whatchange).

