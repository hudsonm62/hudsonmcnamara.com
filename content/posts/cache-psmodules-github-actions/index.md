---
title: "Cache PowerShell Modules in GitHub Actions"
date: 2025-05-02
draft: false
description: "How to cache PowerShell modules in GitHub Actions to speed up CI/CD pipelines."
summary: "Cache PowerShell modules in GitHub Actions."
tags:
  - Automation
  - PowerShell
  - GitHub Actions
---

In a PowerShell Module I was creating, I needed to cache a handful of modules in my GitHub Actions workflows -- I couldn't use some [prebuilt actions][potatoqualitee/PSModuleCache], as my dependencies are centrally defined in a `requirements.psd1`, then using [PSDepend](https://github.com/RamblingCookieMonster/PSDepend) to facilitate installation, obviously my next best choice was to figure it out myself.

## Prebuilt Actions?

If you can use one for your situation, I would actually recommend using [potatoqualitee/PSModuleCache] where possible, it uses `actions/cache` under the hood and does A LOT of the heavy-lifting for you (see [Things to Consider](#things-to-consider) below) including installation.

```yaml
- name: Install/Cache Modules
  uses: potatoqualitee/psmodulecache@v6.2
  with:
    modules-to-cache: PSFramework, PoshRSJob
```

However it explicitly requires you to re-write whatever modules you need to cache **within the step itself**. If you are using other means to track/install your dependencies (such as PSDepend or [PSake](https://psake.dev)) it is not ideal.

## Things to Consider

There are a few big items to consider when caching PowerShell modules:

1. **Install Location** - Depending on the scope of installation, the install location will differ.
2. **Operating System** - The location of the modules will differ depending on the OS.
3. **Preinstalled Modules** - Hosted Runners comes with a [set of preinstalled modules](https://github.com/actions/runner-images/blob/main/images/windows/Windows2022-Readme.md#powershell-tools) on the system level.

By default in GitHub Hosted Runners, modules are installed under the system-level module path. However, **caching this is not ideal** as it contains a lot of the preinstalled modules we mentioned earlier, most are large and more often than not, not relevant to the project.

In my case, I ended up forcing my modules to either a custom location or the user-level module path. You can use a custom location if you want to, as long as the `PSModulePath` environment variable includes the custom location. Obviously, people working on the project will need to set this environment variable in their local environment as well.

- Checkout [Chrissy LeMaire's post for all `PSModulePath`'s on GitHub Runners](https://blog.netnerds.net/2022/03/github-runner-psmodulepath/), for all operating systems.

### PSDepend Install

You can force the installation location in your dependencies file with `Target` set to `CurrentUser` (or a custom path):

```powershell
@{
    'psake'     = @{
        Version = '4.9.1'
        Target  = 'CurrentUser'
    }
    #...
}
```

PSDepend can also add path entries to the `PSModulePath` environment variable for you:

```powershell
@{
    PSDependOptions = @{
        Target      = '.\path\to\custom\location'
        AddToPath   = $true
    }
    'psake'         = '4.9.1'
    'PoshRSJob'     = '3.0.0'
    #...
}

```

## Writing the Action

Once you have the install location set, you can use `actions/cache` to cache the modules. Take note of where the modules are installed, as you will need to set the `path` parameter:

```yaml
# Windows Only
- name: Cache PowerShell modules
  uses: actions/cache@v4
  with:
    path: C:\Users\runneradmin\Documents\PowerShell\Modules
    key: ${{ runner.os }}-pwsh-${{ hashFiles('**/requirements.psd1') }}
    restore-keys: |
      ${{ runner.os }}-pwsh-
```

I'm using a **requirements file as the [cache key](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#matching-a-cache-key)**, so if I add or update something, it will invalidate my cache and re-install the modules (as the Hash will change).

### Matrix Strategy

For a [Matrix](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/running-variations-of-jobs-in-a-workflow), you'll have to **get creative** to run on multiple OS's -- [You _could_ use the `runner.os` variable](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs#runner-context) to determine which you're on, then set the path programmatically:

```yaml
- name: Set cache path
  run: |
    if ('${{ runner.os }}' -eq 'Windows') {
      echo "CACHE_PATH=$(Join-Path $HOME 'Documents\PowerShell\Modules')" >> $env:GITHUB_ENV
    } else {
      echo "CACHE_PATH=$HOME/.local/share/powershell/Modules" >> $env:GITHUB_ENV
    }

- name: Cache PowerShell modules
  uses: actions/cache@v4
  id: cache
  with:
    path: ${{ env.CACHE_PATH }}
    key: ${{ runner.os }}-pwsh-${{ hashFiles('**/requirements.psd1') }}
    restore-keys: |
      ${{ runner.os }}-pwsh-
```

### Install and Import

When a cache key is restored, you don't need to install the modules again, but you will need to import them. Conversely, if the cache is not restored, you'll have to install the modules first, _then_ import them.

{{< alert >}}
Don't use `-Force` when using `Install-Module` in your workflow, as it will always reinstall modules regardless.
{{< /alert >}}

You can use an `if` conditional to install or import depending on if cache was restored, since `actions/cache` will set an output variable `cache-hit`:

```yaml
- name: Install Dependencies if not found
  if: steps.cache.outputs.cache-hit != 'true'
  run: |
    Set-PSRepository PSGallery -InstallationPolicy Trusted
    Install-Module -Name PSDepend -Scope CurrentUser -Confirm:$false
    Invoke-PSDepend -Install -Force -Verbose

- name: Import Modules
  run: |
    Import-Module -Name PSDepend -Verbose
    Invoke-PSDepend -Import -Force -Verbose
```

## Example Flow

The example below shows a complete matrix workflow that caches PowerShell modules using `actions/cache` and installs them if they are not found in the cache.

```yaml
name: Example Matrix Workflow
on: push
defaults:
  run:
    shell: pwsh # ps7
jobs:
  your-matrix-job:
    runs-on: windows-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set cache path
        run: |
          if ('${{ runner.os }}' -eq 'Windows') {
            echo "CACHE_PATH=$(Join-Path $HOME 'Documents\PowerShell\Modules')" >> $env:GITHUB_ENV
          } else {
            echo "CACHE_PATH=$HOME/.local/share/powershell/Modules" >> $env:GITHUB_ENV
          }

      - name: Cache PowerShell modules
        uses: actions/cache@v4
        id: cache
        with:
          path: ${{ env.CACHE_PATH }}
          key: ${{ runner.os }}-pwsh-${{ hashFiles('**/requirements.psd1') }}
          restore-keys: |
            ${{ runner.os }}-pwsh-

      - name: Install Dependencies if not found
        if: steps.cache.outputs.cache-hit != 'true'
        run: |
          Set-PSRepository PSGallery -InstallationPolicy Trusted
          Install-Module -Name PSDepend -Scope CurrentUser -Confirm:$false
          Invoke-PSDepend -Install -Force -Verbose

      - name: Import Modules
        run: |
          Import-Module -Name PSDepend -Verbose
          Invoke-PSDepend -Import -Force -Verbose

      - name: Run Something
        run: Invoke-PSake -NoLogo
```

## What's Next?

Run your Action!

{{< alert "graduation-cap" >}}

If you run a workflow with [debug logging](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/enabling-debug-logging) enabled, it will tell you exactly what was restored and saved to/from cache!

{{< /alert >}}

Still confused or curious? [Checkout the demo repository](https://github.com/hudsonm62/PSModule-CacheDemo) for a working example of everything above.

{{< github repo="hudsonm62/PSModule-CacheDemo" >}}

<!--Links-->

[potatoqualitee/PSModuleCache]: https://github.com/potatoqualitee/psmodulecache

## References

- [Microsoft Learn | Installing a PowerShell Module](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/installing-a-powershell-module)
- [GitHub Docs | Caching Dependencies to Speed Up Workflows](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows)
- [GitHub Docs | About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners)
- [GitHub | actions/cache](https://github.com/actions/cache)
