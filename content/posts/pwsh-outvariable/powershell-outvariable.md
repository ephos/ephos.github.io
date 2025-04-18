+++
categories = ['PowerShell']
tags = ['Migrated Post', 'Tips and Tricks']
date = '2017-05-29T00:00:00-04:00'
draft = false
title = '-OutVariable, A Cautionary Tale...'
author = 'Bobby'
summary = '-OutVariable, a useful common parameter, but maybe not as straight forward as you always thought!'
description = 'Fun fact, this post has been migrated twice now!'
+++

## "-OutVariable" More Tricky Than You Thought

In PowerShell we are provided common parameters.  A common parameter is a parameter that is available on every Cmdlet or also on any advanced function that is decorated with the `[CmdletBinding()]` attribute before the param block.

One of my favorite common parameters *was* -OutVariable.  It allowed me to type a really long command out and then at the end if I decided I wanted to store it in a variable I could just append the -OutVariable or -ov parameter and then the variable name.

```powershell
Invoke-ExampleCommand -Parameter1 Foo -Parameter2 Bar -Parameter3 Fiz -Parameter4 qux -OutVariable variableName
```

It saved me from having to bounce back to the front of the command using either the Ctrl+LeftArrow or the Home key.  Yes I am literally that lazy, the fewer keystrokes the better!

```powershell
$variableName = Invoke-ExampleCommand -Parameter1 Foo -Parameter2 Bar -Parameter3 Fiz -Parameter4 qux
```

Both commands above accomplish the same thing, but the second one might have required me spending those extra moments getting to the front of the command to store it.

I used -OutVariable for awhile before I learned it had a **dark** secret, it wasn't the friend I knew or first met to save me seconds on storing a variable when doing PowerShell from the shell itself.  It was a wolf.. um, ArrayList in.. err... some other type's clothing.

Let me explain...

I wanted to write a function for some DNS work that accepted a CimSession as a parameter type.  One of my really basic profile functions is a quick wrapper for `New-CimSession` to connect to my internal DNS server and return the CimSession as a variable (*for use with DNS Cmdlets*).  To do this I was using -OutVariable.

To show you what I ran into I set up this little test...

```powershell
#Open a PowerShell CimSession to the DNS server, store it in a variable named 'dnsCimSession'
New-CimSession -ComputerName kappa -Credential $myCredential -OutVariable dnsCimSession

#Test function that accepts a single parameter which needs to be of type CimSession (Microsoft.Management.Infrastructure.CimSession).
#It should just return the CimSession if it runs.
function Test-OutVariable
{
    [CmdletBinding()]
    param
    (
        [Parameter(Mandatory = $true, Position = 0)]
        [CimSession]
        $CimSession
    )
    process
    {
            Write-Output -InputObject $CimSession
    }
}
```

This test function CimSession parameter *should* take our CimSession from our 'dnsCimSession' variable that the New-CimSession returned, right?

![OutVariable1](/posts/pwsh-outvariable/OutVariable1.PNG)

What the?...
 Cannot process argument transformation on parameter 'CimSession'. Cannot convert the "System.Collections.ArrayList" value of type "System.Collections.ArrayList" to type "Microsoft.Management.Infrastructure.CimSession".

I passed it a variable that contained a type of CimSession though, who said anything about an ArrayList?

![OutVariable2](/posts/pwsh-outvariable/OutVariable2.PNG)
![OutVariable3](/posts/pwsh-outvariable/OutVariable3.PNG)

Where does the ArrayList come from?  It looks like a CimSession to me!  The thing is, and quite obviously by now, it's not...  Doing a get type on the variable will reveal this. `$dnsCimSession.GetType()`
![OutVariable4](/posts/pwsh-outvariable/OutVariable4.PNG)

It is an ArrayList.  I did some testing by declaring a new ArrayList and adding some Int32 values to it and sure enough PowerShell's Get-Member returns the ArrayLists containing type rather than the type of ArrayList itself.
![OutVariable5](/posts/pwsh-outvariable/OutVariable5.PNG)

Why does -OutVariable return an ArrayList though?  Lets check the help!

```powershell
Get-Help -Name about_CommonParameters -Detailed
```

> #### Excerpt from Get-Help -Name about_CommonParameters -Detailed
> -OutVariable [+] Alias: ov
>
>    Stores output objects from the command in the specified variable and
>    displays it at the command line.
>
>    To add the output to the variable, instead of replacing any output
>    that might already be stored there, type a plus sign (+) before the
>    variable name.
>
>    For example, the following command creates the $out variable and
>    stores the process object in it:
>
>    <b>&emsp;Get-Process PowerShell -OutVariable out</b>
>
>    The following command adds the process object to the $out variable:
>
>    <b>&emsp;Get-Process iexplore -OutVariable +out</b>
>
>    The following command displays the contents of the $out variable:
>
>    <b>&emsp;$out</b>

Would you look at that!?  The -OutVariable common parameter has an ability to append more values to the same variable.  I've never personally had a use case for this myself.  It seems like the PowerShell developers made this design decision for `-OutVariable` this way in the beginning of PowerShell 1.0.  I did some additional digging and found another person who ran into this that posted on Stack OverFlow as well as a few Github issues filed for it.

- [StackOverFlow Post](https://stackoverflow.com/questions/40666291/using-outvariable-creates-arraylist)
- [GitHub PowerShell Issue 3154](https://github.com/PowerShell/PowerShell/issues/3154)
- [GitHub PowerShell Issue 3773](https://github.com/PowerShell/PowerShell/issues/3154)

It was up for committee review and it was decided that the behavior for these common parameters should be to return the type the user expected rather than an ArrayList.  It looks as if it was added to the PowerShell 6.0.0 milestone, exciting times!  In the meantime I've just stopped using -OutVariable, and if you choose to use it, be aware you're getting an ArrayList back rather than the type you think it will be.

### Update [April 20th, 2018]

Boy heckin' howdy!  I followed up on this sucker and there are some updates to bring you!

>It was up for committee review and it was decided that the behavior for these common parameters should be to return the type the user expected rather than an ArrayList.  It looks as if it was added to the PowerShell 6.0.0 milestone, exciting times!

It was up for committee, an [RFC](https://github.com/PowerShell/PowerShell-RFC/pull/120) was opened up.  If you aren't familiar with an RFC is a "Request for Comments" and its more or less used for everyone to voice their concerns, opinions, design and implementation details on protocol, language, technology etc.

What a doozy of a discussion this one was, and honestly after reading it both sides make a solid point.  As it stands, I don't know what my own opinion is and I do understand that changing it would be a potentially big breaking change for the language for anyone who is using this in their code.

In the end, I want to say that it is a truly amazing time where the community and PowerShell developers can have open discussion in a public forum on the future direction of the language!  Regardless of the outcome, I am happy to live in a time where we have this kind of collaboration in the PowerShell and .Net space!

Update 4-22-2018[^1]

---

[^1]:
    I went through a big update in April 2018, this article was moved as part of that!

