---
title: Using Unix terminal keyboard shortcuts in the Windows command prompt
layout: post
categories: powershell
description: "Using an enum flags property to hold users' roles is just one trick I've learned from @SeiginoRaikou, one of my co-geniuses at InterWorks."
---

Unix terminals have a few basic keyboard shortcuts.

* `CTRL+L` to clear the window
* `CTRL+D` to close the window
* `CTRL+V` to paste text

The Windows command prompt doesn't support these by default but we can fix that with [AutoHotkey](http://www.autohotkey.com).

_If you don't already have it installed, you can follow the steps in [my blog post]({{site.url}}/2011/use-autohotkey-to-remap-your-numpad-keys-to-something-useful/)._

Add this magic to your AutoHotkey script to enable these shortcuts in the Windows command prompt and PowerShell.

<pre>#IfWinActive ahk_class ConsoleWindowClass
^L::SendInput , {Esc}cls{Enter}
^D::SendInput , {Esc}exit{Enter}
^V::SendInput , {Raw}%clipboard%
#IfWinActive</pre>