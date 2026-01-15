---
date: '2025-12-5T16:17:48+07:00'
draft: false
title: "Inventory Pending Delete Ansible Automation Platform"
ShowToc: true
---

## Load Django Shell
```
sudo awx-manage shell_plus
```

## Display inventory
```
from awx.main.models import Inventory
```
```
Inventory.objects.all()
```
## Delete inventory based on id
```
Inventory.objects.filter(id=168).delete()
```

## Display inventory "pending delete"
```
Inventory.objects.filter(pending_deletion=True)
```
