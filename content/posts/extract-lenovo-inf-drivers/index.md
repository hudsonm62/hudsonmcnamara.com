---
title: "Extract any Lenovo .INF Drivers"
date: 2025-05-01
draft: false
description: "How to extract the .INF Driver files from Lenovo driver packages, as regular extraction from the .exe does not work."
summary: "Extracting Lenovo driver files from their .exe packages."
tags:
  - Windows
  - Admin
  - Drivers
---

Lenovo distributes drivers exclusively as `.exe` installer packages. Whilst this is convenient for end users and support situations, it's a headache for working on Zero or Low Touch deployments, where `.INF` driver files are required.

Typically, these kind of driver installers are just a fancy "self-extractor" wrapper, so you can simply extract the driver out using [7-Zip](https://www.7-zip.org/) -- However this isn't the case for Lenovo drivers.

## Instructions

Fortunately, Lenovo's `.exe` installers support command-line options to extract files properly.

Download the Driver you need from Lenovo; Then open an elevated Command Prompt or PowerShell terminal window, navigate to where the file is and execute it with: `/VERYSILENT /DIR=".\path\to\extract" /EXTRACT=YES`

### Example

```powershell
.\driver.exe /VERYSILENT /DIR="C:\drivers" /EXTRACT=YES
```

{{< alert "graduation-cap" >}}
`DIR=` specifies the target directory for extracted files.
{{< /alert >}}

## Alternatives

- Lenovo has a tool called [ThinInstaller](https://www.lenovo.com/us/en/software/thin-installer/), but the user experience is awful and one too many steps just to quickly pull a handful of drivers.
- Extracting from an updated lab machine of the same model is also an option, if you have access to one.

## References

- [Lenovo Forums | Extract the .inf without running the .exe?](https://forums.lenovo.com/t5/Enterprise-Client-Management/Extract-the-inf-without-running-the-exe/m-p/5047321?page=1#5174522)
- [Lenovo Administrator Tools](https://support.lenovo.com/us/en/solutions/ht037099-download-thinkvantage-technologies-administrator-tools)

> Article Photo by <a href="https://unsplash.com/@nakul_in?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Nakul</a> on <a href="https://unsplash.com/photos/black-and-gray-laptop-computer-turned-on-in-dark-room-QxPRz2oTOWo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
