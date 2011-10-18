---

title: Tortoise SVN PowerShell helper function
layout: default
categories: powershell svn

---

Do you work from the command line? Use PowerShell? SVN?

While the SVN command line client is certainly usable, there are definitely times where a GUI is more convenient. Specifically during a commit where you want to easily check/uncheck files and view diffs.

I got tired of having to open an explorer window to get into Tortoise SVN's commit screen so I wrote this helper function.

Put this into your profile and you can start typing `tsvn commit` (or just `tsvn`) to open the Tortoise SVN commit dialog. Enjoy!

    # Helper function for opening the Tortoise SVN GUI from a PowerShell prompt.
    # Put this into your PowerShell profile.
    # Ensure Tortoise SVN is in your PATH (usually C:\Program Files\TortoiseSVN\bin) 
    function Svn-Tortoise([string]$Command = "commit") {
    <#
    .SYNOPSIS
      Launches TortoiseSVN with the given command.
      Opens the commit screen if no command is given.
      
      List of supported commands can be found at:
      http://tortoisesvn.net/docs/release/TortoiseSVN_en/tsvn-automation.html
    #>
      TortoiseProc.exe /command:$Command /path:"$pwd"
    }
    Set-Alias tsvn "Svn-Tortoise"

_I also cross-posted this to [my company blog](http://www.interworks.com/blogs/jpoehls/2011/10/12/tortoise-svn-powershell-helper-function "Read this post on the InterWorks blog.")._