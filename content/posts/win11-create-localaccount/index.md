---
title: Create Local Account on Windows 11 24H2
date: 2025-05-07
draft: true
description: The newest method to create a local account in Windows 11 24H2 OOBE, since BYPASSNRO was patched away by Microsoft in builds 26120.
summary: Create a local account in Windows 11 24H2 (26120)
tags:
  - Windows
  - Admin
---

In Windows 11, you can no longer seamlessly create a Local Account during OOBE setup as you could in previous editions. Microsoft are now forcing users to create an account using a Microsoft Login.

{{< alert >}}
As of Windows 11 24H2 (Build 26120), previous methods of bypassing this no longer function and have been patched away by Microsoft (such as using fake creds, and `OOBE\BYPASSNRO` command).
{{< /alert >}}

---

During device setup ([known as OOBE](https://en.wikipedia.org/wiki/Out-of-box_experience)), press `SHIFT` + `F10` to open a command prompt.

Then run:

```powershell
ms-chx:localonly
```

A Windows 10 style window will appear to continue with creating a local account.

<!--image here-->

{{< alert "pencil" >}}
Note this may or may not work in later builds.
{{< / alert >}}

## References

- [Toms Hardware | Install Windows 11 without Microsoft Account](https://www.tomshardware.com/how-to/install-windows-11-without-microsoft-account)

> Photo by <a href="https://unsplash.com/@windows?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Windows</a> on <a href="https://unsplash.com/photos/person-using-windows-11-computer-on-lap-AigsWJmvoEo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
