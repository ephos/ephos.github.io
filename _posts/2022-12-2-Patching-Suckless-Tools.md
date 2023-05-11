---
title: Patching Suckless Tools
teaser: You're using a beloved Suckless tool like dmenu and you want to customize it, it's easy! 
category: linux 
tags: [linux, shell]
published : false
reddit_post: 
featured_comments:
---

## Patching Suckless Tools

### Table of Contents

<!-- vim-markdown-toc GFM -->

* [The Challenge](#the-challenge)
* [What is Suckless?](#what-is-suckless)
* [Patching dmenu](#patching-dmenu)
* [Reference Links](#reference-links)

<!-- vim-markdown-toc -->

### The Challenge

### What is Suckless?

### Patching dmenu 

```bash
git clone https://git.suckless.org/dmenu

cd dmenu

```

**✅TIP**: If you are going to wire `brightnessctl` into another process make sure to leverage the `-m` switch.
Parsing the comma separated values is **MUCH** easier than trying to regex the default output.
I learned this the hard way by not reading the help good.  Always read the help good.

There you have it, hopefully it helps!

### Reference Links

* [Arch Wiki - Backlight](https://wiki.archlinux.org/title/backlight)
https://www.youtube.com/watch?v=bBJ0qxqzlxk

---

[^1]:
