---
date: '2024-12-13T22:35:01+07:00'
draft: false
title: 'RHEL OS failed to boot, got stuck (crowdstrike service) and the OS shut down immediately'
---
### Issue
The RHEL OS running on top of openstack failed to boot, got stuck (crowdstrike service) and the OS shut down immediately. The logs for this event could not be collected even with rescue mode.

### Troubleshooting Steps
1. Boot into rescue mode using rd.break, then check the disk space utilization.
```bash
df -TH 
```
Observe disk usage with a percentage usage of up to 100%. 
> **In this case I found that _/var/log/audit_ was full.**

2. Free up space for the _/var/log/audit/_ partition and reboot the node
```bash
sudo rm -f /var/log/audit/audit.log.*
```

3. Change Audit Rules Configuration to prevent the system halt when the _/var/log/audit/_ full
```bash
vi /etc/audit/auditd.conf
```
```bash
disk_full_action = ignore
```

### Root Cause
This issue caused by audit rule policy trigger the operating system going halt when _/var/log/audit/_ full. _/var/log/audit_ full caused by there is massive increasing log on _/var/log/audit_ which make the 2 partition was full. After free up the space for _/var/log/audit_ problem solved. And to prevent the same issue appear in the future we change the audit rule policy.
