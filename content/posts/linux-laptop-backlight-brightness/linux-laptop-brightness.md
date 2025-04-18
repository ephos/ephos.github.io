+++
categories = ['Linux']
tags = ['Commands', 'Migrated Post']
date = '2022-11-29T12:00:00-04:00'
draft = false
title = 'Linux Laptop Brightness'
author = 'Bobby' 
summary = 'So you went ahead and installed a trendy window manager and now you need to change your laptop screen brightness...'
description = 'brightnessctl to the rescue!'
slug = 'linux-laptop-brightness'
+++

## Change Your Linux Laptop Backlight Brightness

### Table of Contents 

<!-- vim-markdown-toc GFM -->

* [The Challenge](#the-challenge)
* [Change the Backlight from the File](#change-the-backlight-from-the-file)
* [brightnessctl](#brightnessctl)
  * [Basic Usage](#basic-usage)
  * [Changing Brightness](#changing-brightness)
* [Reference Links](#reference-links)

<!-- vim-markdown-toc -->

### The Challenge

So you spent too much time looking at fancy configs on Reddit...

After looking at all the cool setups and decided to use a trendy window manager like **i3**, **dwm**, or even **Sway** if you're one of those fancy Wayland kids with an AMD card.
You finally decided to use one of them and realized, some batteries aren't included and some assembly is required.

Trust me, I lived it.  Granted I love it.  Tweaking every knob and tuning every dial.
You do need to figure some things out that you probably took for granted, simple things like changing your laptop screen brightness.

The laptop backlight on my Lenovo Thinkpad T480 was a bit of a learning journey for me.
I finally got what I needed in the end, gather 'round and hear my tale, maybe it will save you some time.

**⚠️  DISCLAIMER**:[^1] <br />
These are my specs:

* Laptop: Lenovo Thinkpad T480, i5-8250U, Intel UHD Graphics 620
* OS: Arch Linux
* WM: i3 (gaps)

_Obligatory Neofetch_... <br />
![neofetch](/posts/linux-laptop-backlight-brightness/neofetch.png)

I started by manually editing my _brightness_ file found within _/sys/class/backlight/intel_backlight/_.  Very un-fun.
I attempted some other utilities like `xbacklight` before settling on `brightnessctl`.
I don't have a compelling case to why I prefer `brightnessctl`, I just find it easy and simple.

### Change the Backlight from the File

If you [RTFM _(Backlight Hardware Interfaces)_](https://wiki.archlinux.org/title/backlight) you can get some detail on how this all works.  I think the Arch wiki quotes it best:

> The brightness of the screen backlight is adjusted by setting the power level of the backlight LEDs or cathodes. The power level can often be controlled using the ACPI kernel module for video. An interface to this module is provided via a sysfs(5) directory at /sys/class/backlight/.

If you actually look at the contents of the backlight file you'll see it's just a number.
_(Also in my case it is not just 0-100)_
This is the number I was modifying when I was just manually changing the file.

Here we have my backlight _**intel_backlight**_. <br />
We can see the _**brightness**_ of my intel_backlight and it is a not very intuitive _**"758"**_ currently on my system. <br />
We can also view the _**max_brightness**_ which is _**"1515"**_. <br />

![brightnessfile](/posts/linux-laptop-backlight-brightness/brightnessfile.gif)

Given this we can deduce at the time of writing this my screen is at about 50% brightness.
While we could edit the _**brightness**_ file it's probably easier to use a utility unless you want to feel like J.P. from Grandma's Boy.

### brightnessctl

Using `brightnessctl` is easy.
I won't include any gifs here because you won't see the screen change.
You will just have to trust me, a complete internet stranger!

**❗NOTE**: I am not using sudo because I added my user to the _video_ group.

Simple installation via `pacman` on Arch.

```bash
sudo pacman -S brightnessctl
```

#### Basic Usage

```bash
# Check current device backlight, note this will match the file 
brightnessctl

# You can also list other devices with brightness controls with -l, --list
brightnessctl -l

# If you plan to use the output elsewhere you can also use -m, --machine-readable
# This will output in a comma separated form which can be parsed easily by other tools
brightnessctl -l -m
# Lets deserialize it (pwsh only for this example)
brightnessctl -l -m | ConvertFrom-Csv

# Just get the brightness number from a device
brightnessctl -d intel_backlight get

# Check the maximum brightness of a device
brightnessctl -d intel_backlight max
```

#### Changing Brightness

```bash
# Set the backlight!
# This sets the device (-d, --device=DEVICE) to 75%
brightnessctl -d intel_backlight set 75%

# Increase from current value + or - 10%
brightnessctl -d intel_backlight set +10%
brightnessctl -d intel_backlight set -10%

# If you want to supress the output use -q, --quiet
# This can be useful if you just don't care about the return or want to wire this into a script
brightnessctl -d intel_backlight set 100% -q

# You can still set by number too, set to 1515 (max)
brightnessctl -d intel_backlight set 1515

# You can test with the -p, --pretend switch which does a whatif/noop
brightnessctl -d intel_backlight set 1% -p
```

**✅TIP**: If you are going to wire `brightnessctl` into another process make sure to leverage the `-m` switch.
Parsing the comma separated values is **MUCH** easier than trying to regex the default output.
I learned this the hard way by not reading the help good.  Always read the help good.

There you have it, hopefully it helps!


### Reference Links


* [Arch Wiki - Backlight](https://wiki.archlinux.org/title/backlight)
* [Reddit Post on brightnessctl permissions](https://www.reddit.com/r/archlinux/comments/fpklbs/brightnessctl_only_works_as_root/)
* [Github - brightnessctl](https://github.com/Hummer12007/brightnessctl)

[^1]:
    I am by no means an expert at all things Linux and laptop backlights. <br />
    There are more backlight utilities than I can count and more models of laptops that exist than I can even begin to imagine. <br />
    This is just what worked for me on my Thinkpad T480 running Arch Linux with i3. <br />
    Your mileage may vary! <br />

