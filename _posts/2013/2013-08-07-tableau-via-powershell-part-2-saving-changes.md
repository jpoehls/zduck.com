---
title: "Tableau via PowerShell, Part 2: Saving Changes"
layout: post
categories: powershell tableau
iwpost: https://www.interworks.com/blogs/jpoehls/2013/08/07/tableau-powershell-part-2-saving-changes
description: "In part 1 I showed you how to open a Tableau workbook in PowerShell to start editing the workbook XML. Now I'll show you how to save those changes back to a TWB or TWBX file!"
redirect_to: http://joshua.poehls.me/2013/tableau-via-powershell-part-2-saving-changes
---

> This is part 2 of my mini-series on exploring Tableau workbooks with PowerShell. If you missed it, you should read [Part 1: Opening Workbooks]({{site.url}}/2013/exploring-tableau-workbooks-with-powershell-part-1-opening-workbooks/), before continuing.

Welcome back! Last time I showed you how to open a Tableau workbook (TWB or TWBX) in PowerShell so you could start exploring the raw XML. Today I'll show the next logical thing, saving the changes you make.

If you are only opening TWB files, then saving your changes couldn't be easier. Since TWBs are just XML files we can simply write it to a file, like so.

<pre>
$workbook = Get-TableauWorkbookXml 'MyWorkbook.twb'
$workbook.Save('ModifiedWorkbook.twb')
</pre>

Cake.

Saving changes back to a TWBX file is a bit more work but still totally doable. Let's write a new function, `Export-TableauPackagedWorkbook`. We will spice things up a bit by making this function smart. It'll take a few parameters to allow us to control exactly how we want the export to happen.

A `-Force` parameter will let us specify whether we want to overwrite the destination file if it already exists. By default we would be prompted. Note that this isn't a terribly useful parameter since it would replace the TWBX with an empty TWBX containing only the TWB.

`-Update` will let us specify that we want to update the TWB inside the destination TWBX file if the destination file exists. This is the cool parameter because it lets us open an existing TWBX, make changes to the TWB inside, and then save those changes back out into the original TWBX.

We will also support the common `-WhatIf` and `-Confirm` parameters that are so useful in PowerShell.

Here is our function:

> If you find this helpful then check out [TableauKit]({{site.url}}/2013/tableaukit-powershell-module-for-tableau/);
> a full on PowerShell module for working with Tableau files. It contains a new and improved
> version of the function below and much more.

