---
date: '2025-12-17T16:17:48+07:00'
draft: false
title: "RHEL OS cannot boot normally (/sysroot: can't read superblock)"
ShowToc: true
---

### Issue
The RHEL operating system cannot boot normally, experiencing a stuck state due to a mount error `/sysroot: can't read superblock on /dev/mapper/os-lv_root.`
![issue-2](issue-2.png)
![extend-disk1](extend-disk1.jpeg)
![extend-disk2](extend-disk2.jpeg)


### Troubleshooting Steps
1. Boot into rescue mode [How to boot RHEL to Rescue Mode](https://access.redhat.com/articles/3405661)
   
2. Extend LV /dev/mapper/os-lv_root menjadi 125G
   ```
   lvextend -L 125G /dev/mapper/os-lv_root
   ```

3. Resize the filesystem by using the xfs_growfs command
   
   Note ❗
   
   You need to confirm the filesystem type you're using
   ```
   xfs_growfs /dev/mapper/os-lv_root
   ```

### Root Cause
The system requires 80G on the root LV (`dev/mapper/os-lv_root`), but the root LV is only 71G in size.