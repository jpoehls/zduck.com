---
title: Using bookmarks to quickly navigate in PowerShell
layout: post
categories: powershell
description: "There are a few folders that I spend a lot of time and it is surprisingly slow to type `cd c:\\path\\to\\my\\project` many times a day. I solved this by creating a module that allows me to bookmark directories and navigate to them using aliases. Simply put, I can shorten the previous command to `g project`. All you have to do is import the module in your profile and initialize your bookmarks."
---

There are a few folders that I spend a lot of time and it is surprisingly slow to type `cd c:\path\to\my\project` many times a day. I solved this by creating a module that allows me to bookmark directories and navigate to them using aliases. Simply put, I can shorten the previous command to `g project`.

Usage looks like this:

<pre>
<b>Set-Bookmark</b> &lt;name&gt; &lt;path&gt;
<b>Get-Bookmark</b> <i># to list all</i>
<b>Get-Bookmark</b> &lt;name&gt;
<b>Remove-Bookmark</b> &lt;name&gt;
<b>Clear-Bookmarks</b>
<b>Invoke-Bookmark</b> &lt;name&gt;
</pre>

I alias `Invoke-Bookmark` as `g` for speed.

All you have to do is import the module in your profile and initialize your bookmarks.

<pre data-language="powershell">
# Microsoft.PowerShell_profile.ps1

Import-Module bookmarks.psm1
Set-Alias g Invoke-Bookmark

Set-Bookmark project c:\path\to\my\project
</pre>

Here is the module. Happy bookmarking!

<pre data-language="powershell">
# bookmarks.psm1
# Exports: Set-Bookmark, Get-Bookmark, Remove-Bookmark, Clear-Bookmarks, Invoke-Bookmark

# holds hash of bookmarked locations
$_bookmarks = @{}

function Get-Bookmark() {
  Write-Output ($_bookmarks.GetEnumerator() | sort Name)
}

function Remove-Bookmark($key) {
&lt;#
.SYNOPSIS
  Removes the bookmark with the given key.
#&gt;
  if ($_bookmarks.keys -contains $key) {
    $_bookmarks.remove($key)
  }
}

function Clear-Bookmarks() {
&lt;#
.SYNOPSIS
  Clears all bookmarks.
#&gt;
  $_bookmarks.Clear()
}

function Set-Bookmark($key, $location) {
&lt;#
.SYNOPSIS
  Bookmarks the given location or the current location (Get-Location).
#&gt;
  # bookmark the current location if a specific path wasn't specified
  if ($location -eq $null) {
    $location = (Get-Location).Path
  }

  # make sure we haven't already bookmarked this location (no need to clutter things)
  if ($_bookmarks.values -contains $location) {
    Write-Warning ("Already bookmarked as: " + ($_bookmarks.keys | where { $_bookmarks[$_] -eq $location }))
    return
  }

  # if no specific key was specified then auto-set the key to the next bookmark number
  if ($key -eq $null) {
    $existingNumbers = ($_bookmarks.keys | Sort-Object -Descending | where { $_ -is [int] })
    if ($existingNumbers.length -gt 0) {
      $key = $existingNumbers[0] + 1
    }
    else {
      $key = 1
    }
  }

  $_bookmarks[$key] = $location
}

function Invoke-Bookmark($key) {
&lt;#
.SYNOPSIS
  Goes to the location specified by the given bookmark.
#&gt;
  if ([string]::IsNullOrEmpty($key)) {
    Get-Bookmarks
    return
  }

  if ($_bookmarks.keys -contains $key) {
    Push-Location $_bookmarks[$key]
  }
  else {
    Write-Warning "No bookmark set for the key: $key"
  }
}

Export-ModuleMember Get-Bookmark, Remove-Bookmark, Clear-Bookmarks, Set-Bookmark, Invoke-Bookmark
</pre>