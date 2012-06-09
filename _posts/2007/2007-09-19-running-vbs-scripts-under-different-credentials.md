---
title: Running VBS scripts under different credentials
layout: post
categories: vbscript windows
---

Today I wrote a VBS script to test out some permission settings on a server. We were debugging a problem and I needed to find out whether different user accounts had permissions to see whether a given file existed. The script was simple enough:

<pre>
Dim fso
Set fso = Server.CreateObject("FileSystemObject")
MsgBox fso.FileExists("\\superserver\path\to\file.txt")
</pre>

Now how do I run that script as someone other than the logged in user? Go to the command line and type:

<pre>
runas /profile /user:domain\username "cscript.exe C:\path\to\script.vbs"
</pre>

When you run that it will prompt for the password of the user account you specified.

Note that if you leave off the `cscript.exe` part you'll get an error about your VBS file not being a valid Win32 application.

The `/profile` part may not be required depending on what your script is doing. I left it out for my simple script.

Visit Microsoft for [more information](http://www.microsoft.com/technet/scriptcenter/resources/qanda/apr06/hey0428.mspx).