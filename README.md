# check_snmp_cpu
nagios check for CPU utilization via SNMP

# Usage

Copy the check_snmp_cpu script to /usr/local/nagios/libexec/ on the nagios server (or wherever your environment keeps its nagios checks)


Add the following section to commands.cfg on the nagios server
```
# 'check_snmp_cpu' command definition
define command{
        command_name    check_snmp_cpu
        command_line    $USER1$/check_snmp_cpu -H $HOSTADDRESS$ -C $ARG1$ -w $ARG2$  -c $ARG3$
        }
```

Add the following section to services.cfg on the nagios server
```
# Define a service to check CPU util on Linux / FreeBSD / MacOS boxes / Windows
# syntax is check_snmp_cpu!community!warn!crit
define service{
        use                             generic-service
        hostgroup_name                  all_windows,all_linux
        service_description             CPU util
        check_command                   check_snmp_cpu!public!70!90
        }
```
