+++
categories = ['PowerShell']
tags = ['Migrated Post', 'Scripting', 'Tips and Tricks']
date = '2018-08-01T00:00:00-04:00'
draft = false
title = 'Powershell and Markdown'
author = 'Bobby'
summary = 'If you somehow made it to 2018 without using markdown yet, prepare to change that. Markdown support comes to PowerShell!'
description = "If you're reading this migrated post, check out Charmbracelet's `glow`"
+++

## PowerShell (Core) and Markdown

Markdown is a great way to document, well anything!  If you haven't had a chance to write it yet a time will probably come sooner than later when you'll have a case to use it.

You'll find markdown just about everywhere these days.  Places like VSTS, Github repository documentation, comment sections on sites Github or Reddit, and even the engine for this blog uses markdown files for posts!  If you aren't familiar with markdown, you probably have seen it without even realizing yet.

If you read the [patch notes][patch] (_also written in markdown_) for PowerShell *Core* 6.1.0 Preview 4 you'll find the exciting new news!  Under the "General Cmdlet Updates and Fixes" section there is a bullet for "_Add Markdown rendering Cmdlets ([#6926][issue])_".  Let's take a look!

### Table of Contents

<!-- TOC -->

- [PowerShell (Core) and Markdown](#powershell-core-and-markdown)
  - [Table of Contents](#table-of-contents)
  - [Update May 10th 2019](#update-may-10th-2019)
  - [How to Use It](#how-to-use-it)
  - [Viewing the Markdown](#viewing-the-markdown)
    - [In the Console](#in-the-console)
    - [In the Browser](#in-the-browser)
  - [Changing the Console Colors](#changing-the-console-colors)
  - [Quirks](#quirks)
    - [Quirk 1 - Setting Persistance](#quirk-1---setting-persistance)
    - [Quirk 2 - ConvertFrom-Markdown Console Text Issue](#quirk-2---convertfrom-markdown-console-text-issue)
    - [Quirk 3 - General Usage](#quirk-3---general-usage)
  - [Conclusion](#conclusion)

<!-- /TOC -->

### Update May 10th 2019

I noticed this post got shared on **Y-Cominator Hacker News** and a **Github** issue, so if you are browsing in from there, quirks 2 and 3 from the original post should no longer be valid!

I wanted to give this feature the justice it deserved.  When I originally wrote this, the feel of the commands felt very _-beta-ish-_.  I can safely say, the strange kinks seem to have been worked out!

You can now do the following...

```powershell
# Render a markdown file by path
Get-Item -Path ./README.md | Show-Markdown
```

Due to the changes in how the commands work they no longer can skew your console colors either, at least not in any way I have seen!

I originally thought this was a novelty, I ended up finding myself using it in the real world the other day as I needed to quickly reference a markdown file and didn't want to open it outside of my current PowerShell prompt.  This is most definitely a welcome feature to PowerShell in my opinion, especially with the quirks ironed out!

### How to Use It

At the time of writing, you will need to have installed **PowerShell Core 6.1.0 Preview 4**.  If you stumble on this blog in the future and are running a minimum of PowerShell Core 6.1.x this functionality should be available to you though it's possible the syntax could have changed.

![pwshVersion](/posts/pwsh-markdown/pwshMarkdown1.png)

Good ol' `Get-Command` should tell you what markdown related commands are available.

```powershell
Get-Command -Noun Markdown*
```

You can see we have 4 commands available for some markdown action.

![commands](/posts/pwsh-markdown/pwshMarkdown2.png)

There are two ways you can use this new markdown functionality in PowerShell.  Using `Show-Markdown` you can take markdown and display it as a VT100 encoded string right in the console.  Alternatively you can use `Show-Markdown` to display a markdown file in a web browser.

### Viewing the Markdown

#### In the Console

To parse and view markdown in the comfort of your PowerShell console you can run the following commands, to specify a markdown file, and then display it in the console.  Using this blog post as an example your commands would look like the following.

```powershell
ConvertFrom-Markdown -Path '.\_posts\2018-8-1-PowerShell-Markdown.md' -AsVT100EncodedString | Show-Markdown
```

When you run this it will yield results similar to the gif below[^1].

![console](/posts/pwsh-markdown/pwshMarkdown1.gif)

Pretty cool, you can take raw markdown, and view it in a slightly less jarring way in your PowerShell console.  This could be useful when there is a time when you need to work on a system without a Desktop environment, or just for console diehards who want to do as much in a console as possible.

#### In the Browser

You alternatively can view the parsed markdown in the browser.

```powershell
ConvertFrom-Markdown -Path '.\_posts\2018-8-1-PowerShell-Markdown.md' | Show-Markdown -UseBrowser
```

![browser](/posts/pwsh-markdown/pwshMarkdown6.png)

Not the prettiest markdown rendering, but it gets the job done.  Note that it will use your systems default browser, in this example Google Chrome.

### Changing the Console Colors

You may want to change the default colors that are used in the console display.

Back to console land for a minute, what the cuss is VT100?  I am admittedly not an expert, like, at all.  I do know ANSI/VT100 enabled consoles/terminal emulators are able to display colored text.  To see this, go ahead and run `Get-MarkdownOption`.

![vt100](/posts/pwsh-markdown/pwshMarkdown3.png)

You'll see that you have different markdown elements like Header1-6, Code, Link, Image, etc.  Each has an associated ANSI/VT100 escape code which will determine its color and formatting in the PowerShell console.  You can learn more about these escape sequences [here][ANSIVT100] if you're interested.

So, now you know how to see what the console colors will be by default, and have a nice link to a reference sheet of what to change them to if you want.

Let's say you want to change 'Header2' so that it is something other than yellow.  You can do that by using `Set-MarkdownOption`.  To make it underlined green you can use `[4;32m`

```powershell
Set-MarkdownOption -Header2Color '[4;32m'
```

You can preview your changes again with `Get-MarkdownOption`

![colorchange](/posts/pwsh-markdown/pwshMarkdown4.png)

There you have it, next time you display a markdown document in your console, Header2 will be green underlined text rather than the original yellow.

### Quirks

**NOTE!**  This is a preview edition of PowerShell with this functionality.  With that said, here are some quirks.

#### Quirk 1 - Setting Persistance

The `Set-MarkdownOption` changes don't persist.  If you change some colors, close the console and then start it back up the colors reset to the defaults.  This can be fixed by possible scripting your preferences to be set on load of your PowerShell profile.  Alternatively you can just change them when needed.

I am not sure if this is by design or the developer didn't want to worry about saving user preferences and where to save them across different operating systems, but either way, just be aware of it.

#### Quirk 2 - ConvertFrom-Markdown Console Text Issue

If you run the `ConvertFrom-Markdown` without piping into `Show-Markdown` your console formatting gets weird...  You can see below, a lot of my text became Header2-ish, yellow and underlined.

![whoops](/posts/pwsh-markdown/pwshMarkdown7.png)

#### Quirk 3 - General Usage

If you hit the footnote in the article you might have already saw that the general usage is kind of confusing and a lot of folks tend to think so as well.  You can follow the discussion on [Github][commandusage]

### Conclusion

Regardless of the quirks, this is really cool new functionality we have in PowerShell!  If you are interested in the PowerShell RFC for native markdown rendering you can check it out on Github, or if you want to use your newfound skills to view it the PowerShell way! [^2]

```powershell
Invoke-WebRequest -Uri https://raw.githubusercontent.com/PowerShell/pwsh-RFC/master/3-Experimental/RFC0025-Native-Markdown-Rendering.md | Select-Object -ExpandProperty Content | ConvertFrom-Markdown | Show-Markdown -UseBrowser
```

[^1]:
    Some say this feels awkward, do I?  Heck yeah, the fact that when using the console you supply the `-AsVT100EncodedString` in the `ConvertFrom-Markdown` command but to view it in the browser you leave this parameter off and instead pass `-UseBrowser` to the `Show-Markdown` command.  It just feels... odd.

[^2]:
    I so badly wanted to do this in the console but couldn't get `ConvertFrom-Markdown` to work without throwing a *ConvertFrom-Markdown : Object reference not set to an instance of an object.* exception.  I am not sure if it's because its trying to parse the VT100 escape sequences contained in the RFC,  I suspect something of that nature is occurring.

[patch]:https://github.com/PowerShell/PowerShell/releases/tag/v6.1.0-preview.4
[issue]:https://github.com/PowerShell/PowerShell/pull/6926
[ANSIVT100]:https://misc.flogisoft.com/bash/tip_colors_and_formatting

[commandusage]:https://github.com/PowerShell/PowerShell/issues/7338


