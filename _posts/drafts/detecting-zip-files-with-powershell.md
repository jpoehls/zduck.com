---
title: Detecting ZIP files with PowerShell
layout: post
published: false
categories: powershell
---

# Detecting ZIP files with PowerShell (or C#)

Have you heard of [magic numbers][magicnumbers]? Some file formats are designed such that files are always saved with a specific byte sequence in the header. JPEG, PDF, and ZIP are all such formats.

You _could_ look for ZIP files by searching for all files with a `.zip` extension but a better way would be to look for all files that have `50 4b 03 04` as the first 4 bytes of the file. [All ZIP files will start with those bytes.][so] Not all ZIP files have the `.zip` extension.

Here's a `Test-ZipFile` PowerShell cmdlet that will return true or false whether the specified file has this magic header. You may also note that this cmdlet is a good citizen by [accepting file path input in an idiomatic way][pathinput].

	# Tests for the magic zip file header.
	# Inspired by http://stackoverflow.com/a/1887113/31308
	function Test-ZipFile([string]$path) {
		<#
		.SYNOPSIS
			Tests whether the specified file is a ZIP file
			by reading the initial 4 bytes of the file.
		#>
	    try {
	       $stream = New-Object System.IO.StreamReader -ArgumentList @($path)
	       $reader = New-Object System.IO.BinaryReader -ArgumentList @($stream.BaseStream)
	       $bytes = $reader.ReadBytes(4)
	       if ($bytes.Length -eq 4) {
	           if ($bytes[0] -eq 80 -and
	                $bytes[1] -eq 75 -and
	                $bytes[2] -eq 3 -and
	                $bytes[3] -eq 4) {

	                return $true
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

	    return $false
	}

_`Test-ZipFile` is also part of [Poshato][poshato], my personal PowerShell module of miscellaneous goodness._

[magicnumbers]: http://en.wikipedia.org/wiki/Magic_number_(programming)#Magic_numbers_in_files
[so]: http://stackoverflow.com/a/1887113/31308
[pathinput]: ...
[poshato]: http://github.com/jpoehls/poshato