+++
categories = ['Powershell']
tags = ['Migrated Post', 'Tips and Tricks', 'Scripting']
date = '2022-08-20T00:00:00-04:00'
draft = false
title = 'Ready, Set, Wait! PowerShell Timers.'
author = 'Bobby'
summary = 'Do you have to code around long running jobs? Processes that might get stuck with no end? Use a Stopwatch!'
description = 'Timers can help end those runaway scripts you have.'
+++

## Using a StopWatch in PowerShell

### Table of Contents

<!-- TOC -->

- [Using a StopWatch in PowerShell](#using-a-stopwatch-in-powershell)
    - [Table of Contents](#table-of-contents)
    - [The Problem](#the-problem)
    - [Our Working Example](#our-working-example)
    - [Time This](#time-this)
        - [Create the StopWatch](#create-the-stopwatch)
        - [Create the TimeSpan](#create-the-timespan)
        - [Time Math](#time-math)
    - [Putting it all Together](#putting-it-all-together)

<!-- /TOC -->

### The Problem

Okay, we've all been there.  Yeah, you know, the time you need to automate something around some kind of process or task that is less than cooperative or predictable.  Sometimes it finishes, but sometimes it gets hung up, and your script gets stuck in a loop... forever and ever and ever.

![ShamelessMeme1](/posts/pwsh-timers/morpheus.jpg)

This is a trick I have used at work a few times.  Let me be forward and state, **every time** I have implemented this solution to _"fix"_ a problem, it is generally due to me coding around something bug ridden, or sporadically unpredictable that is entirely outside of my control.

With that said this is also useful to record timings on jobs that run long but definitely **do** have a set end time.

You may also be thinking, _"Rob, I already have a method to do this using `Start-Sleep` and an incrementing index!"_.  You know what, that is completely valid too, this is just a different way to solve the same problem!

Let's write some awful code to demonstrate this!

![endless](/posts/pwsh-timers/endless.gif)

Okay, feel free to watch this gif, it's entirely too long, and it will never end.  It is still running on my computer to this day _(not really)_.

Say this is some task you need to wait for in your automation, you need a way to defensively code in that failsafe _"enough is enough just stop running already!"_.

### Our Working Example

If you want to follow along at home, you can use the following whipped up code to simulate something a little better than `while` loop based on a condition of `$true`.  This is what will be used for the rest of this write up.

```powershell
Start-Process -FilePath 'C:\Windows\System32\calc.exe'
Start-Sleep -Seconds 5

do
{
    Write-Output -InputObject 'Checking if calculator is running...'
    $calcProcess = Get-Process -Name Calculator -ErrorAction SilentlyContinue
    if ($calcProcess)
    {
        Write-Output -InputObject ('Calculator is running, PID {0}' -f $calcProcess.Id)
    }
    Start-Sleep -Seconds 5
}
until (-not $calcProcess)
```

This block of code will open up calculator _(on Windows)_ and then enter a `do` loop checking any matching processes until it doesn't find one[^1].  This loop will run as long as any calculator is open, you can close it and end the loop, but this is to simulate a situation where you can't.

Imagine in this situation you can't stop the calculator, or maybe checking a locked file, or waiting for a VMware Guest OS customization to finish.

Okay let's implement a workaround!

### Time This

Where do you start?  You have a `do` loop doing some stuff that could get nasty, so you'll need to add some logic to your `until`.  Currently your `until` is only checking if a calculator is open, but you can add some additional logic to it.

#### Create the StopWatch

You'll need to instantiate a `[System.Diagnostics.StopWatch]` object.

_"What the heckin' cuss Rob, I am not a C# developer, what in tarnation are you saying!?"_

Many of you probably have done this without realizing, and if you haven't it's easy.  You just need to use `New-Object` to create a stopwatch!  It's as easy as 1, 2, `$stopWatch = New-Object -TypeName System.Diagnostics.Stopwatch`!

```powershell
#Create a Stopwatch
$stopWatch = New-Object -TypeName System.Diagnostics.Stopwatch

#You can use the $stopWatch variable to see it
$stopWatch

#Go ahead and check out the methods and properties it has
$stopWatch | Get-Member
```

![stopwatchcreate](/posts/pwsh-timers/stopwatch.gif)

Boy howdy, that was easy, right?  Did you see those nifty methods?  You can use them to...

- Start the stopwatch
- Stop the stopwatch
- Restart the stopwatch (and start running from 0 again)
- Reset the stopwatch (and keep it stopped)
- Use the `.Elapsed` property to get the elapsed time as a `[System.TimeSpan]`

You can give it a try yourself!

![stopwatchinaction](/posts/pwsh-timers/stopwatch-2.gif)

Okay, you're keeping time now, but a timer isn't really useful by itself.  You'll need to evaluate how much time has passed in for your check when you add it to the `until` logic.

You could just do something like `$stopWatch.Elapsed.Seconds -ge 60` if you want to wait until 60 seconds has passed.  This works, but can get complex if you want to wait for a more defined **time span**.  For example, if you want to wait 1 day, 5 hours, 34 minutes and 9 seconds this logic is going to get pretty annoying to maintain.

#### Create the TimeSpan

You can use a TimeSpan to do the time math, why even bother with comparative logic for time when you have PowerShell to handle that peasant work for you?

You can use `New-TimeSpan` to create a `[System.TimeSpan]` object.  You can then use this time span to do comparative logic against your timers elapsed time!

Remember that the `.Elapsed` property returns a `[System.TimeSpan]` of your timer.  You can even double check if you'd like by running `($stopWatch.Elapsed).GetType()`

![stopwatchelapsed](/posts/pwsh-timers/StopWatchElapsed.png)

Now create a time span with `New-TimeSpan`.  For this, you can create it with **1** minute and **30** seconds.

```powershell
#Create a time span
$timeSpan = New-TimeSpan -Minutes 1 -Seconds 30

#You can inspect it if you want using the variable
$timeSpan
```

![timespan](/posts/pwsh-timers/TimeSpan.png)

#### Time Math

If you were playing around with your stopwatch, make sure you stop and reset it before continuing if you are following along.

```powershell
$stopWatch.Reset()
```

You can very easily do comparisons between your time span, and your stopwatches elapsed time now as they are both of type `[System.TimeSpan]`.  Even your current elapsed time at 0, you can still see the comparison.

```powershell
#Stopwatch should be at 0 elapsed time.
$stopWatch.Elapsed

#Time span should be set for 1 minute and 30 seconds.
$timeSpan

#You can compare [TimeSpan] to [TimeSpan]!
#This will return true, as the stopwatch elapsed time of 0, is of course less than 1 minute and 30 seconds.
$stopWatch.Elapsed -le $timeSpan

#And similarly you can check if your stopwatch elapsed time is greater than or equal to the specified time span, in this case it is not.
$stopWatch.Elapsed -ge $timeSpan
```

![timemath](/posts/pwsh-timers/timemath.gif)

Would you look at that?!  Comparing time has never been easier!

![lookatit](/posts/pwsh-timers/lookatit.jpg)

Now that we know how to do that, let's use all of this to solve our original problem!

### Putting it all Together

Okay, now that we know how to create a timer and a time span, let's use those to solve the long running calculator job.

You can even use the code from above with modified values![^2]

```powershell
Start-Process -FilePath 'C:\Windows\System32\calc.exe'
Start-Sleep -Seconds 5

$stopWatch = New-Object -TypeName System.Diagnostics.Stopwatch
$timeSpan = New-TimeSpan -Minutes 1 -Seconds 30
$stopWatch.Start()

do
{
    Write-Output -InputObject 'Checking if calculator is running...'
    $calcProcess = Get-Process -Name Calculator -ErrorAction SilentlyContinue
    if ($calcProcess)
    {
        Write-Output -InputObject ('Calculator is running, PID {0}' -f $calcProcess.Id)
    }
    Start-Sleep -Seconds 5
}
until ((-not $calcProcess) -or ($stopWatch.Elapsed -ge $timeSpan))
```

You can see with not much modification you now have a timer condition built into your job.

![diff](/posts/pwsh-timers/finished.png)

Now when you run the code if you don't close the calculator in 1 minute and 30 seconds, the script will stop running.  You could have it throw an exception, or handle this in some other way, but now you have the flexibility to do so!

[^1]:
    That `Start-Sleep` on line 2 _may_ not be needed.  I found my computer was so awesomely powerful and fast that when the first check for the calculator process ran it wasn't fully open yet!  If you're running a Packard Bell from 1996 you might not need that `Start-Sleep`, otherwise just leave it.

[^2]:
    If you want to start the stopwatch automatically on creation there is a static method available to do that.  You would use `$stopWatch = [System.Diagnostics.Stopwatch]::StartNew()` .  Since `New-Object` can't execute a static method this syntax is likely what you'll need to use.


