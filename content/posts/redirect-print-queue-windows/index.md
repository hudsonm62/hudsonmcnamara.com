---
title: "Redirect Print Queue to a Different Queue"
date: 2025-02-17
draft: false
description: "Redirect a Windows Printer queue to a different port to fix offline, incorrect or stuck print jobs without losing queued documents or reinstalling the printer."
summary: "Redirect a Windows Printer queue to a different port."
tags:
  - Windows
  - Printers
---

Whenever you print, your operating system adds the job to a "[Print Queue](https://www.ibm.com/docs/en/ftmswsfm300?topic=service-print-queues)" which is tied to a specific "[Port](https://www.papercut.com/blog/print_basics/what-is-a-printer-port-and-why-should-you-care/)". If a queue is stuck, offline, or incorrect, you can redirect jobs to a different queue by switching ports.

To transfer a queue by port on Windows:

1. Go to "**Printer Properties**" for the old queue.
2. Click "**Ports**" tab & find the associated Port.
3. Deselect it, and choose a port with a working queue (or vice versa)

Once changed, your print jobs should automatically start using the new queue. If this is a temporary fix, remember to switch it back later.

This can quickly resolve printing issues without reinstalling the printer or losing queued print jobs.

> Article Photo by <a href="https://unsplash.com/@enginakyurt?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">engin akyurt</a> on <a href="https://unsplash.com/photos/a-white-and-black-printer-sitting-on-top-of-a-counter-CGnoRQZGWmw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>
