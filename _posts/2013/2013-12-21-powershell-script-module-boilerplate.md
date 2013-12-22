---
title: PowerShell Script Module Boilerplate
layout: post
categories: powershell
---

One of the things I always look for when getting familiar with a new language or environment is examples of the physical file structure and logical organization of the code. How do you layout your project files? What is idiomatic?

Unsurprisingly, this isn't always as simple as finding a few open-source projects on GitHub to reference. Believe it or not, there are a lot of pretty unorganized coders out there. I admit to being a bit OCD with my project structures. I like them clean, organized, and consistent.

In this post I'm going to cover my preferred boilerplate for [PowerShell Script Modules][script_module].

Script Modules are about as simple as it gets. Typically you have one or more PS1 or PSM1 files that contain your module's cmdlets. Beyond that you _should_ have a [PSD1 manifest][manifest].

## Fork this!

This entire boilerplate [is on GitHub][fork]. If you just want a solid starting point, download this repo. If you want to know more, keep reading.

## File Structure

	+- src/
	| +- source_file.ps1
	| +- ...
	+- tools/
	| +- release.ps1
	+- LICENSE
	+- README.md

- `src/` contains the PS1, PSM1, PS1XML, and any other source files for the module.
- `tools/` is where I put any meta scripts for the project. Usually there is just one script here that builds a release version of my module.
- `LICENSE` - if your module is open-source, always specify what license you are releasing it under.
- `README.md` - always have a README file. Even if it is only a one sentence description. [Markdown][markdown] is a great format to use for this.

## Explicit Exports

By default, PowerShell will export all of the functions in your module. I recommend being explicit about this and always specifying which functions should be publically exported. This way it is easy to add private helper functions that are internal to your module without worrying about them being made public accidentally.

You do this by calling [`Export-ModuleMember`][export_module_member] at the bottom of your source file. If you have a `source.psm1` file that contains a `Show-Calendar` function that should be public, you would do something like this:

	# Show-Calendar.ps1
	#
	# Show-Calendar will be public.
	# Any other functions in this file will be private.

	function Show-Calendar {
		# ...
	}

	Export-ModuleMember -Function Show-Calendar

[Full example &#8594;][export_example]

## Release Script

**Every project should have a release script.** You should never be manually building your distributable release. At a minimum my release script will:

1. Generate the PSD1 manifest for my module.
2. Save the manifest into a temporary `./dist` folder.
3. Copy all of the module source files into `./dist`.
4. Add all of the module's source files to a ZIP file ready for me to distribute.

Here is what a simple `release.ps1` script might look like.

[View on GitHub &#8594;][release_script]

	<#
	.SYNOPSIS
		Generates a manifest for the module
		and bundles all of the module source files
		and manifest into a distributable ZIP file.
	#>

	[CmdletBinding()]
	param(
	    [Parameter(Mandatory = $true)]
	    [version]$ModuleVersion
	)

	$ErrorActionPreference = "Stop"

	Write-Host "Building release for v$moduleVersion"

	$scriptPath = Split-Path -LiteralPath $(if ($PSVersionTable.PSVersion.Major -ge 3) { $PSCommandPath } else { & { $MyInvocation.ScriptName } })

	$src = (Join-Path (Split-Path $scriptPath) 'src')
	$dist = (Join-Path (Split-Path $scriptPath) 'dist')
	if (Test-Path $dist) {
	    Remove-Item $dist -Force -Recurse
	}
	New-Item $dist -ItemType Directory | Out-Null

	Write-Host "Creating module manifest..."

	$manifestFileName = Join-Path $dist 'YourModule.psd1'

	# TODO: Tweak the manifest to fit your module's needs.
	New-ModuleManifest `
	    -Path $manifestFileName `
	    -ModuleVersion $ModuleVersion `
	    -Guid fe524c79-95a6-4d02-8e15-30dddeb8c874 `
	    -Author 'Your Name' `
	    -CompanyName 'Your Company' `
	    -Copyright '(c) $((Get-Date).Year) Your Company. All rights reserved.' `
	    -Description 'Description of your module.' `
	    -PowerShellVersion '3.0' `
	    -DotNetFrameworkVersion '4.5' `
	    -NestedModules (Get-ChildItem $src -Exclude *.psd1 | % { $_.Name })

	Write-Host "Creating release archive..."

	# Copy the distributable files to the dist folder.
	Copy-Item -Path "$src\*" `
	          -Destination $dist `
	          -Recurse

	# Requires .NET 4.5
	[Reflection.Assembly]::LoadWithPartialName("System.IO.Compression.FileSystem") | Out-Null

	$zipFileName = Join-Path ([System.IO.Path]::GetDirectoryName($dist)) "$([System.IO.Path]::GetFileNameWithoutExtension($manifestFileName))-$ModuleVersion.zip"

	# Overwrite the ZIP if it already already exists.
	if (Test-Path $zipFileName) {
	    Remove-Item $zipFileName -Force
	}

	$compressionLevel = [System.IO.Compression.CompressionLevel]::Optimal
	$includeBaseDirectory = $false
	[System.IO.Compression.ZipFile]::CreateFromDirectory($dist, $zipFileName, $compressionLevel, $includeBaseDirectory)

	Move-Item $zipFileName $dist -Force

## Version Control

Always exclude your `./dist` folder from source control. As a rule of thumb, you never want to store the build output of any project in source control.

Depending on how you plan to release your module, you may prefer to exclude `*.psd1` manifest files from source control. This just keeps things clean and enforces that you use your release script to build the distributable.
	
## Good Examples

Here are a few open-source PowerShell modules that I've found to be good examples to follow.

- [PoshInternals](https://github.com/adamdriscoll/PoshInternals) by Adam Driscoll
- [Psake](https://github.com/psake/psake) by James Kovacs
- [Poshato](https://github.com/jpoehls/poshato)

<!--
## Binary Modules

I use a very similar file structure for [PowerShell Binary Modules][binary_module] but they do introduce more complexity into the release script. I'll be covering my binary module boilerplate in a future post. 
-->

[binary_module]: http://msdn.microsoft.com/en-us/library/dd878342(v=vs.85).aspx
[script_module]: http://msdn.microsoft.com/en-us/library/dd878340(v=vs.85).aspx
[manifest]: http://msdn.microsoft.com/en-us/library/dd878337(v=vs.85).aspx
[export_example]: http://msdn.microsoft.com/en-us/library/dd878340(v=vs.85).aspx
[export_module_member]: http://technet.microsoft.com/en-us/library/hh849736.aspx
[markdown]: http://daringfireball.net/projects/markdown/
[fork]: http://github.com/jpoehls/powershell-script-module-boilerplate
[release_script]: https://github.com/jpoehls/powershell-script-module-boilerplate/blob/master/tools/release.ps1