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

In a PowerShell Module I was creating, I needed to cache the modules in GitHub Actions -- I couldn't use some prebuilt Actions (like [potatoqualitee/PSModuleCache]), as my dependencies are centrally defined in a `requirements.psd1`, then using [PSDepend](https://github.com/RamblingCookieMonster/PSDepend) to facilitate installation, obviously my next best choice was to figure it out myself.

## Prebuilt Actions?

If you can use one for your situation, I would actually recommend using [potatoqualitee/PSModuleCache] where possible, it uses `actions/cache` under the hood and does A LOT of the heavy-lifting for you (see [Things to Consider](#things-to-consider) below) including installation.

```yaml
- name: Install/Cache Modules
  uses: potatoqualitee/psmodulecache@v6.2
  with:
    modules-to-cache: PSFramework, PoshRSJob
```

However it explicitly requires you to re-write whatever modules you need to cache within the step itself, _if_ you are using other means to track/install your dependencies (such as PSDepend or [PSake](https://psake.dev)) it is not ideal.

## Things to Consider

There are a few big items to consider when caching PowerShell modules in GitHub Actions:

1. **Install Location** - Depending on the scope of installation, the install location will differ.
2. **Operating System** - The location of the modules will further differ depending on the OS.
3. **Preinstalled Modules** - Hosted Runners comes with a [set of preinstalled modules](https://github.com/actions/runner-images/blob/main/images/windows/Windows2022-Readme.md#powershell-tools) on the system level.

By default in GitHub Hosted Runners, modules are installed under the `C:\Program Files\WindowsPowerShell\Modules` directory, which is the system-level module path. However, caching this is not ideal as it contains a lot of the preinstalled modules we mentioned earlier, most are large and more often than not, not relevant to the project.

I ended up having to force my modules to either a custom location or the user-level module path, which is `C:\Users\runneradmin\Documents\WindowsPowerShell\Modules`. I ended up using the user-level module path, but you can use a custom location if you want to as long as you set the `PSModulePath` environment variable to include the custom location. Obviously, people working on the project will need to set this environment variable in their local environment as well.

- Checkout [Chrissy LeMaire's post for all `PSModulePath`'s on GitHub Runners](https://blog.netnerds.net/2022/03/github-runner-psmodulepath/), for all operating systems.

**For PSDepend users**, you can force the installation location in your dependencies file with `Target` set to `CurrentUser` (or a custom location):

```powershell
# requirements.psd1
'psake'     = @{
    Version = '4.9.1'
    Target  = 'CurrentUser'
}
#...
```

## Writing the Action

Once you have the install location set and an OS chosen, you can use `actions/cache` to cache the modules. Take note of where the modules are installed, as you will need to set the `path` parameter:

```yaml
- name: Cache PowerShell modules
  uses: actions/cache@v4
  with:
    path: C:\Users\runneradmin\Documents\PowerShell\Modules
    key: ${{ runner.os }}-powershell-${{ hashFiles('**/requirements.psd1') }}
    restore-keys: |
      ${{ runner.os }}-powershell-
```

I'm using a **requirements file as the [cache key](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#matching-a-cache-key)**, so if I add or update something, it will invalidate my cache and re-install the modules (as the Hash will change). If you're not sure what `restore-keys` are, check out the [action/cache inputs](https://github.com/actions/cache?tab=readme-ov-file#inputs) and read the actual [GitHub "Using the `cache` action" documentation](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#using-the-cache-action).

Note that the above example is for Windows, you'll have to **get creative if you're using a matrix** to run on multiple OS's -- As an example, [you could use the `runner.os` variable](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs#runner-context) to determine which you're on, then set the path programmatically.

## Example Flow

```yaml
name: CI Example
on: push
defaults:
  run:
    shell: pwsh # ps7
jobs:
  some-job:
    runs-on: windows-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Cache PowerShell modules
        uses: actions/cache@v4
        with:
          path: C:\Users\runneradmin\Documents\PowerShell\Modules
          key: ${{ runner.os }}-powershell-${{ hashFiles('**/requirements.psd1') }}
          restore-keys: |
            ${{ runner.os }}-powershell-

      - name: Do Something
        run: |
          Install-Module -Name PSDepend -Scope CurrentUser
          Get-Module PSDepend -ListAvailable
```

{{< alert >}}
**Never** `-Force` when using `Install-Module` in your workflow, as it will always reinstall modules regardless.
{{< /alert >}}

## What's Next?

Run your Action!

Each time you run your Action, it will check if the cache exists. If it does, it will restore the modules from the cache. If not, it will install the modules and create a new cache.

If you run a workflow with [debug logging](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/troubleshooting-workflows/enabling-debug-logging) enabled, it will tell you exactly what was restored and saved to/from cache!

Still confused or curious? [Checkout my demo repository](https://github.com/hudsonm62/PSModule-CacheDemo) for a working example of everything above.

{{< github repo="hudsonm62/PSModule-CacheDemo" >}}

<!--Links-->

[potatoqualitee/PSModuleCache]: https://github.com/potatoqualitee/psmodulecache

## Resources

- [Microsoft Learn | Installing a PowerShell Module](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/installing-a-powershell-module)
- [GitHub Docs | Caching Dependencies to Speed Up Workflows](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows)
- [actions/cache](https://github.com/actions/cache)
- [GitHub Docs | About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-github-hosted-runners/about-github-hosted-runners)

Article Photo by <a href="https://unsplash.com/@synkevych?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Roman Synkevych</a> on <a href="https://unsplash.com/photos/blue-and-black-penguin-plush-toy-UT8LMo-wlyk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
