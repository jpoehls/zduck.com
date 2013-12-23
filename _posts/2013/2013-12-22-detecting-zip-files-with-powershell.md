---
title: Detecting ZIP files with PowerShell
layout: post
description: "Have you heard of magic numbers? Every ZIP file, for example, has same first 4 bytes. I'll show you how to find ZIP files by checking for these header bytes."
categories: powershell
---

Have you heard of [magic numbers][magicnumbers]? Some file formats are designed such that files are always saved with a specific byte sequence in the header. JPEG, PDF, and ZIP are all such formats.

You _could_ look for ZIP files by searching for all files with a `.zip` extension but a better way would be to look for all files that have `50 4b 03 04` as the first 4 bytes of the file. [All ZIP files will start with those bytes.][so] Not all ZIP files have the `.zip` extension.

Here's a `Test-ZipFile` PowerShell cmdlet that will return true or false whether the specified file has this magic header. You may also note that this cmdlet is a good citizen by [accepting file path input in an idiomatic way][pathinput].

> This cmdlet is also a great example of accepting `-Path` and `-LiteralPath` arguments in an idiomatic way. Including wildcard support for `-Path`.

[View on GitHub &#8594;](https://github.com/jpoehls/Poshato/blob/master/Test-ZipFile.ps1)

	function Test-ZipFile
	{
	<#
	.SYNOPSIS
	    Tests for the magic ZIP file header bytes.

	.DESCRIPTION
	    Inspired by http://stackoverflow.com/a/1887113/31308
	#>
		[CmdletBinding()]
		param(
			[Parameter(
				ParameterSetName  = "Path",
	            Mandatory = $true,
				ValueFromPipeline = $true,
				ValueFromPipelineByPropertyName = $true
			)]
			[string[]]$Path,

	        [Alias("PSPath")]
			[Parameter(
				ParameterSetName = "LiteralPath",
	            Mandatory = $true,
				ValueFromPipeline = $true,
				ValueFromPipelineByPropertyName = $true
			)]
			[string[]]$LiteralPath
		)

	    process {
	        $provider = $null

	        # Only expand wildcards if the -Path parameter was used.
	        if ($PSCmdlet.ParameterSetName -eq "Path") {
	            $filePaths = $PSCmdlet.GetResolvedProviderPathFromPSPath($Path, [ref]$provider)
	        }
	        elseif ($PSCmdlet.ParameterSetName -eq "LiteralPath") {
	            $filePaths = $PSCmdlet.GetResolvedProviderPathFromPSPath($LiteralPath, [ref]$provider)
	        }

	        foreach ($filePath in $filePaths) {
	            $isZip = $false
		        try {
		            $stream = New-Object System.IO.StreamReader -ArgumentList @($filePath)
		            $reader = New-Object System.IO.BinaryReader -ArgumentList @($stream.BaseStream)
		            $bytes = $reader.ReadBytes(4)
		            if ($bytes.Length -eq 4) {
		                if ($bytes[0] -eq 80 -and
		                    $bytes[1] -eq 75 -and
		                    $bytes[2] -eq 3 -and
		                    $bytes[3] -eq 4) {
		                    $isZip = $true
		                }
		            }
		        }
		        finally {
		            if ($reader) {
		                $reader.Dispose()
		            }
		            if ($stream) {
		                $stream.Dispose()
		            }
		        }

	            Write-Output $isZip
	        }
	    }
	}

_`Test-ZipFile` is part of [Poshato][poshato], my personal PowerShell module of miscellaneous goodness._

[magicnumbers]: http://en.wikipedia.org/wiki/Magic_number_(programming)#Magic_numbers_in_files
[so]: http://stackoverflow.com/a/1887113/31308
[pathinput]: http://stackoverflow.com/a/8506768
[poshato]: http://github.com/jpoehls/poshato