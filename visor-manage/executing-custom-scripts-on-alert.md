* [Alert Command Specification](#alert-command-specification)
* [Examples](#examples)
* [Custom Script](#custom-script)
[block:api-header]
{
  "type": "basic",
  "title": "Alert Command Specification"
}
[/block]
Register alert: ```alert -r {-t=<sec>} {-<metric>=<condition><value>} ... {-<metric>=<condition><value>}```
Unregister: ```alert -u {-id=<alert-id>|-a}```

Alert options:
* ```-n``` Alert name
* ```-u ``` Unregisters alert(s). Either '-a' flag or '-id' parameter is required. Note that only one of the '-u' or '-r' is allowed. If neither '-u' or '-r' provided - all alerts will be printed.   
* ```-a ``` When provided with '-u' - all alerts will be unregistered.
* ```-id=<alert-id>``` When provided with '-u' - alert with matching ID will be unregistered.  
* ```-r``` Register new alert with mnemonic predicate(s).
**Note** only one of the '-u' or '-r' is allowed. If neither '-u' or '-r' provided - all alerts will be printed.
* ```-t``` Defines notification frequency in seconds. Default is 60 seconds. 
**Note** This parameter can only appear with ```-r```
* ```-s``` Define script for execution when alert triggered.
For configuration of throttle period see -i argument. Script will receive following arguments: alert name or alert ID when name is not defined, condition as string, values of alert conditions ordered as in alert command.
* ```-i``` Configure alert notification minimal throttling interval in seconds. Default is 60 seconds.
* ```-<metric>``` This defines a mnemonic for the metric that will be measured.
Grid-wide metrics (not node specific):
    * cc - Total number of available CPUs in the grid.
    * nc - Total number of nodes in the grid.
    * hc - Total number of physical hosts in the grid.
    * cl - Current average CPU load (in %) in the grid.

Per-node current metrics:
    * aj - Active jobs on the node.
    * cj - Cancelled jobs on the node.
    * tc - Thread count on the node.
    * ut - Up time on the node. By default (no suffix provided) value is assumed to be in milliseconds.
  **Note** <num> can have 's', 'm', or 'h' suffix indicating seconds, minutes, and hours.
    * je - Job execute time on the node.
    * jw - Job wait time on the node.
    * wj - Waiting jobs count on the node.
    * rj - Rejected jobs count on the node.
    * hu - Heap memory used (in MB) on the node.
    * cd - Current CPU load on the node.
    * hm - Heap memory maximum (in MB) on the node.
* ```<condition>```
      Comparison part of the mnemonic predicate:
         eq - Equal '=' to '<value>' number.
         neq - Not equal '!=' to '<value>' number.
         gt - Greater than '>' to '<value>' number.
         gte - Greater than or equal '>=' to '<value>' number.
         lt - Less than '<' to 'NN' number.
         lte - Less than or equal '<=' to '<value>' number.
[block:api-header]
{
  "type": "basic",
  "title": "Examples"
}
[/block]
```alert ```
   Prints all currently registered alerts.
    
```alert -u -a```
   Unregisters all currently registered alerts.
    
```alert -u -id=12345678```
   Unregisters alert with provided ID.
    
```alert -r -t=900 -cc=gte4 -cl=gt50```
   Register alert that will notify every 15 min if grid has >= 4 CPUs and > 50% CPU load.

```alert -r -n=Nodes -t=15 -nc=gte3 -s=/home/user/scripts/alert.sh -i=300```
   Register alert that will notify every 15 second if grid has >= 3 nodes and execute script "/home/user/scripts/alert.sh" with repeat interval not less than 5 min.
[block:api-header]
{
  "type": "basic",
  "title": "Custom Script"
}
[/block]
Register alert that will execute script: ```/home/user/myScript.sh``` every 15 second if grid has >= 2 nodes and cpu count <= 16 with repeat interval not less than 5 min.
[block:code]
{
  "codes": [
    {
      "code": "alert -r -t=5 -n=MyAlert -nc=gte2 -cc=lte16 -i=15 -s=/home/user/myScript.sh",
      "language": "shell",
      "name": "Usage"
    }
  ]
}
[/block]
Alert handle script:
[block:code]
{
  "codes": [
    {
      "code": "echo ALERT [$1] CONDITION [$2] alarmed with node count [$3] and cpu count [$4]",
      "language": "shell",
      "name": "myScript.sh"
    }
  ]
}
[/block]
Will generate output in terminal like this:
[block:code]
{
  "codes": [
    {
      "code": "ALERT [MyAlert] CONDITION [-nc=gte2 -cc=lte16] alarmed with node count [2] and cpu count [8]",
      "language": "shell",
      "name": "Output"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Please note, that $1 points to alert name, $2 points to alert conditions and $3, $4,.... point to value of each sub-condition."
}
[/block]