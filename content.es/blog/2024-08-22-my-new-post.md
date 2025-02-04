---
categories:
- Linux
- AlmaLinux
date: "2024-08-22T00:00:00Z"
tags:
- quotas
- linux
- directadmin
title: Enable Quotas AlmaLinux 9 Directadmin
---


Please try running this command:

   ```
  grubby --args="rootflags=uquota,pquota" --update-kernel=ALL
   ```

and then reboot your server.
After that check if the XFS quota is OK.