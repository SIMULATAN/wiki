---
title: Arch Linux
description: 
published: true
date: 2025-05-24T14:55:32.925Z
tags: 
editor: markdown
dateCreated: 2025-05-24T14:55:32.925Z
---

# Configuration Tips
## Remove `sudo` Ctrl-C delay
Source: https://bbs.archlinux.org/viewtopic.php?id=277348

```
#%PAM-1.0

auth       required                    pam_faillock.so      preauth nodelay # ADD HERE
# Optionally use requisite above if you do not want to prompt for the password
# on locked accounts.
-auth      [success=2 default=ignore]  pam_systemd_home.so
auth       [success=1 default=bad]     pam_unix.so          try_first_pass nullok
auth       [default=die]               pam_faillock.so      authfail nodelay # ADD HERE
auth       optional                    pam_permit.so
auth       required                    pam_env.so
auth       required                    pam_faillock.so      authsucc nodelay # ADD HERE
# If you drop the above call to pam_faillock.so the lock will be done also
# on non-consecutive authentication failures.

-account   [success=1 default=ignore]  pam_systemd_home.so
account    required                    pam_unix.so
account    optional                    pam_permit.so
account    required                    pam_time.so

-password  [success=1 default=ignore]  pam_systemd_home.so
password   required                    pam_unix.so          try_first_pass nullok shadow nodelay # ADD HERE (doesn't seem to work though)
password   optional                    pam_permit.so

-session   optional                    pam_systemd_home.so
session    required                    pam_limits.so
session    required                    pam_unix.so
session    optional                    pam_permit.so
```