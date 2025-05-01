---
title: "Extract Lenovo .INF Drivers"
date: 2025-05-01
draft: true
description: "How to extract the .INF Driver files from Lenovo driver packages, as regular extraction from the .exe does not work."
summary: "Extracting the .INF driver files from Lenovo."
tags:
  - Windows
  - Admin
  - Drivers
---

Lenovo only provides `.exe` installers for drivers - This is handy for end-users and in support situations, but annoying for IT Admins doing Zero or Low Touch style setup of some variety.

Typically, these kind of driver installers are just a fancy "self-extractor" wrapper, in those situations you can simply extract the driver out using [7-Zip](https://www.7-zip.org/) -- However this isn't the case for Lenovo drivers.

## Instructions

Download the Driver you need from Lenovo; Then open an elevated Command Prompt or PowerShell window, navigate to where the file is and execute it with: `/VERYSILENT /DIR=".\path\to\extract" /EXTRACT=YES`

Full command example:

```powershell
.\pkg.exe /VERYSILENT /DIR="C:\driver" /EXTRACT=YES
```

{{< alert "graduation-cap" >}}
`DIR=` is the target path to extract to.
{{< /alert >}}

## Alternatives

Lenovo has an app called [ThinInstaller](https://www.lenovo.com/us/en/software/thin-installer/), but the user experience is awful and one too many steps just to quickly pull a handful of drivers.

## References

- [Lenovo Forums | Extract the .inf without running the .exe?](https://forums.lenovo.com/t5/Enterprise-Client-Management/Extract-the-inf-without-running-the-exe/m-p/5047321?page=1#5174522)
- [Lenovo Administrator Tools](https://support.lenovo.com/us/en/solutions/ht037099-download-thinkvantage-technologies-administrator-tools)
