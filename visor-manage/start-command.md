* [Start Command Specification](#start-command-specification)
* [Examples](#examples)
[block:api-header]
{
  "type": "basic",
  "title": "Start command specification"
}
[/block]
Starts or restarts nodes on remote host.
```start -f=<path> {-m=<num>} {-r}```
```start -h=<hostname> {-p=<num>} {-u=<username>} {-pw=<password>} {-k=<path>}```
    ```{-n=<num>} {-g=<path>} {-c=<path>} {-s=<path>} {-m=<num>} {-r}```
Alert options:
* ```-f=<path>``` Path to INI file that contains topology specification. 
Topology specification file can contain the next properties:
    * ```[host name]```
    Name of section with properties for specified host.
    * ```host```
    IP address or host name
    * ```port```
    SSH port
    * ```uname```
    SSH login
    * ```passwd=password```
    SSH password
    * ```key```
    SSH key path
    * ```nodes```
    Start node count
    * ```igniteHome```
    Ignite home path
    * ```cfg```
    Ignite config path
    * ```script```
    Ignite node start script
* ```-h=<hostname>``` Hostname where to start nodes.
Can define several hosts if their IPs are sequential.
Example of range is 192.168.1.100~150, which means all IPs from 192.168.1.100 to 192.168.1.150 inclusively.
* ```-p=<num>``` Port number (default is 22).
* ```-u=<username>``` Username (if not defined, current local username will be used).
* ```-pw=<password>``` Password (if not defined, private key file must be defined).
* ```-k=<path>``` Path to private key file. Define if key authentication is used.
* ```-n=<num>``` Expected number of nodes on the host.
If some nodes are started already, then only remaining nodes will be started.
If current count of nodes is equal to this number and '-r' flag is not set, then nothing will happen.
* ```-g=<path>``` Path to Ignite installation folder.
If not defined, IGNITE_HOME environment variable must be set on remote hosts.
* ```-c=<path>``` Path to configuration file (relative to Ignite home).
If not provided, default Ignite configuration is used.
* ```-s=<path>``` Path to start script (relative to Ignite home).
Default is "bin/ignite.sh" for Unix or
"bin\ignite.bat" for Windows.
* ```-m=<num>``` Defines maximum number of nodes that can be started in parallel on one host.
This actually means number of parallel SSH connections to each SSH server.
Default is 5.
* ```-t=<num>``` Defines connection timeout in milliseconds (default is 2000).
* ```-r``` Indicates that existing nodes on the host will be restarted.
By default, if flag is not present, existing nodes will be left as is.
[block:api-header]
{
  "type": "basic",
  "title": "Examples"
}
[/block]
```start "-h=10.1.1.10 -u=uname -pw=passwd -n=3"```
Starts three nodes with default configuration (password authentication).
```start "-h=192.168.1.100~104 -u=uname -k=/home/uname/.ssh/is_rsa -n=5"```
Starts 25 nodes on 5 hosts (5 nodes per host) with default configuration (key-based authentication).
```start "-f=start-nodes.ini -r"```
Starts topology defined in 'start-nodes.ini' file. Existing nodes are stopped.

[block:code]
{
  "codes": [
    {
      "code": "# section with settings for host 1\n[host1]\n# ip address or host name\nhost=192.168.1.1\n# ssh port\nport=22\n# ssh login\nuname=userName\n# ssh password\npasswd=password\n# ssh key path\nkey=~/.ssh/id_rsa\n# start node count\nnodes=1\n# ignite home path\nigniteHome=/usr/lib/ignite\n# ignite config path\ncfg=/examples/exmaple-ignite.xml\n# ignite node start script\nscript=/bin/ignite.sh\n",
      "language": "text",
      "name": "start-nodes.ini"
    }
  ]
}
[/block]