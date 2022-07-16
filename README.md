# check_qnap_nas
nagios check for QNAP NAS 

# NOTES
This script runs on a nagios server, and queries a QNAP NAS device via SNMP for the following metrics
- SMART errors
- Physical disk errors
- Logical disk errors
- Power supply health
- Disk temperature

# ASSUMPTIONS
SNMP is enabled on QNAP NAS

# USAGE

The SNMP queries will sometimes timeout if there is a disk failure on the NAS, which can cause the nagios check to timeout.  To avoid this, create crontab entries for the nagios user that runs this check every 15 minutes, and creates a temporary file containing the check output.  Subsequent checks will read from the temporary file, which is refreshed every 15 minutes.  Example crontab entries:
```
#run via cron every 15 minutes in case the script times out in nagios
1,16,31,46 * * * * /usr/local/nagios/libexec/check_qnap_nas -H mynas01 -C public >/dev/null 2>&1
2,17,32,47 * * * * /usr/local/nagios/libexec/check_qnap_nas -H mynas02 -C public  >/dev/null 2>&1
```

Create a section similar to the following in the commands.cfg file on the nagios server
```
# 'check_qnap_nas' command definition
# parameters are -H hostname -C snmp_community
define command{
        command_name    check_qnap_nas
        command_line    $USER1$/check_qnap_nas -H $HOSTADDRESS$ -C $ARG1$
        }
```

Create a section similar to the following in the services.cfg on the nagios server
```
define service {
   use                             generic-service
   host_name                       mynas01,mynas02 
   service_description             QNAP NAS Health
   check_command                   check_qnap_nas!public
   check_interval                  60           ; Actively check the host every 60 minutes
   }
```

# SAMPLE OUTPUT
<img src=images/qnap.png>
