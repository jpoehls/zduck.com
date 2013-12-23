---
title: "Take your PowerShell profile everywhere with Dropbox"
layout: post
categories: powershell
iwpost: http://www.interworks.com/blogs/jpoehls/2011/03/25/scripting-tips-take-your-powershell-profile-everywhere-dropbox
---

I'll cover these prerequisites in brief, but for this tip I'm going to assume a few things:

* You are running Windows and have [PowerShell](http://msdn.microsoft.com/en-us/library/aa973757.aspx) installed.
* You are familiar enough with PowerShell to care about your [$PROFILE](http://msdn.microsoft.com/en-us/library/bb613488.aspx).

### What is PowerShell?

In short, PowerShell is a new scripting language from Microsoft designed to replace the use of batch files and VBScript for administrating and automating Windows machines. The language leverages the .NET Framework such that anything you can do with .NET you can do with PowerShell. Typically you will be using PowerShell from the command line but there are lots of other ways to use it.

### What is a PowerShell profile?

PowerShell has the concept of a script that will run each time you launch PowerShell. This is a really handy place to put any functions that you want to be available all the time. Initialize some variables, create some aliases, anything you do here will be there when you need it.

### What are we doing here?

I'm going to give you one possible solution for keeping all of your scripts and settings (i.e. your profile) in sync no matter which computer you are on. No copy/pasting. No more wearing that USB dongle around your neck - seriously, that just _screams_ geek.

Most steps have a sample PowerShell command that will do the trick. So fire up PowerShell and put on your scripting hat 'cuz here we go!

1. **[Get Dropbox](http://db.tt/i03IdN4).**  
Among other things, Dropbox makes keeping your files in sync across computers dead simple. If you aren't using it you should be. It's just that good.

2. **Create a folder in Dropbox to hold your scripts.**

    <pre>
    PS&gt; New-Item $dropbox\scripts</pre>

	_Anywhere you see `$dropbox`, assume this to be the path to your Dropbox folder. By default on Windows 7 this is `%USERPROFILE%\Dropbox`._

3. **Copy your profile `ps1` file into your new scripts folder.**

    <pre>
    PS&gt; Copy-Item $PROFILE $dropbox\scripts\profile.ps1</pre>

4. **Edit your profile `ps1` file (the original one) and "dot source" the one in your Dropbox folder.**

    <pre>
	PS&gt; ". $dropbox\scripts\profile.ps1" &gt; $PROFILE</pre>

**Pro Tip!** As an alternative to dot sourcing your profile from Dropbox, you could instead [create a symlink]({{site.url}}/2012/soft-links-hard-links-junctions-oh-my-symlinks-on-windows/) to it.
    
<pre>
PS&gt; /cmd /c mklink `"$PROFILE`" `"$dropbox\scripts\profile.ps1`"
</pre>

{{site.powershell_book_ad}}