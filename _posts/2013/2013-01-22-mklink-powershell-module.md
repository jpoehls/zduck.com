---
title: An MKLINK PowerShell Module
layout: post
categories: powershell
description: MKLINK is a very useful utility on Windows. Unfortunately there aren't any native PowerShell functions that replace it so we still have to shell out to the command prompt. This can be made less painful with a helper function that will pipe all arguments to the MKLINK command. Personally though, I prefer to use native PowerShell functions whenever possible so I put together a module that provides wrappers for the key functionality of MKLINK.
---

[MKLINK][1] is a very useful utility on Windows. Unfortunately there aren't any native PowerShell functions that replace it so we still have to shell out to the command prompt.

This can be made less painful with a helper function that will pipe all arguments to the MKLINK command.

<pre data-language="powershell">
function mklink { cmd /c mklink $args }
</pre>

Personally though, I prefer to use native PowerShell functions whenever possible so I put together a module that provides wrappers for the key functionality of MKLINK.

Usage looks like this:

<pre>
<b>New-Symlink</b> &lt;link_path&gt; &lt;target_path&gt; &lt;force?&gt;
<b>New-Hardlink</b> &lt;link_path&gt; &lt;target_path&gt; &lt;force?&gt;
<b>New-Junction</b> &lt;link_path&gt; &lt;target_path&gt; &lt;force?&gt;
<b>mklink</b> &lt;args&gt; <i># raw pipe to mklink</i>
</pre>

Import this module in your profile and you'll have it whenever you need it.

<pre data-language="powershell">
# Microsoft.PowerShell_profile.ps1

Import-Module mklink.psm1
</pre>

<pre data-language="powershell">
# mklink.psm1
# Exports: New-Symlink, New-Hardlink, New-Junction, mklink

function Force-Resolve-Path {
    &lt;#
    .SYNOPSIS
        Calls Resolve-Path but works for files that don't exist.
    .REMARKS
        From http://devhawk.net/2010/01/21/fixing-powershells-busted-resolve-path-cmdlet/
    #&gt;
    param (
        [string] $FileName
    )
    
    $FileName = Resolve-Path $FileName -ErrorAction SilentlyContinue `
                                       -ErrorVariable _frperror
    if (-not($FileName)) {
        $FileName = $_frperror[0].TargetObject
    }
    
    return $FileName
}

function New-Symlink {
    &lt;#
    .SYNOPSIS
        Creates a symbolic link.
    #&gt;
    param (
        [Parameter(Position=0, Mandatory=$true)]
        [string] $Link,
        [Parameter(Position=1, Mandatory=$true)]
        [string] $Target,
        [Parameter(Position=2)]
        [switch] $Force
    )

    Invoke-MKLINK -Link $Link -Target $Target -Symlink -Force $Force
}

function New-Hardlink {
    &lt;#
    .SYNOPSIS
        Creates a hard link.
    #&gt;
    param (
        [Parameter(Position=0, Mandatory=$true)]
        [string] $Link,
        [Parameter(Position=1, Mandatory=$true)]
        [string] $Target,
        [Parameter(Position=2)]
        [switch] $Force
    )

    Invoke-MKLINK -Link $Link -Target $Target -HardLink -Force $Force
}

function New-Junction {
    &lt;#
    .SYNOPSIS
        Creates a directory junction.
    #&gt;
    param (
        [Parameter(Position=0, Mandatory=$true)]
        [string] $Link,
        [Parameter(Position=1, Mandatory=$true)]
        [string] $Target,
        [Parameter(Position=2)]
        [switch] $Force
    )

    Invoke-MKLINK -Link $Link -Target $Target -Junction -Force $Force
}

function Invoke-MKLINK {
    &lt;#
    .SYNOPSIS
        Creates a symbolic link, hard link, or directory junction.
    #&gt;
    [CmdletBinding(DefaultParameterSetName = "Symlink")]
    param (
        [Parameter(Position=0, Mandatory=$true)]
        [string] $Link,
        [Parameter(Position=1, Mandatory=$true)]
        [string] $Target,

        [Parameter(ParameterSetName = "Symlink")]
        [switch] $Symlink = $true,
        [Parameter(ParameterSetName = "HardLink")]
        [switch] $HardLink,
        [Parameter(ParameterSetName = "Junction")]
        [switch] $Junction,
        [Parameter()]
        [bool] $Force
    )
    
    # Resolve the paths incase a relative path was passed in.
    $Link = (Force-Resolve-Path $Link)
    $Target = (Force-Resolve-Path $Target)

    # Ensure target exists.
    if (-not(Test-Path $Target)) {
        throw "Target does not exist.`nTarget: $Target"
    }

    # Ensure link does not exist.
    if (Test-Path $Link) {
        if ($Force) {
            Remove-Item $Link -Recurse -Force
        }
        else {
            throw "A file or directory already exists at the link path.`nLink: $Link"
        }
    }

    $isDirectory = (Get-Item $Target).PSIsContainer
    $mklinkArg = ""

    if ($Symlink -and $isDirectory) {
        $mkLinkArg = "/D"
    }

    if ($Junction) {
        # Ensure we are linking a directory. (Junctions don't work for files.)
        if (-not($isDirectory)) {
            throw "The target is a file. Junctions cannot be created for files.`nTarget: $Target"
        }

        $mklinkArg = "/J"
    }

    if ($HardLink) {
        # Ensure we are linking a file. (Hard links don't work for directories.)
        if ($isDirectory) {
            throw "The target is a directory. Hard links cannot be created for directories.`nTarget: $Target"
        }

        $mkLinkArg = "/H"
    }

    # Capture the MKLINK output so we can return it properly.
    # Includes a redirect of STDERR to STDOUT so we can capture it as well.
    $output = cmd /c mklink $mkLinkArg `"$Link`" `"$Target`" 2>&1

    if ($lastExitCode -ne 0) {
        throw "MKLINK failed. Exit code: $lastExitCode`n$output"
    }
    else {
        Write-Output $output
    }
}

function mklink {
    &lt;#
    .SYNOPSIS
        Helper function for calling mklink directly.
        All arguments are piped through to mklink.
        
        Hat tip to http://stackoverflow.com/questions/894430/powershell-hard-and-soft-links#comment9823010_5549583
    #&gt;
    cmd /c mklink $args
}

Export-ModuleMember New-Symlink, New-Hardlink, New-Junction, mklink
</pre>

I've heard it rumored that the [PowerShell Community Extensions](http://pscx.codeplex.com/) project has a lot of these functions as well, and more. It might be worth looking into if you want a more robust solution.

  [1]: http://technet.microsoft.com/en-us/library/cc753194(v=WS.10).aspx