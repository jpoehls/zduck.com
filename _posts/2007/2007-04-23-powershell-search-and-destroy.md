---
title: Recursively deleting files with PowerShell
redirect_to: http://joshua.poehls.me/2007/powershell-search-and-destroy
layout: post
categories: powershell
---

So out of all the things I don't know, PowerShell is the one thing that made it to the top of my To Learn stack. Last week I ordered [PowerShell In Action](http://www.amazon.com/Windows-PowerShell-Action-Bruce-Payette/dp/1932394907/ref=pd_bbs_sr_1/102-7531966-1417704?ie=UTF8&s=books&qid=1177382702&sr=8-1), it hasn't arrived yet but I've already read about 100 pages out of the PDF (don't ask). The book is very well written and describes the core concepts of PowerShell very well.

Having (almost) never written a batch file in my life it is taking a bit for me to wrap my head around the basics but so far I love it. Compiling C# apps to do simple tasks like iterate directory structures and process files has always annoyed me. There is just too much overhead in having to compile and manage both the source code an executable when you only need the code once (until tomorrow comes).

I just want to create a script and run it. No compilation, no exe. Oh, and since I've been completely spoiled by the .NET Framework I want access to all that too. PowerShell fits the bill.

My first task. Find all files with an extension of `*.bak` on my `C:\` drive and delete them.

<pre>
#  move to C: drive
cd c:\

#  list all files in C:
dir

#  list all *.BAK files in C:
dir *.BAK

#  do it recursively
dir -recurse -filter *.BAK
</pre>

Ok, so now that I've got PowerShell spinning through my entire `C:\` drive, how do I stop it? You can always kill the process but I was in a pretty good mood today and decided to be nice. This is likely to be one of the most important things you learn about PowerShell so remember it well...

<pre>
CTRL+C
</pre>

That will halt execution of whatever PowerShell is doing and get you back to a prompt. Very handy for people like me who like tend to hit `[Enter]` before thinking about how long it might take to run.

> **TIP:** `dir` (along with `ls`) is just an alias for the `Get-ChildItem` command.  
> You can get the full syntax for this command by running `Get-Help dir`.

And if you want to delete all those `*.BAK` files you just found? Just pipe your `dir` to `del` (which is an alias for `Remove-Item`).

<pre>
dir -recurse -filter *.BAK | del
</pre>

{{site.powershell_book_ad}}