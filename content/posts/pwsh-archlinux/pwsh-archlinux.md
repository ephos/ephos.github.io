+++
categories = ['Linux', 'PowerShell']
tags = ['Migrated Post']
date = '2018-09-17T00:00:00-04:00'
draft = false
title = 'Powershell + Arch Linux = AWESOME!'
author = 'Bobby'
summary = 'Sometimes you just want an object based pipeline!'
description = 'Join me in installing PowerShell on my favorite Linux distribution, Arch Linux!'
+++

## PowerShell and Arch Linux

### Table of Contents

<!-- TOC -->

- [PowerShell and Arch Linux](#powershell-and-arch-linux)
    - [Table of Contents](#table-of-contents)
    - [Intro](#intro)
        - [Arch Linux](#arch-linux)
    - [Documentation](#documentation)
    - [AUR and an AUR Helper](#aur-and-an-aur-helper)
        - [Using makepkg](#using-makepkg)
        - [Using an AUR Helper (yay)](#using-an-aur-helper-yay)
    - [Conclusion](#conclusion)

<!-- /TOC -->

### Intro

I won't pull your leg, I have primarily been a Windows user my whole life.  Before I lose any die hard Linux readers, I will also say that I love Linux, like, A LOT!  Since I like to play PC games and have worked places that have huge Microsoft footprints I have just spent more time in the Windows world (_for better or worse_).  Between those two things, that's what made me jump into PowerShell over another language a long time ago.

I have been _**ecstatic**_ since PowerShell Core (_and .Net Core_) released and brought a era of open source and cross platform goodness to Windows, Linux, and Mac!

I still play PC games, but despite that I have decided that I am going to live as a Linux user henceforth, only booting into Windows to play games that can't run on Linux yet natively.  So **hello** from Linux, where I intend to stay (at least on my home desktop and laptop) from now on!

This isn't about me using Linux, it's about how to install PowerShell Core on my favorite Linux distribution, [Arch Linux][ArchLinux]!

![whynotboth](/posts/pwsh-archlinux/whynotboth.jpg)

#### Arch Linux

**NOTE BEFORE READING:** Arch Linux has a somewhat unique package manager (_pacman_) and has a rolling release model, which is part of why I love the distribution.  With that said, this process is a lot different than if you are going to install PowerShell on CentOS, RedHat, Debian, Ubuntu or almost any other distribution!

**Make sure you read the instructions specific to your Linux distribution!**

### Documentation

Before diving in, the most important place by far to start here is going to be the documentation on [Github][GitHubPwsh] for PowerShell.

The _README.md_ markdown file has a lot of details for installing PowerShell on Windows, Mac and many of the most popular Linux distributions.

In this example we're looking for Arch Linux, and we can see that it's in the section for Linux distributions that have community support.

![RTFM](/posts/pwsh-archlinux/Image1.png)

If you follow the link it'll bring you to official Microsoft documentation.  There you'll see that Arch support is experimental and you need to install PowerShell from the [AUR][AUR] (_Arch User Repository_).  Since this is a personal machine, you likely won't need official Microsoft support.  Also if you are familiar with Arch, installing packages from the AUR is a fairly standard activity[^1].

![MSFTDocs](/posts/pwsh-archlinux/Image2.png)

You can also see there a few different options available to us in the AUR.  If you navigate to the links listed on the Microsoft page you can view a bit about each of the package options.  You can install a package based off the latest Git commit to the master branch, the latest tagged release, or the latest available release binary[^2].

### AUR and an AUR Helper

We'll go ahead and take the latest available release [binary package][AURPowerShellBin] `powershell-bin` from the AUR.

You can see if you try to use `pacman` to find the package it doesn't show up.  `pacman` can't help us with packages in the AUR.

```bash
# Refresh the database
pacman -Syy

# Search the Arch repositories for PowerShell
pacman -Ss powershell-bin
```

![NoDice](/posts/pwsh-archlinux/NoPowerShell.gif)

You have two primary options for installing packages from the AUR.

1. You can get the build files, review them, and execute a `makepkg -si`, this is outlined in the [Installing Packages][InstallingAURPackages] section of the AUR page in the ArchWiki.
2. You can use an AUR helper.  An AUR helper will assist in searching the AUR, getting the build files, and running through the steps for you.
    1. As AUR helpers are not officially supported, if using one you will need to install it manually first!

We'll walk through both options.  Note that you only will need to do one if you're following along.

#### Using makepkg

Using `git` to get the build files.  Install `git` if needed `sudo pacman -S git`.

```bash
# Clone the AUR package down with git, use the "Git Clone URL"
git clone https://aur.archlinux.org/powershell-bin.git

# Navigate into the directory from the Git clone
cd powershell-bin

# AUR Packages are community created, MAKE SURE YOU REVIEW THE FILES BEFORE INSTALL!
cat PKGBUILD

# Run makepkg to build the AUR package, '-s' will sync dependencies, '-i' will install the package after build.
makepkg -si
```

This gif will show the process!

![makepkg](/posts/pwsh-archlinux/makepkg.gif)

Now we can PowerShell our days away!

#### Using an AUR Helper (yay)

Boy howdy you will have [A TON][aurhelperlist] of options for which AUR helper you want to use. My personal  preference lately has been "`yay`", [yay][yay]!

`yay` is a `pacman` wrapper, it can function as `pacman` or be used to install AUR packages.  It is a lightweight utility written in Go with a feel very similar to using `pacman` that also has all the [features][aurlegend] an AUR helper should have.

First you'll need to install `yay`.  The instructions for installing `yay` can be found on the [README.md][yayrtfm] of the Github project.

```bash
# Clone down the yay build files.
git clone https://aur.archlinux.org/yay.git

# Navigate into the directory from the Git clone
cd yay

# Run makepkg to build the package, '-s' will sync dependencies, '-i' will install the package after build.
makepkg -si
```

Just like that you should have a shiny new AUR helper to help install AUR packages!

Installing PowerShell now with `yay` is a one liner.  You will be asked if you want to review the files before install (_which you always should_).  You'll also be asked if you want to do a "Clean build", which just means that it will not export new variables that may prevent a successful build.

```bash
yay -S powershell-bin
```

When you go through the package you should review the files when prompted by the `yay` install.  When you are done reviewing you would use "**q**" to stop reviewing the files and continue the install.

![yay](/posts/pwsh-archlinux/yay.gif)

Again, we're ready to PowerShell it up!

### Conclusion

You should now have PowerShell installed on your Arch system, cool!

I usually don't expressly call them out, but the foot notes on this one are worth reading.  Lastly I want to re-stress, this blog post is specific to Arch Linux, the process for installing on any other distribution will be different!

[^1]:
    Arch Linux package management as well as the AUR are pretty huge topics in themselves which could be worthy of multiple blog posts.  The follow links will provide more context if interested.  [pacman][pacmanwiki], [aur][aurwiki], [aur helpers][aurhelperwiki]

[^2]:
    At the time of writing this (September 17th 2018) PowerShell Core 6.1.0 has released.  However as AUR packages are community maintained there is no AUR package that has 6.1.0, yet...  Hopefully at some point it gets picked up and added to the Arch Linux "extra" repository (like Ruby and Python).

[ArchLinux]:https://www.archlinux.org/
[GitHubPwsh]:https://github.com/PowerShell/PowerShell
[AUR]:https://aur.archlinux.org/
[AURPowerShellBin]:https://aur.archlinux.org/packages/powershell-bin/
[InstallingAURPackages]:https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages
[aurhelperlist]:https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers
[yay]:https://github.com/Jguer/yay
[aurlegend]:https://wiki.archlinux.org/index.php/AUR_helpers#Legend
[yayrtfm]:https://github.com/Jguer/yay/blob/master/README.md

[pacmanwiki]:https://wiki.archlinux.org/index.php/pacman
[aurwiki]:https://wiki.archlinux.org/index.php/Arch_User_Repository
[aurhelperwiki]:https://wiki.archlinux.org/index.php/AUR_helpers

