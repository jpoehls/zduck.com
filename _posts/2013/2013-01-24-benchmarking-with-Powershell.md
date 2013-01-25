---
title: Benchmarking scripts and programs with PowerShell
layout: post
categories: powershell
description: "UNIX has the `time` command that will measure the execution time of a program. Timings are returned for the wall clock and CPU time. PowerShell's equivalent of `time` is `Measure-Command`. Unfortunately it only measures wall clock time. It also swallows the output of the command you are measuring."
---

UNIX has the `time` command that will measure the execution time of a program. Timings are returned for the wall clock and CPU time. It looks something like this.

<pre>
<b>$</b> time ping google.com -c 1
<span class="comment">PING google.com (74.125.225.232) 56(84) bytes of data.
64 bytes from dfw06s26-in-f8.1e100.net (74.125.225.232): icmp_req=1 ttl=54 time=19.7 ms

--- google.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 19.767/19.767/19.767/0.000ms</span>

real   0m0.173s
user   0m0.010s
sys    0m0.100s
</pre>

The `real`, `user`, and `sys` information at the end is the output from `time`. StackOverflow has a [great explanation](http://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1/556411#556411) of what those values mean.

PowerShell's equivalent of `time` is `Measure-Command`. Unfortunately it only measures wall clock time. It also swallows the output of the command you are measuring.

<pre>
<b>PS&gt;</b> Measure-Command { ping google.com -n 1 }
Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 0
Milliseconds      : 282
Ticks             : 2822765
TotalDays         : 3.26708912037037E-06
TotalHours        : 7.84101388888889E-05
TotalMinutes      : 0.00470460833333333
TotalSeconds      : 0.2822765
TotalMilliseconds : 282.2765
</pre>

You might prefer calling `ToString()` on the result to make it more readable.

<pre>
<b>PS&gt;</b> (Measure-Command { ping google.com -n 1 }).ToString()
00:00:00.2822765
</pre>

One thing that `time` and `Measure-Command` have in common is that neither of them is well suited to benchmarking. That is, running the same command multiple times and providing statistics on the timings of those runs.

Enter `Benchmark-Command`. This PowerShell function does exactly that.

<pre>
<b>PS&gt;</b> Benchmark-Command { ping google.com -n 1 } -Samples 3
<span class="comment">Pinging google.com [74.125.225.226] with 32 bytes of data:
Reply from 74.125.225.226: bytes=32 time=20ms TTL=54

Ping statistics for 74.125.225.226:
    Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 20ms, Maximum = 20ms, Average = 20ms

Pinging google.com [74.125.225.226] with 32 bytes of data:
Reply from 74.125.225.226: bytes=32 time=17ms TTL=54

Ping statistics for 74.125.225.226:
    Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 17ms, Maximum = 17ms, Average = 17ms

Pinging google.com [74.125.225.226] with 32 bytes of data:
Reply from 74.125.225.226: bytes=32 time=18ms TTL=54

Ping statistics for 74.125.225.226:
    Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 18ms, Maximum = 18ms, Average = 18ms</span>

Avg: 56.2542ms
Min: 49.542ms
Max: 66.7821ms
</pre>

We can also swallow the command output if we want a more minimal display. This is very useful when taking a higher number of samples.

<pre>
<b>PS&gt;</b> Benchmark-Command { ping -n 1 google.com } -Samples 50 -Silent
..................................................
Avg: 46.9643ms
Min: 39.887ms
Max: 74.8587ms
</pre>

Import this in your profile and alias it to `time` and you get a nice counterpart to UNIX's `time` command with a few more bells and whistles.

<pre data-language="powershell">
# Microsoft.PowerShell_profile.ps1

Import-Module benchmark.psm1
Set-Alias time Benchmark-Command
</pre>

<pre data-language="powershell">
# benchmark.psm1
# Exports: Benchmark-Command

function Benchmark-Command ([ScriptBlock]$Expression, [int]$Samples = 1, [Switch]$Silent, [Switch]$Long) {
&lt;#
.SYNOPSIS
  Runs the given script block and returns the execution duration.
  Hat tip to StackOverflow. http://stackoverflow.com/questions/3513650/timing-a-commands-execution-in-powershell
  
.EXAMPLE
  Benchmark-Command { ping -n 1 google.com }
#&gt;
  $timings = @()
  do {
    $sw = New-Object Diagnostics.Stopwatch
    if ($Silent) {
      $sw.Start()
      $null = &amp; $Expression
      $sw.Stop()
      Write-Host "." -NoNewLine
    }
    else {
      $sw.Start()
      &amp; $Expression
      $sw.Stop()
    }
    $timings += $sw.Elapsed
    
    $Samples--
  }
  while ($Samples -gt 0)
  
  Write-Host
  
  $stats = $timings | Measure-Object -Average -Minimum -Maximum -Property Ticks
  
  # Print the full timespan if the $Long switch was given.
  if ($Long) {  
    Write-Host "Avg: $((New-Object System.TimeSpan $stats.Average).ToString())"
    Write-Host "Min: $((New-Object System.TimeSpan $stats.Minimum).ToString())"
    Write-Host "Max: $((New-Object System.TimeSpan $stats.Maximum).ToString())"
  }
  else {
    # Otherwise just print the milliseconds which is easier to read.
    Write-Host "Avg: $((New-Object System.TimeSpan $stats.Average).TotalMilliseconds)ms"
    Write-Host "Min: $((New-Object System.TimeSpan $stats.Minimum).TotalMilliseconds)ms"
    Write-Host "Max: $((New-Object System.TimeSpan $stats.Maximum).TotalMilliseconds)ms"
  }
}

Export-ModuleMember Benchmark-Command
</pre>