---
title: Killing Processes In Disconnected Terminal Service Sessions
layout: post
categories: powershell windows-server
iwpost: http://www.interworks.com/blogs/jpoehls/2012/12/12/killing-processes-disconnected-terminal-service-sessions
---

Usually it is best to configure your terminal service policy to log out disconnected sessions, however there may be cases where you only want to kill specific applications when the user disconnects.

This is easy to do with a PowerShell script that you run periodically as a scheduled task. The processes won't be killed _immediately_ when the user disconnects but will be killed shortly after.

This script will, by default, only kill the OUTLOOK processes in disconnected sessions after waiting 15 seconds for it to gracefully close. It is trivial to tweak the script to kill whichever processes you care about.

<pre data-language="powershell">
# kill-processes.ps1

# This script supports being run with -WhatIf and -Confirm parameters.
[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='Medium')]
param (
    # Regex of the states that should be included in the process killing field.
    [string]$IncludeStates = '^(Disc)$', # Only DISCONNECTED sessions by default.
    # Regex of the processes to kill
    [string]$KillProcesses = '^(OUTLOOK)$', # Only OUTLOOK by default.
    [int]$GracefulTimeout = 15 # Number of seconds to wait for a graceful shutdown before forcefully closing the program.
)

function Get-Sessions
{
    # `query session` is the same as `qwinsta`

    # `query session`: http://technet.microsoft.com/en-us/library/cc785434(v=ws.10).aspx

    # Possible session states:
    &lt;#
    http://support.microsoft.com/kb/186592
    Active. The session is connected and active.
    Conn.   The session is connected. No user is logged on.
    ConnQ.  The session is in the process of connecting. If this state
            continues, it indicates a problem with the connection.
    Shadow. The session is shadowing another session.
    Listen. The session is ready to accept a client connection.
    Disc.   The session is disconnected.
    Idle.   The session is initialized.
    Down.   The session is down, indicating the session failed to initialize correctly.
    Init.   The session is initializing.
    #&gt;

    # Snippet from http://poshcode.org/3062
    # Parses the output of `qwinsta` into PowerShell objects.
    $c = query session 2>&amp;1 | where {$_.gettype().equals([string]) }

    $starters = New-Object psobject -Property @{"SessionName" = 0; "Username" = 0; "ID" = 0; "State" = 0; "Type" = 0; "Device" = 0;};
     
    foreach($line in $c) {
         try {
             if($line.trim().substring(0, $line.trim().indexof(" ")) -eq "SESSIONNAME") {
                $starters.Username = $line.indexof("USERNAME");
                $starters.ID = $line.indexof("ID");
                $starters.State = $line.indexof("STATE");
                $starters.Type = $line.indexof("TYPE");
                $starters.Device = $line.indexof("DEVICE");
                continue;
            }
           
            New-Object psobject -Property @{
                "SessionName" = $line.trim().substring(0, $line.trim().indexof(" ")).trim(">")
                ;"Username" = $line.Substring($starters.Username, $line.IndexOf(" ", $starters.Username) - $starters.Username)
                ;"ID" = $line.Substring($line.IndexOf(" ", $starters.Username), $starters.ID - $line.IndexOf(" ", $starters.Username) + 2).trim()
                ;"State" = $line.Substring($starters.State, $line.IndexOf(" ", $starters.State)-$starters.State).trim()
                ;"Type" = $line.Substring($starters.Type, $starters.Device - $starters.Type).trim()
                ;"Device" = $line.Substring($starters.Device).trim()
            }
        } catch {
            throw $_;
            #$e = $_;
            #Write-Error -Exception $e.Exception -Message $e.PSMessageDetails;
        }
    }
}

# Helper function for getting the singular or plural form of
# a word based on the given count.
# Because we want proper log messages.
function Get-ProperWord([string]$singularWord, [int]$count) {
    if ($count -eq 0 -or $count -gt 1) {
        if ($singularWord.EndsWith("s")) {
            return "$($singularWord)es";
        }
        else {
            return "$($singularWord)s";
        }
    }
    else {
        return $singularWord;
    }
}

# Get a list of all terminal sessions that are in the state we care about.
$IncludedSessions = Get-Sessions `
                        | Where { $_.State -match $IncludeStates } `
                        | Select -ExpandProperty ID

# Get a list of all processes in one of those terminal sessions
# that match a process we want to kill.
$SessionProcesses = $IncludedSessions `
    | % { $id = $_;
          Get-Process `
            | Where { $_.SessionID -eq $id -and $_.Name -match $KillProcesses } }

# Get some words to use in log output.
$wordSecond = $(Get-ProperWord 'second' $GracefulTimeout)
$wordProcess = $(Get-ProperWord 'process' $SessionProcesses.Length)

if ($SessionProcesses.Length -gt 0) {
    # Initiate a graceful shutdown of the processes.
    # http://powershell.com/cs/blogs/tips/archive/2010/05/27/stopping-programs-gracefully.aspx
    Write-Output "Gracefully closing $($SessionProcesses.Length) $wordProcess"
    $SessionProcesses `
        | % { if ($PSCmdlet.ShouldProcess("$($_.Name) ($($_.Id))", "CloseMainWindow")) { $_.CloseMainWindow() } } `
        | Out-Null

    # Wait X seconds for the programs to close gracefully.
    Write-Output "Waiting $GracefulTimeout $wordSecond for the $wordProcess to close"
    if ($GracefulTimeout -gt 0) {
        if ($PSCmdlet.ShouldProcess("Current Process", "Start-Sleep")) {
            Start-Sleep -Seconds $GracefulTimeout
        }
    }

    # Force any remaining processes to close, the hard way.
    Write-Output "Forcefully closing any remaining processes"
    $SessionProcesses `
        | Where { $_.HasExited -ne $true } `
        | Stop-Process -ErrorAction SilentlyContinue
}
else {
    Write-Output "No processes to close"
}
</pre>

{{site.powershell_book_ad}}