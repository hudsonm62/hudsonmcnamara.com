---
title: "Create a PowerShell Module"
date: 2025-06-27
description: "Learn the basics of how to create a PowerShell Module from scratch using a maintainable file structure, and later publish it to the PowerShell Gallery or other repository."
summary: "Learn the Basics of creating and publishing a PowerShell Module."
tags:
  - PowerShell
---

A [PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_modules) is basically a collection of functions and other resources that are packaged for reuse and distribution. If you're familiar with other languages, think of it like a Node.js package or a Python module.

{{< alert >}}
Basic PowerShell knowledge is assumed, if you're new to PowerShell, check out the [PS101 Course](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/00-introduction) and familiarise yourself with the basics before diving into module creation.
{{< /alert >}}

There are two types of modules in PowerShell:

- **Script Modules**: These are `.psm1` files that contain raw PowerShell code. They can be as simple as a single file or a collection of files in a directory.
- **Binary Modules**: These are compiled C# code in `.dll` files.

In this post, I'll be focusing on Script Modules, but you can also mix and match both types in a single module.

## Basic Concepts

The following are basic concepts you'll _need_ to understand when creating a PowerShell module.

### Module Locations

PowerShell modules can be stored anywhere on your filesystem. The directories where PowerShell looks for modules by default are defined in the `$env:PSModulePath` [environment variable](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables). You can view the current module paths by running:

```powershell
$env:PSModulePath -split ';'
```

When PowerShell Modules are installed via `Install-Module`, they are typically placed in one of those directories.

The locations included by default depend on your PowerShell version and OS, but you can also add custom paths to this variable by modifying it in your [PS Profile](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles) or adding paths to the variable mid session.

