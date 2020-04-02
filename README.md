# pfSense Zabbix template

This is a pfSense active template for Zabbix, based on [Keenton Zabbix Template](https://github.com/keentonsas/zabbix-template-pfsense) for freeBSD part and a php script using pfSense functions library for monitoring specific data.

Tested with pfSense 2.4 and Zabbix 4.0

## What it does

 - pfSense Version/Update Available
 - Gateway Monitoring (Gateway Status/RTT with discovery)
 - OpenVPN Server Monitoring (Server Status/Tunnel Status with discovery)
 - CARP Monitoring (Global CARP State)
 - Basic service monitoring (Service Status with discovery)
 

## Configuration

First copy the file pfsense_zbx.php to your pfsense box (e.g. to /root/scripts).

For example, from pfSense shell:

```bash
mkdir /root/scripts
curl -o /root/scripts/pfsense_zbx.php https://raw.githubusercontent.com/rbicelli/pfsense-zabbix-template/master/pfsense_zbx.php
```

Then install package "Zabbix Agent 4" on your pfSense Box


In Advanced Features-> User Parameters

```bash
AllowRoot=1
UserParameter=pfsense.states.max,grep "limit states" /tmp/rules.limits | cut -f4 -d ' '
UserParameter=pfsense.states.current,grep "current entries" /tmp/pfctl_si_out | tr -s ' ' | cut -f4 -d ' '
UserParameter=pfsense.mbuf.current,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f1
UserParameter=pfsense.mbuf.cache,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f2
UserParameter=pfsense.mbuf.max,netstat -m | grep "mbuf clusters" | cut -f1 -d ' ' | cut -d '/' -f4
UserParameter=pfsense.discovery[*],/usr/local/bin/php /root/scripts/pfsense_zbx.php discovery $1
UserParameter=pfsense.value[*],/usr/local/bin/php /root/scripts/pfsense_zbx.php $1 $2 $3
```

_Please note that **AllowRoot=1** option is required in order to execute correctly OpenVPN checks and others._

Then import xml template in Zabbix and add your pfSense hosts.

If you are running a redundant CARP setup you can adjust the macro {#EXPECTED_CARP_STATUS} to a value representing what is CARP expected status on monitored box.

Possible values are:

 - 0: Disabled
 - 1: Master
 - 2: Backup

This is useful when monitoring services which could stay stopped on CARP Backup Member.
