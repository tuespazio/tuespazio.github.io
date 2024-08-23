---
layout: post
title: Enable Quotas AlmaLinux 9 Directadmin
date: 2024-08-22 22:37 -0600
categories: [Linux, AlmaLinux]
tags: [quotas,linux,directadmin]     # TAG names should always be lowercase
---


Please try running this command:

   ```
  grubby --args="rootflags=uquota,pquota" --update-kernel=ALL
   ```

and then reboot your server.
After that check if the XFS quota is OK.