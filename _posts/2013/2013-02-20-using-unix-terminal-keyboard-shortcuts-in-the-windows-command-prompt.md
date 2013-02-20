---
title: Using Unix terminal keyboard shortcuts in the Windows command prompt
layout: post
categories: powershell
description: "Using an enum flags property to hold users' roles is just one trick I've learned from @SeiginoRaikou, one of my co-geniuses at InterWorks."
---

Do you miss the common shortcut keys available in Unix terminals?
Clearing the window with `CTRL+L`, exiting with `CTRL+D`,
or how about being able to paste text with `CTRL+V`?

[AutoHotkey](http://www.autohotkey.com/) to the rescue.
If you don't already have it installed, you can follow the steps in [my blog post](http://zduck.com/2011/use-autohotkey-to-remap-your-numpad-keys-to-something-useful/).

Add this magic to your AutoHotkey script to enable these shortcuts in the Windows command prompt and PowerShell.

<pre>#IfWinActive ahk_class ConsoleWindowClass
^L::SendInput , {Esc}cls{Enter}
^D::SendInput , {Esc}exit{Enter}
^V::SendInput , {Raw}%clipboard%
#IfWinActive</pre>