+++
categories = ['VS Code', 'PowerShell']
tags = ['IDE', 'Debug']
date = '2017-09-10T00:00:00-04:00'
draft = false
title = 'VSCode Debugging - Conditional Breakpoints'
author = 'Bobby'
summary = 'Learn how to use conditional breakpoints in VSCode.  The better you debug, the faster you solve problems!'
description = 'This post was migrated, have you considered using Neovim?'
+++

## Louie, I Think This is the Beginning of a Beautiful Debugging Session.

We all know that conditional love is a bad thing, sometimes conditional contracts depending on which side you're on.  There is however *one* conditional thing we are going to talk about today that is great...  Conditional breakpoints in Visual Studio Code debugging!

#### Background on the Problem

I found myself stepping through a function I was writing to let me just upgrade all my modules which I have installed from an upstream PS Repository, typically the PowerShell Gallery.

*...for those interested, the rough draft.  A little sloppy but im just putzing around today...*

```powershell
function Update-PowerShellModules
{
    [CmdletBinding(SupportsShouldProcess = $true, ConfirmImpact = 'High')]
    param
    (
        # Scope for upgrading modules, AllUsers requires Administrator rights.
        [Parameter(Mandatory = $false)]
        [string]
        [ValidateSet('CurrentUser', 'AllUsers')]
        $Scope = 'CurrentUser'
    )
    begin
    {
        Set-StrictMode -Version Latest
    }
    process
    {
        $loadedModules = Get-Module
        foreach ($module in $loadedModules)
        {
            $moduleLookup = Find-Module -Name $module.Name -ErrorAction SilentlyContinue
            if (-not $moduleLookup)
            {
                Write-Verbose -Message "Module $($module.Name) is either an embedded PowerShell module or has no upstream repository."
                continue
            }
            else
            {
                Write-Verbose -Message "Module $($module.Name) was found in PSRepository, $($moduleLookup.Repository)."
            }

            if ($module.Version -lt $moduleLookup.Version)
            {
                Write-Verbose -Message "Module $($module.Name) is at version $($module.Version), version $($moduleLookup.Version) is available."
                try
                {
                    if ($PSCmdlet.ShouldProcess("Update $($module.Name) to $($moduleLookup.Version)?"))
                    {
                        Install-Module -Name $module.Name -Scope $Scope -Force
                    }
                }
                catch
                {
                    throw
                }
            }
            else
            {
                Write-Verbose -Message "Module $($module.Name) is at version $($module.Version), which is up to date."
            }

            #Clear for the next run of the loop.
            Clear-Variable -Name moduleLookup -Force -ErrorAction SilentlyContinue -WhatIf:$false
        }
    }
}
```

Now, there are a fair deal of modules that come with PowerShell itself *(Microsoft.PowerShell.Management, Microsoft.PowerShell.Security etc..)* or modules that are installed separately with tools like RSAT *(ActiveDirectory, Hyper-V, etc.. )*.  I thought about trying to find a way to exclude these while including Pester, PackageManagement, PowerShellGet, etc.. but for the time being just decided to let them get checked and skip them if no action could be taken.

Before you suggest just splitting the `$env:PSModulePath` variable and checking locations that aren't in *C:\Windows\System32\WindowsPowerShell\v1.0\Modules\\*, I have been trying to write all my PowerShell to be compatible with PowerShell Core 6.0 so that approach won't work for that.

#### Onto the Debugging Business

I love debugging, maybe its weird, but I really do enjoy it.  If you want to get started debugging in VSCode check this [page](https://code.visualstudio.com/docs/editor/debugging) out.

So I dive into debugging my function, add a breakpoint to it and hit `F11` to step into the function.  As I continue to step through it I of course hit my `foreach` loop.  Here is the part thats kind of a bummer.  The first 3 items in my `$loadedModules` list are those built-in PowerShell modules.  Ugh, this means each time I debug I need to step through these 3, over, and over, and over...  What a serious waste of time!

![Debug1](/posts/vscode-debug-breakpoint/Debug1.PNG)

Conditional breakpoints to the rescue, these little guys are really cool.  They are user defined breakpoints where if on a certain line, the condition is met, the debugger pauses and lets you do your debuggin' thang!

To add a conditional breakpoint you can use the menu or in the debug pane of VS Code right click where you would normally add a breakpoint and select **Add Conditional Breakpoint...**.

![Debug2](/posts/vscode-debug-breakpoint/Debug2.PNG)

![Debug3](/posts/vscode-debug-breakpoint/Debug3.PNG)

A special line will showup on the line you started on with two options, **Hit Count** and **Expression**.  A breakdown of the choices...

- **Hit Count:** breaks/pauses the debugger after the breakpoint has been hit ***X*** number of times, where ***X*** is the number you assign in the conditional breakpoint.
- **Expression:** breaks/pauses the debugger when a condition is met.  In PowerShell our condition could be something like `$x -le 9` or `$variable -eq 'Foo'`.

Lets add one, since we want to see how this puppy runs when processing the Pester module we'll add a conditional breakpoint for that specifically.  We write it as if we were going to write any other type of PowerShell conditional statement.

![Debug4](/posts/vscode-debug-breakpoint/Debug4.PNG)

...and hit enter.  Now we see it and the red breakpoint circle has a special `=` in it indicating its a conditional breakpoint!  If you go crazy with this you can hover over them to see what they are (as well as still see them listed with all the other normal breakpoints in the Breakpoints section in Debug pane).

![Debug5](/posts/vscode-debug-breakpoint/Debug5.PNG)

Now, rather than having a single normal breakpoint and stepping through the loop multiple times, we can pause the debugger the exact moment that `$module.Name -eq 'Pester'` evaluates to `$true`.

![Debug6](/posts/vscode-debug-breakpoint/Debug6.PNG)

Now it doesn't matter what position Pester ends up in the list, we'll pause when we hit it regardless of if its at `[0]` or `[101]`.  Pretty nifty.

#### A Note on Debugging PowerShell

If you're new to debugging PowerShell it can be a little intimidating at first, but it will honestly help you solve those really strange *"What the hell is my code even doing???"* and *"How is this happening?"* kind of problems.  Check out the link above on how to use the VS Code debugger, it's honestly never been easier than now to debug PowerShell.  Even if you only break it out for those complicated super stumpers, give it a try you won't regret it!  9 out of 10 dentists recommend you debug more, it's for your own good.

Update 4-22-2018[^1]

---

[^1]:
    I went through a big update in April 2018, this article was moved as part of that!