<pre data-language="powershell">
function Export-TableauPackagedWorkbook {
&lt;#
.SYNOPSIS
    Exports the workbook XML to a packaged workbook (TWBX) file.
 
.PARAMETER Path
    The literal file path to export to.
 
.PARAMETER WorkbookXml
    The workbook XML to export.
 
.PARAMETER Update
    Whether to update the TWB inside the destination TWBX file
    if the destination file exists.
 
.PARAMETER Force
    Whether to overwrite the destination TWBX file if it exists.
    By default, you will be prompted whether to overwrite any
    existing file.
 
.NOTES
    Author: Joshua Poehls ({{site.url}})
#&gt;
    [CmdletBinding(
        SupportsShouldProcess=$true
    )]
    param(
        [Parameter(
            Position=0,
            Mandatory=$true
        )]
        [string]$Path,
 
        [Parameter(
            Position=1,
            Mandatory=$true,
            ValueFromPipeline=$true
        )]
        [xml]$WorkbookXml,
 
        [switch]$Update,
        [switch]$Force
    )
 
    begin {
        $originalCurrentDirectory = [System.Environment]::CurrentDirectory
 
        # System.IO.Compression.FileSystem requires at least .NET 4.5
        [System.Reflection.Assembly]::LoadWithPartialName("System.IO.Compression") | Out-Null
    }
 
    process {
        [System.Environment]::CurrentDirectory = (Get-Location).Path
        $entryName = [System.IO.Path]::GetFileNameWithoutExtension($Path) + '.twb'
        $createNewTwbx = $false
       
        if (Test-Path $Path) {
            if ($Update -or $Force -or $PSCmdlet.ShouldContinue('Overwrite existing file?', 'Confirm')) {
                if ($Update) {
                    if ($PSCmdlet.ShouldProcess($Path, 'Update TWB in packaged workbook')) {
 
                        [System.IO.FileStream]$fileStream = $null
                        [System.IO.Compression.ZipArchive]$zip = $null
                        try {
                            $fileStream = New-Object System.IO.FileStream -ArgumentList $Path, ([System.IO.FileMode]::Open), ([System.IO.FileAccess]::ReadWrite), ([System.IO.FileShare]::Read)
                            $zip = New-Object System.IO.Compression.ZipArchive -ArgumentList $fileStream, ([System.IO.Compression.ZipArchiveMode]::Update)
 
                            # Locate the existing TWB entry and remove it.
                            $entry = $zip.Entries |
                                where {
                                    # Look for a .twb file at the root level of the archive.
                                    $_.FullName -eq $_.Name -and ([System.IO.Path]::GetExtension($_.Name)) -eq '.twb'
                                } |
                                select -First 1
                            if ($entry) {
                                $entry.Delete()
                            }
 
                            $entry = $zip.CreateEntry($entryName, ([System.IO.Compression.CompressionLevel]::Optimal))
                            [System.IO.Stream]$entryStream = $null
                            try {
                                $entryStream = $entry.Open()
                                $WorkbookXml.Save($entryStream)
                            }
                            finally {
                                if ($entryStream) {
                                    $entryStream.Dispose()
                                }
                            }
                        }
                        finally {
                            if ($zip) {
                                $zip.Dispose()
                            }
                            if ($fileStream) {
                                $fileStream.Dispose()
                            }
                        }
                    }
                }
                else {
                    if ($PSCmdlet.ShouldProcess($Path, 'Replace existing packaged workbook')) {
                        # delete existing TWBX
                        Remove-Item $Path -ErrorAction Stop #TODO: Figure out how to pass WhatIf and Confirm to this
                        
                        $createNewTwbx = $true
                    }
                }
            }
        }
        else {
            if ($PSCmdlet.ShouldProcess($Path, 'Export packaged workbook')) {
                $createNewTwbx = $true
            }
        }
 
        if ($createNewTwbx) {
            [System.IO.FileStream]$fileStream = $null
            [System.IO.Compression.ZipArchive]$zip = $null
            try {
                $fileStream = New-Object System.IO.FileStream -ArgumentList $Path, ([System.IO.FileMode]::CreateNew), ([System.IO.FileAccess]::ReadWrite), ([System.IO.FileShare]::None)
                $zip = New-Object System.IO.Compression.ZipArchive -ArgumentList $fileStream, ([System.IO.Compression.ZipArchiveMode]::Update)
 
                $entry = $zip.CreateEntry($entryName, ([System.IO.Compression.CompressionLevel]::Optimal))
                [System.IO.Stream]$entryStream = $null
                try {
                    $entryStream = $entry.Open()
                    $WorkbookXml.Save($entryStream)
                }
                finally {
                    if ($entryStream) {
                        $entryStream.Dispose()
                    }
               }
            }
            finally {
                if ($zip) {
                    $zip.Dispose()
                }
                if ($fileStream) {
                    $fileStream.Dispose()
                }
            }
        }
    }
 
    end {
        [System.Environment]::CurrentDirectory = $originalCurrentDirectory
    }
}
</pre>

Not too bad. Here are some quick usage examples.

**Convert a TWB into a TWBX**

<pre>
$workbook = Get-TableauWorkbookXml 'MyWorkbook.twb'
$workbook | Export-TableauPackagedWorkbook 'MyPackagedWorkbook.twbx'
</pre>

**Update an existing TWBX**

<pre>
$workbook = Get-TableauWorkbookXml 'MyPackagedWorkbook.twbx'
# TODO: Make some changes to the workbook XML
$workbook | Export-TableauPackagedWorkbook 'MyPackagedWorkbook.twbx' -Update
</pre>

Leave me a comment if you have any questions about working with Tableau workbooks in PowerShell. Maybe I'll cover your topic in the next post!

{{site.powershell_book_ad}}