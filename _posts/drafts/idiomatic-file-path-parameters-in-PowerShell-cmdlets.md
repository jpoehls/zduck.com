---
title: Idiomatic file path parameters in PowerShell cmdlets
layout: post
categories: powershell
---

How often do you write cmdlets that operating on files? Probably all the time. You may be tempted to simply use a `[string]` type for your file path parameter but you may be missing a few things that really make this work well.

Let's cover a few downsides to using `[string]` as your parameter type.

- You can't pipe multiple file paths to your cmdlet. Using an array `[string[]]` will fix this.
- You can't (easily) pipe output from `Get-ChildItem` into your cmdlet.

There are other complexities you may not be thinking about as well.

- Do you resolve relative paths?
- Do you support `-LiteralPath`?
- Do you support wildcard paths?
- What if you want to accept paths that are file system paths? That is, paths from other PSProviders?

The goal of this post is to show you how to write a cmdlet that accepts `-Path` and `-LiteralPath` parameters in a way consistent with the behavior of the built-in cmdlets such as `Test-Path` and `Get-ChildItem`.

Let's get started. I'll show you how to do this in a PowerShell cmdlet first, then in a C# cmdlet. [Skip ahead to the C#][c#] if that's what you need.

> Credit where credit's due. I gathered most of this information from [an excellent post](http://stackoverflow.com/a/8506768) on StackOverflow.

-------------------

## Parameter Sets

If you want to accept either `-Path` or `-LiteralPath` parameters then you'll need to separate them into their own [parameter sets][parameter_sets]. The idea here is that you want to only accept one or the other, not both.

	[CmdletBinding()]
	param(
		[Parameter(
			ParameterSetName = "Path"
		)]
		[string[]]$Path,

		[Parameter(
			ParameterSetName = "LiteralPath"
		)]
		[string[]]$LiteralPath
	)

`[CmdletBinding()]` is necessary to turn this into an [advanced function][advanced_function]. This let's use use the `[Parameter]` attribute.

## Piping from `Get-ChildItem`

You may be surprised that piping `Get-ChildItem` to your cmdlet doesn't work. The reason is that `Get-ChildItem` returns `FileInfo` and `DirectoryInfo` objects, not strings. You work around this by accepting pipeline input by property name.

	[CmdletBinding()]
	param(
		[Parameter(
			ParameterSetName  = "Path",
			ValueFromPipeline = $true,
			ValueFromPipelineByPropertyName = $true
		)]
		[string[]]$Path,

		[Parameter(
			ParameterSetName = "LiteralPath",
			ValueFromPipeline = $true,
			ValueFromPipelineByPropertyName = $true
		)]
		[string[]]$LiteralPath
	)

Now our parameters accept input from the pipeline and piping input from `Get-ChildItem` works as we'd expect.

## Relative Paths

Typically you'll want to resolve any relative paths before processing them, here's how you might do that.

> Note: `Resolve-Path` will throw an exception for paths that don't exist.

	if ($PSCmdlet.ParameterSetName -eq "Path") {
		$paths = Resolve-Path -Path $Path |
				 	select -ExpandProperty Path
	}
	elseif ($PSCmdlet.ParameterSetName -eq "LiteralPath") {
		$paths = Resolve-Path -LiteralPath $LiteralPath |
				 	select -ExpandProperty Path
	}

The `$paths` variable now contains fully resolved paths.

## PowerShell Cmdlet

After we put it all together, here's what it looks like:

	[CmdletBinding()]
	param(
		[Parameter(
			ParameterSetName  = "Path",
			ValueFromPipeline = $true,
			ValueFromPipelineByPropertyName = $true
		)]
		[string[]]$Path,

		[Parameter(
			ParameterSetName = "LiteralPath",
			ValueFromPipeline = $true,
			ValueFromPipelineByPropertyName = $true
		)]
		[string[]]$LiteralPath
	)

	process {
		if ($PSCmdlet.ParameterSetName -eq "Path") {
			$paths = Resolve-Path -Path $Path |
					 	select -ExpandProperty Path
		}
		elseif ($PSCmdlet.ParameterSetName -eq "LiteralPath") {
			$paths = Resolve-Path -LiteralPath $LiteralPath |
					 	select -ExpandProperty Path
		}

		# TODO: Insert cmdlet logic here.
	}

## C# Cmdlet

