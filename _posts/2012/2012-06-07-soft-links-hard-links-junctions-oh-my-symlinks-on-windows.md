---

title: "Soft links, hard links, junctions, oh my! Symlinks on Windows, a how-to."
layout: default
categories: windows

---

First, a quick definition of terms. There are three kinds of "[symlinks](https://en.wikipedia.org/wiki/Symbolic_link)" on Windows.

* soft links (also called symlinks, or symbolic links)
* hard links
* junctions (a type of soft link only for directories)

Soft links can be created for files or directories.

Hard links can only be created for files.

Both soft and hard links must be created on the same volume as the target. i.e. You can't link something on `C:\` to something on `D:\`.

### Deleting Links

This is where the difference between soft and hard links is most evident.

Deleting the target will cause soft links to stop working. What it points to is gone. Hard links however will keep right on working until you delete the hard link itself. The hard link acts just like the original file, because for all intents and purposes, it is the original file.

### Junctions

Windows also has another type of link just for directories, called Junctions.

Junctions look and act like soft links. The key difference is that they allow you to link directories that are located on different local volumes (but still on the same computer). You can't create a junction to a network location.

## Using MKLINK

Create a soft link to a directory.

    c:\symlink_test> mklink symlink_dir real_dir
    symbolic link created for symlink_dir <<===>> real_dir

Create junction link to a directory.

    c:\symlink_test> mklink /J junction_dir real_dir
    Junction created for junction_dir <<===>> real_dir

Create a soft link to a file.

    c:\symlink_test> mklink symlink_file.txt real_file.txt
    symbolic link created for symlink_file.txt <<===>> real_file.txt

Create a hard link to a file.

    c:\symlink_test> mklink /H hardlink_file.txt real_file.txt
    Hardlink created for hardlink_file.txt <<===>> real_file.txt

What they look like.

    c:\symlink_test> dir
    Volume in drive C is OS
    Volume Serial Number is 7688-08EC

    Directory of c:\symlink_test

    06/07/2012  10:32 AM    <DIR>          .
    06/07/2012  10:32 AM    <DIR>          ..
    06/07/2012  09:51 AM                15 hardlink_file.txt
    06/07/2012  09:59 AM    <JUNCTION>     junction_dir [c:\symlink_test\real_dir]
    06/07/2012  09:47 AM    <DIR>          real_dir
    06/07/2012  09:51 AM                15 real_file.txt
    06/07/2012  10:00 AM    <SYMLINKD>     symlink_dir [real_dir]
    06/07/2012  10:31 AM    <SYMLINK>      symlink_file.txt [real_file.txt]
                   3 File(s)             30 bytes
                   5 Dir(s)  145,497,268,224 bytes free

![Screenshot of folder in Windows Explorer](https://github.com/jpoehls/jpoehls.github.com/raw/master/_posts/2012/2012-06-07-1.png "Screenshot of folder in Windows Explorer")

**Note for PowerShell users:**  
MKLINK isn't an executable that you can just call from PowerShell. You have to call it through the command prompt.

    cmd /c mklink /D symlink_dir real_dir
    
[Read about MSLINK on MSDN.](http://technet.microsoft.com/en-us/library/cc753194(v=WS.10).aspx)

## Using FSUTIL

FSUTIL is another way to create hard links (but not soft links). This is the same as `mklink /H`.

    c:\symlink_test>where fsutil
    C:\Windows\System32\fsutil.exe

    c:\symlink_test> fsutil hardlink create hardlink_file.txt real_file.txt
    Hardlink created for c:\symlink_test\hardlink_file.txt <<===>> c:\symlink_test\real_file.txt

[Read about FSUTIL on MSDN.](http://technet.microsoft.com/en-us/library/cc753059(v=WS.10).aspx)

## Using Junction

Junction is a tool provided by Sysinternals and provides another way to create junctions. Same as `mklink /J`.
It also has some other tools for working with junctions that I won't cover here.

    c:\symlink_test> junction junction_dir real_dir
    Junction v1.06 - Windows junction creator and reparse point viewer
    Copyright (C) 2000-2010 Mark Russinovich
    Sysinternals - www.sysinternals.com

    Created: c:\symlink_test\junction_dir
    Targetted at: c:\symlink_test\real_dir
    
[Download the Junction tool from Sysinternals.](http://technet.microsoft.com/en-us/sysinternals/bb896768.aspx)

--------------------

Read more about hardlinks and junctions [on MSDN](http://msdn.microsoft.com/en-us/library/aa365006%28VS.85%29.aspx).