---
title: PowerShell, batch files, and exit codes. Recipes & Secrets.
layout: post
categories: powershell
---

## TL;DR;

`PowerShell.exe` doesn't return correct exit codes when using the `-File` option. Use `-Command` instead.

[This](#bat-wrapper) is a batch file wrapper for executing PowerShell scripts. It forwards arguments to PowerShell, and correctly bubbles up the exit code.

You can use [black magic](#black-magic) to include spaces and quotes in the arguments you pass through the batch file wrapper to PowerShell.

## PowerShell

PowerShell is a great scripting environment, and it is my preferred tool for writing build scripts for .NET apps. Exit codes are vital in build scripts because they are how your Continuous Integration server knows whether the build passed, or failed.

This is a <strike>quick</strike> tour of working with exit codes in PowerShell scripts, and batch files. I'm including batch files because they are often necessary to wrap the execution of your PowerShell scripts.

Let's start easy. Say you need to run a command line app, or batch file from your PowerShell script. How can you check the exit code of that process?

<pre data-language="powershell">
# script.ps1

cmd /C exit 1
Write-Host $LastExitCode    # 1
</pre>

`$LastExitCode` is a special variable that holds the exit code of the last Windows based program that was run. So says [the documentation](http://technet.microsoft.com/en-us/library/dd347675.aspx).

Remember though, `$LastExitCode` doesn't do squat for PowerShell commands. Use `$?` for that.

<pre data-language="powershell">
# script.ps1

Get-ChildItem "C:\"
Write-Host $?    # True

Get-ChildItem "Z:\some\non-existant\path"
Write-Host $?    # False
</pre>

Anytime you run an external command like this, you need to check the exit code and throw an exception if needed. Otherwise the PowerShell script will keep right on trucking after a failure.

<pre data-language="powershell">
# script.ps1

cmd /C exit 1
if ($LastExitCode -ne 0) {
    throw "Command failed with exit code $LastExitCode."
}
Write-Host "You'll never see this."
</pre>

Writing these assertions all the time will get old. Fortunately you can use a helper function, like this one found in the [excellent psake project](http://github.com/psake/psake).

<pre data-language="powershell">
# script.ps1

function Exec
{
    [CmdletBinding()]
    param (
        [Parameter(Position=0, Mandatory=1)]
        [scriptblock]$Command,
        [Parameter(Position=1, Mandatory=0)]
        [string]$ErrorMessage = "Execution of command failed.`n$Command"
    )
    &amp; $Command
    if ($LastExitCode -ne 0) {
        throw "Exec: $ErrorMessage"
    }
}

Exec { cmd /C exit 1 }
Write-Host "You'll never see this."
</pre>

The `throw` keyword is how you generate a terminating error in PowerShell. It will, _sometimes_ cause your PowerShell script to return a failing exit code (1). Wait, when does it _not_ cause a failing exit code, you ask? Ah, this is where PowerShell's warts start to show. Let me demonstrate some scenarios.

<pre data-language="powershell">
# broken.ps1

throw "I'm broken"
</pre>

**From the PowerShell command prompt:**

<pre>
PS&gt; .\broken.ps1
<span style="color: red;">I'm broken.
At C:\broken.ps1:1 char:6
+ throw &lt;&lt;&lt;&lt;  "I'm broken."
    + CategoryInfo          : OperationStopped: (I'm broken.:String) [], RuntimeException
    + FullyQualifiedErrorId : I'm broken.</span>
    
PS&gt; $LastExitCode
1
</pre>

**From the Windows command prompt:**

<pre>
&gt; PowerShell.exe -NoProfile -NonInteractive -ExecutionPolicy unrestricted -Command ".\script.ps1"
<span style="color: red;">I'm broken.
At C:\code\pserrorexit\broken.ps1:1 char:6
+ throw &lt;&lt;&lt;&lt;  "I'm broken."
    + CategoryInfo          : OperationStopped: (I'm broken.:String) [], RuntimeException
    + FullyQualifiedErrorId : I'm broken.</span>
    
&gt; echo %errorlevel%
1
</pre>

That worked too. Good.

**Again, from the Windows command prompt:**

<pre>
&gt; PowerShell.exe -NoProfile -NonInteractive -ExecutionPolicy unrestricted -File ".\script.ps1"
<span style="color: red;">I'm broken.
At C:\code\pserrorexit\broken.ps1:1 char:6
+ throw &lt;&lt;&lt;&lt;  "I'm broken."
    + CategoryInfo          : OperationStopped: (I'm broken.:String) [], RuntimeException
    + FullyQualifiedErrorId : I'm broken.</span>

&gt; echo %errorlevel%
0
</pre>

Whoa! We still saw the error, but PowerShell returned a passing exit code. What the heck?! Yes, this is the wart.

`-File` allows you to pass in a script for PowerShell to execute, however terminating errors in the script will not cause PowerShell to return a failing exit code. I have no idea why this is the case. If you know why, please share!

A workaround is to add a `trap` statement to the top of your PowerShell script. (Thanks, Chris Oldwood, for [pointing this out](http://chrisoldwood.blogspot.com/2011/05/powershell-throwing-exceptions-exit.html)!)

<pre data-language="powershell">
# broken.ps1

trap
{
    Write-Error $_
    exit 1
}
throw "I'm broken."
</pre>

**From the Windows command prompt:**

<pre>
&gt; PowerShell.exe -NoProfile -NonInteractive -ExecutionPolicy unrestricted -File ".\script.ps1"
<span style="color: red;">C:\broken.ps1 : I'm broken.
    + CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,broken.ps1</span>

&gt; echo %errorlevel%
1
</pre>

Notice that we got the correct exit code this time, but our error output didn't include as much detail. Specifically, we didn't get the line number of the error like we were getting in the previous tests. So it isn't a perfect workaround.

**Remember.** Use `-Command` instead of `-File` whenever possible. If for some reason you must use `-File`, or your script needs to support being run that way, then use the trap workaround above.

_Note: This was tested under PowerShell v2, on Windows 7 (x64)._

## Batch files

I mentioned early on that it is often necessary to wrap the execution of your PowerShell script in a batch file. Some common reasons for this might be:

* You want users of your script to be able to double-click to run it.
* Your build runner doesn't support execution of PowerShell scripts directly.

Whatever the reason, writing a batch file wrapper for a PowerShell script is easy. You just need to make sure that your batch file properly returns the exit code from PowerShell. Otherwise, your PowerShell script might fail and your batch file would return a successful exit code (0).

<a id="bat-wrapper"> </a>

This is a safe template for you to use. Bookmark it.

<pre>
:: script.bat

@ECHO OFF
PowerShell.exe -NoProfile -NonInteractive -ExecutionPolicy unrestricted -Command "&amp; %~d0%~p0%~n0.ps1" %*
EXIT /B %errorlevel%
</pre>

This wrapper will execute the PowerShell script with the same file name (i.e. `script.ps1` if the batch file is named `script.bat`), and then exit with the same code that PowerShell exited with. It will also forward any arguments passed to the batch file, to the PowerShell script.

Let's test it out.

<pre data-language="powershell">
# script.ps1

param($Arg1, $Arg2)
Write-Host "Arg 1: $Arg1"
Write-Host "Arg 2: $Arg2"
</pre>

**From the Windows command prompt:**

<pre>
&gt; script.bat happy scripting
Arg 1: happy
Arg 2: scripting
</pre>

What if we want "happy scripting" to be passed as a single argument?

<pre>
&gt; script.bat "happy scripting"
Arg 1: happy
Arg 2: scripting
</pre>

<a id="black-magic"> </a>

Well that didn't work at all. This is the secret recipe.

<pre>
&gt; script.bat "'Happy scripting with single '' and double \" quotes!'"
Arg 1: Happy scripting with single ' and double " quotes!
Arg 2:
</pre> 

Please don't ask me to explain this black magic, I only know that it works. Much credit to [this StackOverflow question](http://stackoverflow.com/questions/6714165/powershell-stripping-double-quotes-from-command-line-arguments) for helping me solve this!

For comparison, here is how you would do it if you were executing the script from PowerShell, without using the batch file wrapper.

**From the PowerShell command prompt:**

<pre>
PS&gt; .\script.ps1 happy scripting
Arg 1: happy
Arg 2: scripting

PS&gt; .\script.ps1 "Happy scripting with single ' and double `" quotes included!"
Arg 1: Happy scripting with single ' and double " quotes included!
Arg 2: 
</pre>

That's all folks!