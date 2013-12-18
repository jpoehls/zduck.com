---
title: "PowerShell Script Boilerplate"
layout: post
categories: powershell
description: "A walkthrough of my boilerplate PowerShell script and batch file wrapper. Includes argument pass-through and exit code bubbling."
---

This post is as much for me as it is for you, my reader. I write a lot of
PowerShell scripts and they tend to follow a certain pattern. This is my
personal boilerplate for PowerShell scripts. Maybe it can help you as well.

## PowerShell Script

    $ErrorActionPreference = "Stop"

    $scriptPath = Split-Path -LiteralPath $(if ($PSVersionTable.PSVersion.Major -ge 30) { $PSCommandPath } else { & { $MyInvocation.ScriptName } })

    $stopwatch = [System.Diagnostics.Stopwatch]::StartNew()

    try
    {
        # TODO: Insert script here...
    }
    finally
    {
        Write-Host "Done!"
        Write-Host "Time to complete: $($stopwatch.Elapsed)"
    }

**Highlights:**

- Sets `$ErrorActionPreference` so that unhandled exceptions will halt
  the script execution. By default, PowerShell will roll on when an exceptions
  is thrown and this statement makes my scripts safe by default.
- Set `$scriptPath` to the directory path of the current script.
  Note that this is different than the working directory.
- Most of the time I want to know how long my script takes to run so I include
  a `$stopwatch` that will always output the elapsed time when the script finishes.

## Batch File Wrapper

    @ECHO OFF

    SET SCRIPTPATH=%~d0%~p0boilerplate-script.ps1

    PowerShell.exe -NoProfile -NonInteractive -NoLogo -ExecutionPolicy Unrestricted -Command "$ErrorActionPreference = 'Stop'; & '%SCRIPTPATH%' %*; EXIT $LASTEXITCODE" 
    EXIT /B %ERRORLEVEL%

PowerShell is still kind of a different beast so I usually like to include a
batch file wrapper for PowerShell scripts that I will be distributing or
that are shared with my team.

**Highlights:**

- Store the full path of the PowerShell script we want to execute in the
  `%SCRIPTPATH%` variable. `%d0%~p0` magic gets the directory path of the
  current batch script. By specifying the full path of the PowerShell script
  like this we can guarantee that it is always executed from the right place
  no matter what your working directory is.
- Run `PowerShell.exe` with:
    - `-NoProfile` to improve startup performance. Scripts you are distributing
      shouldn't rely on anything in your profile anyway.
    - `-NonInteractive` because usually my scripts don't need input from the user.
    - `-ExecutionPolicy Unrestricted` to ensure that the PowerShell script can
      be executed regardless of the machine's default Execution Policy.
    - `-Command` syntax for executing the command ensures that PowerShell
      returns the correct exit code from your script.
      Using `-Command` with `$ErrorActionPreference = 'Stop'` also ensures that
      errors thrown from your script cause PowerShell.exe to return a failing
      exit code (1). [PowerShell is quite buggy when it comes to bubbling exit 
      codes.]({{site.url}}/2012/powershell-batch-files-exit-codes/)
      This is the safest method I've found.
- All of the batch file's arguments are passed through to the PowerShell script.
  _Note that you may have to do some funky stuff to escape special characters in the arguments that you pass to the batch file._

Do you have a boilerplate for scripts that you write? What do you include in yours? Let me know in the comments!