---
title: "ERRORLEVEL is not %ERRORLEVEL%"
layout: post
categories: windows
description: "ERRORLEVEL is not the same as %ERRORLEVEL%. Use the right one in your command line scripts, or else!"
---

**DO** use `IF ERRORLEVEL 1 ECHO error level is 1 or more` to check for errors
in your batch/command line scripts.

**DO NOT** use `IF %ERRORLEVEL% NEQ 0 ECHO error level is not equal to 0`.

Most of the time `%ERRORLEVEL%` will be what you expect but it isn't guaranteed.
`%ERRORLEVEL%` is simply an environment variable like any other. It just happens to fallback to returning the `ERRORLEVEL` value if an environment variable with that name isn't defined.

This is how I lost about an hour of my morning. Fortunately Google guided me to [this excellent post](http://blogs.msdn.com/b/oldnewthing/archive/2008/09/26/8965755.aspx) and saved
my sanity.

Remember `%ERRORLEVEL%` is not the same as `ERRORLEVEL`.