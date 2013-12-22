---
title: Idiomatic file path parameters in PowerShell cmdlets
layout: post
categories: powershell
---

# Writing a PowerShell cmdlet that accepts file path input

## Script cmdlet

function Get-TableauXml {
31 <#
32 .SYNOPSIS
33     Gets the workbook XML from a TWB or TWBX file.
34 
35 .NOTES
36     Author: Joshua Poehls <joshua@zduck.com>
37 #>
38     [CmdletBinding()]
39     param(
40         [Parameter(
41             Mandatory = $true,
42             ParameterSetName = "Path",
43             Position = 0,
44             ValueFromPipeline = $true,
45             ValueFromPipelineByPropertyName = $true)]
46         [ValidateNotNullOrEmpty()]
47         [string[]]$Path,
48 
49         [Parameter(
50             Mandatory = $true,
51             ParameterSetName = "LiteralPath",
52             ValueFromPipeline = $true,
53             ValueFromPipelineByPropertyName = $true)]
54         [ValidateNotNullOrEmpty()]
55         [Alias("FullName")]
56         [string[]]$LiteralPath
57     )
58 
59     begin {
60         # System.IO.Compression.FileSystem requires at least .NET 4.5
61         Add-Type -AssemblyName "System.IO.Compression"
62     }
63 
64     process {
65         if ($PSCmdlet.ParameterSetName -eq "Path") {
66             $paths = Resolve-Path -Path $Path | select -ExpandProperty Path
67         }
68         elseif ($PSCmdlet.ParameterSetName -eq "LiteralPath") {
69             $paths = Resolve-Path -LiteralPath $LiteralPath | select -ExpandProperty Path
70         }
71 
72         foreach ($p in $paths) {
73             #$extension = [System.IO.Path]::GetExtension($p)
74             if (-not(Test-ZipFile $p)) {
75                 Write-Output ([xml](Get-Content -LiteralPath $p))
76             }
77             else {
78                 $archiveStream = $null
79                 $archive = $null
80                 $reader = $null
81 
82                 try {
83                     $archiveStream = New-Object System.IO.FileStream($p, [System.IO.FileMode]::Open)
84                     $archive = New-Object System.IO.Compression.ZipArchive($archiveStream)
85                     $twbEntry = ($archive.Entries | Where-Object { $_.FullName -eq $_.Name -and [System.IO.Path]::GetExtension($_.Name) -eq ".twb" })[0]
86                     $reader = New-Object System.IO.StreamReader $twbEntry.Open()
87 
88                     [xml]$xml = $reader.ReadToEnd()
89                     Write-Output $xml
90                 }
91                 finally {
92                     if ($reader -ne $null) {
93                         $reader.Dispose()
94                     }
95                     if ($archive -ne $null) {
96                         $archive.Dispose()
97                     }
98                     if ($archiveStream -ne $null) {
99                         $archiveStream.Dispose()
100                     }
101                 }
102             }
103         }
104     }
105 }

## C# cmdlet