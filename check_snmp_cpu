#!/bin/bash

# check_snmp_cpu
# Description : Check the CPU LOAD using SNMP
# Version : 1.7
# Author : Yoann LAMY
# Licence : GPLv2

# Commands
CMD_BASENAME="/bin/basename"
CMD_SNMPWALK="/usr/bin/snmpwalk"
CMD_AWK="/bin/awk"
CMD_EXPR="/usr/bin/expr"
CMD_GREP="/bin/grep"

# Script name
SCRIPTNAME=`$CMD_BASENAME $0`

# Version
VERSION="1.7"

# Plugin return codes
STATE_OK=0
STATE_WARNING=1
STATE_CRITICAL=2
STATE_UNKNOWN=3

# 'hrProcessorFrwID', HOST-RESOURCES-MIB
OID_PROCESSID=".1.3.6.1.2.1.25.3.3.1.1"

# 'hrProcessorLoad', HOST-RESOURCES-MIB
OID_LOAD=".1.3.6.1.2.1.25.3.3.1.2"

# Default variables
DESCRIPTION="Unknown"
STATE=$STATE_UNKNOWN
CPU_LOAD=0
NUMBER_PROC=0

# Default options
COMMUNITY="public"
HOSTNAME="127.0.0.1"
WARNING=0
CRITICAL=0

# Option processing
print_usage() {
  echo "Usage: ./check_snmp_cpu -H 127.0.0.1 -C public -w 80 -c 90"
  echo "  $SCRIPTNAME -H ADDRESS"
  echo "  $SCRIPTNAME -C STRING"
  echo "  $SCRIPTNAME -w INTEGER"
  echo "  $SCRIPTNAME -c INTEGER"
  echo "  $SCRIPTNAME -h"
  echo "  $SCRIPTNAME -V"
}

print_version() {
  echo $SCRIPTNAME version $VERSION
  echo ""
  echo "This nagios plugins comes with ABSOLUTELY NO WARRANTY."
  echo "You may redistribute copies of the plugins under the terms of the GNU General Public License v2."
}

print_help() {
  print_version
  echo ""
  print_usage
  echo ""
  echo "Check the CPU load"
  echo ""
  echo "-H ADDRESS"
  echo "   Name or IP address of host (default: 127.0.0.1)"
  echo "-C STRING"
  echo "   Community name for the host SNMP agent (default: public)"
  echo "-w INTEGER"
  echo "   Warning level for the cpu load in percent (default: 0)"
  echo "-c INTEGER"
  echo "   Critical level for the cpu load in percent (default: 0)"
  echo "-h"
  echo "   Print this help screen"
  echo "-V"
  echo "   Print version and license information"
  echo ""
  echo ""
  echo "This plugin uses the 'snmpwalk' command included with the NET-SNMP package."
  echo "This plugin support performance data output."
  echo "If the warning level and critical levels are both set to 0, then the script returns OK state."
}

while getopts H:C:w:c:hV OPT
do
  case $OPT in
    H) HOSTNAME="$OPTARG" ;;
    C) COMMUNITY="$OPTARG" ;;
    w) WARNING=$OPTARG ;;
    c) CRITICAL=$OPTARG ;;
    h)
      print_help
      exit $STATE_UNKNOWN
      ;;
    V)
      print_version
      exit $STATE_UNKNOWN
      ;;
   esac
done

# Plugin processing
for ID in `$CMD_SNMPWALK -t 2 -r 2 -v 1 -c $COMMUNITY $HOSTNAME $OID_PROCESSID | $CMD_AWK '{ print $1 }' | $CMD_AWK -F "." '{print $NF}'`
do
  VALUE=`$CMD_SNMPWALK -t 2 -r 2 -v 1 -c $COMMUNITY -Ovq $HOSTNAME "${OID_LOAD}.${ID}"`
  if [ -z "$VALUE" ]; then
    VALUE=0
  fi
  CPU_LOAD=`$CMD_EXPR $CPU_LOAD + $VALUE`
  NUMBER_PROC=`$CMD_EXPR $NUMBER_PROC + 1`
done

if [ $NUMBER_PROC != 0 ]; then
  CPU_LOAD=`$CMD_EXPR $CPU_LOAD / $NUMBER_PROC`

  if [ $WARNING != 0 ] || [ $CRITICAL != 0 ]; then
    if [ $CPU_LOAD -gt $CRITICAL ] && [ $CRITICAL != 0 ]; then
      STATE=$STATE_CRITICAL
    elif [ $CPU_LOAD -gt $WARNING ] && [ $WARNING != 0 ]; then
      STATE=$STATE_WARNING
    else
      STATE=$STATE_OK
    fi
  else
    STATE=$STATE_OK
  fi
fi

DESCRIPTION="CPU load : $CPU_LOAD% | cpu_used=$CPU_LOAD;$WARNING;$CRITICAL;0"

echo $DESCRIPTION
exit $STATE
