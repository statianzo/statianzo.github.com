---
post: post
title: "Stacking Boxes: VirtualBox on Dropbox"
---

Two Homes(~)
---

Since starting at Neumont in 2007, I've owned, used, and abused a Lenovo Z61p.
It's treated me well and still in excellent running condition. After getting
hired year ago, I received a new Dell M4400 for work. Exciting as it was to have
another new piece of equipment, I quickly realized that maintaining two
development environments would become a hassle.  Storing personal projects in
source control was a start. However, there are things that don't get carried
over with that such as settings and installed software. Also, I frequently use
Linux on my personal laptop while my work laptop is strictly a Windows machine.
Choosing one was not an option; my Dell at home is a downer because of its 
keyboard.


Going Virtual
---

Throwing some ideas around, I realized that using a virtual machine would be a
good solution. They're easy, disposable, and most importantly, portable. Having
a history using Oracle's (previously Sun's) [VirtualBox][virtualbox] and its
cross platform usability, I took that option.

Choosing a distribution took a few attempts. [Damn Small Linux][dsl] was the
first attempt. Definitely small (sub 200MB) and easy to install. However, it's
package selection is small and outdated. DSL just didn't feel suited for
developing; facing issues building several tools, I switched to its
(considerably) larger parent, [Debian][debian].

Setting up Debian with X11, lxde, Ruby, ghci, et al. fits into ~1.5GB, which was
small enough to be portable. Also, I appreciated how easy everything was to get
working. 

To The Cloud
---

I've had a [DropBox][dropbox] account for awhile, but rarely used it after
initially signing up. With 2.5GB of account free space, my virtual hard disk fit
perfectly. 