- Read more on [Default Module Locations](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_modules#default-module-locations)

### Import vs Install Module

When you want to use a module, you can either **import** it or **install** it. Installing a module means downloading it from a NuGet repository like the PowerShell Gallery, while importing a module means loading it into your current PowerShell session.

To install a module from the PowerShell Gallery, you can use the `Install-Module` cmdlet:

```powershell
Install-Module MyModule
# OR
Install-Module -Name MyModule -Scope CurrentUser
# OR
Install-Module -Name MyModule -Repository MyPersonalRepo
```

When you Install a module it does 2 things:

1. Downloads the module files to one of the directories in `$env:PSModulePath` (depending on the `-Scope` parameter).
2. **Imports** the module into your current session automatically.

To import a module, you can use the `Import-Module` cmdlet:

```powershell
Import-Module MyModule
# OR
Import-Module .\MyModule.psm1
# OR
Import-Module C:\Path\To\MyModuleFolder
```

As you can see from the above, you can import a module from either a [`.psm1` module script file](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/understanding-a-windows-powershell-module), a folder (containing a [`psd1` manifest file](#manifest-files)), or directly by name if it's already installed in a `$env:PSModulePath` directory.

During testing, you might want to import a module from a specific folder or the module script file.

{{< alert icon="fire" >}}
When importing from a folder, the [Manifest](#manifest-files) and Module filenames should match the directory name. For example, if your module is in a folder called `MyModule`, the manifest file should be named `MyModule.psd1` and the module file should be named `MyModule.psm1`.
{{< /alert >}}

### Manifest Files

Manifest files are stored in a [PowerShell Data file (`.psd1`)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_data_files) and used to define metadata about your module, including the module name, version, author, description, etc. They also allow you to specify dependencies and other environment requirements.

It should be noted that the manifest file is actually optional as modules only really require a `.psm1` file, but a manifest file is always required when publishing to PSGallery and enables you to organise your module in a more structured manner.

You can quickly generate a manifest file using the inbuilt `New-ModuleManifest` cmdlet:

```powershell
New-ModuleManifest -Path .\MyModule.psd1 -RootModule MyModule.psm1
```

You can also define the basic set of metadata in this cmdlet, but it's far easier to edit the file directly after creation if you're not using a script or other means of automation to generate it depending on your workflow.

You can always use the `Test-ModuleManifest` to validate the manifest file, and `Update-ModuleManifest` to update it with new metadata or exported functions/aliases.

{{< alert >}}
The generated manifest file is noticeably different in PS7 compared to 5.1 - If you import a module created in 5.1 inside of 7, it will automatically update the manifest file to the new format.
{{< /alert >}}

- Read more on [Module Manifest Files](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_module_manifests)

## Getting Started

### File Structure

When creating a PowerShell module, it's best to maintain some form of structure, as it's not always feasible to store everything in a single `.psm1` file, especially as a module grows in complexity.

In this post, I'll be using a common structure that separates public and private functions, as well as classes. This structure isn't a requirement, and you can alter it to suit your own needs.

In terms of PowerShell Modules, Public functions are those that you want to expose to users of your module (using `Export-ModuleMember`), while private functions are internal helper functions and not meant to be used outside the module itself.

You can also just create a single `.psm1` file without any subfolders if you prefer everything in one file but makes maintenance harder as/if your module grows.

The resulting structure would look like this:

```txt
MyModule
├── MyModule.psm1
├── MyModule.psd1
├── Public
│   └── __.ps1
├── Private
│   └── __.ps1
└── Classes
    └── __.ps1
```

{{< alert >}}
Please note that this is my preferred way of doing things, and is not a requirement nor the only way to structure a module.
Feel free to experiment with what suits you best.
{{< /alert >}}

If you're working inside a Git repository, always create a subfolder for your module, otherwise when sharing it will include unnecessary items and will bloat your module. You also can't guarantee that the folder name will match the module name, which is a requirement for importing modules by folder.

### Initial Setup

To create your initial module, start off with [a Manifest file](#manifest-files) and a `.psm1` file. You can use the following commands to get started:

```powershell
$ModuleName = "MyModule"

# Create a Module File
New-Item -Path ".\$ModuleName\$ModuleName.psm1" -ItemType File -Force

# Create a Manifest File
New-ModuleManifest -Path ".\$ModuleName\$ModuleName.psd1" -RootModule "$ModuleName.psm1" -Author "Your Name" -Description "A brief description of your module"

# Create folder structure
New-Item -Path .\$ModuleName\Public -ItemType Directory
New-Item -Path .\$ModuleName\Private -ItemType Directory
New-Item -Path .\$ModuleName\Classes -ItemType Directory
```

### Initial Module File

**Module files are executed as is, like a normal script** when Imported, so we can utilise this logic to tell PowerShell to find our public and private functions.

{{< alert >}}
If you're not using any folder structure, ensure all your "_public_" functions are exported using `Export-ModuleMember` at the end of your `.psm1` module file.
{{< /alert >}}

If you're using a private/public style folder structure, you could use my following code to import all public and private functions from their respective folders:

```powershell
# MyModule\MyModule.psm1

[string]$Root = (Get-Item -Path $PSScriptRoot -Force)
[string]$ModuleName = "MyModule"

# paths
$ManifestPath = Join-Path $Root "$ModuleName.psd1"
$PublicPath = Join-Path $Root "public"
$PrivatePath = Join-Path $Root "private"
$ClassesPath = Join-Path $Root "classes"

$Manifest = Test-ModuleManifest $ManifestPath -ErrorAction Stop

# get all files for import/export
$aliases = @()
$public  = Get-ChildItem -Path $PublicPath  -Recurse -Force | Where-Object { $_.Extension -eq ".ps1" }
$private = Get-ChildItem -Path $PrivatePath -Recurse -Force | Where-Object { $_.Extension -eq ".ps1" }
$classes = Get-ChildItem -Path $ClassesPath -Recurse -Force | Where-Object { $_.Extension -eq ".ps1" }

# Import all to session
$public | ForEach-Object { . $_.FullName }
$private | ForEach-Object { . $_.FullName }
$classes | ForEach-Object { . $_.FullName }

# Export 'public' functions (w/ aliases if present)
$public | ForEach-Object {
    $alias = Get-Alias -Definition $_.BaseName -ErrorAction SilentlyContinue
    if ($alias) {
        # Export defined aliases
        $aliases += $alias
        Export-ModuleMember -Function $_.BaseName -Alias $alias
    } else {
        # Export with no alias
        Export-ModuleMember -Function $_.BaseName
    }
}

# Update the module manifest on changes
$Added = $public | Where-Object {$_.BaseName -notin $Manifest.ExportedFunctions.Keys}
$Removed = $Manifest.ExportedFunctions.Keys | Where-Object {$_ -notin $public.BaseName}
$aliasesAdded = $aliases | Where-Object {$_ -notin $Manifest.ExportedAliases.Keys}
$aliasesRemoved = $Manifest.ExportedAliases.Keys | Where-Object {$_ -notin $aliases}
if ($Added -or $Removed -or $aliasesAdded -or $aliasesRemoved) {
    try {
        $updateModuleManifestParams = @{}
        $updateModuleManifestParams.Add("Path", $ManifestPath)
        $updateModuleManifestParams.Add("ErrorAction", "Stop")
        if ($aliasesAdded.Count -gt 0) { $updateModuleManifestParams.Add("AliasesToExport", $aliases) }
        if ($Added.Count -gt 0) { $updateModuleManifestParams.Add("FunctionsToExport", $public.BaseName) }

        Update-ModuleManifest @updateModuleManifestParams
    }
    catch {
        $_ | Write-Error
    }
}
```

All this code does is:

1. **Defines the root path** of the module and the paths for public, private, and class files.
2. **Imports everything** into the special module session.
3. **Exposes public functions** and aliases into the active session by file name.
4. **Updates the module manifest** with any changes to exported functions or aliases.

You may want to read more on [exporting functions](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/export-modulemember), [setting aliases](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_aliases) and [writing classes](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_classes).

## Writing Functions

If you're using the code from [Initial Module File](#initial-module-file), you can now create your functions in the `Public` folder and they will be automatically imported when the module is loaded. Just **ensure that the file name matches the function name**, as the script will `Export-ModuleMember` based on the file name.

```powershell
# MyModule\Public\Use-PublicFunction.ps1
function Use-PublicFunction {
    param ( [string]$Name )
    Write-Output "Hello, $Name!"
}
```

{{< alert >}}
Keep in mind you should **always use [Approved Verbs](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands)** as PowerShell will complain with a Warning if it finds any unapproved verbs in public function during Import.
{{< /alert >}}

You can now test your new function by importing the module (by directory):

```powershell
Import-Module .\MyModule
Use-PublicFunction -Name "World"
# Output: Hello, World!
```

PowerShell will automatically find the [module manifest](#manifest-files), then will execute the `.psm1` file. If you're using my PSM1 script, it will also Export all public functions from the `Public` folder and update the manifest with any new/removed functions and aliases.

### Using Private Functions

If you want to create a private function to use inside any other functions, do the same thing but place the function in the `Private` folder instead. The script will automatically import it, but it won't be available to users of the module:

```powershell
# MyModule\Private\Get-World.ps1
function Get-World {
    return "World"
}
```

Then use in your public functions:

```powershell
function Use-PublicFunction {
    param ( [string]$Name = (Get-World) )
    Write-Output "Hello, $Name!"
}
# Output: Hello, World!
```

{{< alert icon="fire" >}}
You can also define private functions in your `.psm1` file but it's a good practice to keep them separate for clarity and maintainability.
{{< /alert >}}

```powershell
Import-Module .\MyModule
Use-PublicFunction
# Output: Hello, World!

Get-World
# Output: Get-World : The term 'Get-World' is not recognized
```

## Testing Your Module

Apart from the literal basic testing of your module by importing it and running the functions, you should also use [Pester](https://pester.dev/) to write [unit tests](https://en.wikipedia.org/wiki/Unit_testing) for your module.

It's a bit out of scope to go into detail here, but you can learn more from the [Testing PowerShell with Pester Series](https://learn.microsoft.com/en-us/shows/testing-powershell-with-pester/) and from the [Pester Documentation](https://pester.dev/docs/)

Essentially, you can test public functions by importing the module and calling the functions directly in your tests. But if you want to test private functions and basic classes, you'll need to [use an `InModuleScope` block](https://pester.dev/docs/commands/InModuleScope) to access them, assuming the module is imported.

```powershell
# Tests\Use-PublicFunction.Tests.ps1

It "Should use the private Get-World function" {
    InModuleScope MyModule {
        Get-World | Should -Be "World"
    }
}
It "Should Run Public Function with default" {
    Use-PublicFunction | Should -Be "Hello, World!"
}
It "Should Run Public Function with parameter" {
    Use-PublicFunction -Name "Foo" | Should -Be "Hello, Foo!"
}
```

## Publishing Your Module

Once you're happy, you may want to publish your module to the [PowerShell Gallery](https://www.powershellgallery.com/) (or other [compatible NuGet repositories](https://learn.microsoft.com/en-us/powershell/gallery/powershellget/supported-repositories)).

To publish your module, you must ensure the following:

- Your module has a valid manifest file (`.psd1`).
- The manifest file has the required metadata filled out (like `ModuleVersion`, `Author`, `Description`, etc.).
- Everything is fully tested and working as expected.

When you're ready, [create an account on the PSGallery](https://learn.microsoft.com/en-us/powershell/gallery/how-to/publishing-packages/creating-an-accoun) and obtain your API Key, you can then use the `Publish-Module` cmdlet to publish your module:

```powershell
$params = @{
    Path        = '.\path\to\MyModule'
    NuGetApiKey = '' # fill in with api key
    LicenseUri  = 'http://path.to/license.txt'
    Tag         = 'Foo', 'Bar', 'Baz'
    ReleaseNote = 'Initial Release.'
    Repository  = 'PSGallery'
}
Publish-Module @params
```

- Read more on [PSGallery Publishing Guidelines](https://learn.microsoft.com/en-us/powershell/gallery/concepts/publishing-guidelines) and [Creating and Publishing Items](https://learn.microsoft.com/en-us/powershell/gallery/how-to/publishing-packages/publishing-a-package)

---

And that's it! You've learned the basics of creating a PowerShell module: structuring, writing functions, and publishing it.

Whilst it's _far_ from the only way to go about things, this should give you a solid foundation to build upon, and _mostly everything_ can be adapted or automated to suit your needs and workflow.

If you want to take it a step further, consider looking into:

- Using [PSDepend](https://psdepend.dev/) to manage module dependencies
- Validating script files using [PSScriptAnalyzer](https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/overview)
- Writing comprehensive tests for your module using Pester and using [Code Coverage](https://pester.dev/docs/usage/code-coverage)
- Generating [Helpfiles](https://learn.microsoft.com/en-us/powershell/utility-modules/platyps/create-help-using-platyps) and Documentation with [PlatyPS](https://learn.microsoft.com/en-us/powershell/utility-modules/platyps/overview)
- Automating Testing and Publishing using GitHub Actions or Azure DevOps

---

See the example module I created for this post:

{{< github repo="hudsonm62/PSModule-Example" showThumbnail=false >}}
