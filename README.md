# Template Module Linux XFS by Zabbix agent 

## Overview

For Zabbix version: 5.0 and higher.

Requires: `jq`, `grep`, `awk` and `sudo` configuration.

I needed to use `grep` because of the slashes in XFS project paths. If you know a way to resolve the issue with only `awk` let me know.

Create file `/etc/zabbix/zabbix_agentd.d/template_linux_xfs.conf` with contents:
```
UserParameter=xfs.quota.projects, awk -F':' '{ printf "{\"{#XFSNAME}\":\"%s\"}\n",$2 }' /etc/projects | jq --slurp 'map(select(. != ""))'
UserParameter=xfs.quota.project.used[*], sudo xfs_quota -c 'df -N' | grep ' $1$$' | awk '{print $$3*1024}'
UserParameter=xfs.quota.project.total[*], sudo xfs_quota -c 'df -N' | grep ' $1$$' | awk '{print $$2*1024}'
UserParameter=xfs.quota.project.pused[*], sudo xfs_quota -c 'df -N' | grep ' $1$$' | awk '{print +$$5}'
UserParameter=xfs.quota.project.pfree[*], sudo xfs_quota -c 'df -Ni' |grep ' $1$$' | awk '{print 100-$5}'
```

Create sudoers file `/etc/sudoers.d/zabbix_xfs` with contents:
```
Cmnd_Alias XFS_QUOTA = /usr/sbin/xfs_quota -c df -N, /usr/sbin/xfs_quota -c df -Ni
zabbix ALL = (ALL) NOPASSWD: XFS_QUOTA
```

You can test if sudo is working by trying it:
```
# su - zabbix -s /bin/bash -c "sudo /usr/sbin/xfs_quota -c 'df -N'"
```
and
```
# su - zabbix -s /bin/bash -c "sudo /usr/sbin/xfs_quota -c 'df -N'"
```

## Zabbix configuration

No specific Zabbix configuration is required.

### Macros used

Please read the descriptions in the template.
