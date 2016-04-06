##System File Descriptor Limit

When running a large number of threads accessing the grid as in the case of large-scale server-side applications, you may end up with a large number of open files used both on client and server nodes. It is recommended that you increase the default values to the max defaults.

Misconfiguring the file descriptors settings will impact application stability and performance. For this we have to set both “System level File Descriptor Limit” and “Process level File Descriptor Limit”, respectively, by following these steps as a root user:

1. Modify the following line in the **/etc/sysctl.conf** file:
[block:code]
{
  "codes": [
    {
      "code": "fs.file-max = 300000",
      "language": "text"
    }
  ]
}
[/block]
2. Apply the change by executing the following command:
[block:code]
{
  "codes": [
    {
      "code": "/sbin/sysctl -p",
      "language": "shell"
    }
  ]
}
[/block]
Verify your settings using:
[block:code]
{
  "codes": [
    {
      "code": "cat /proc/sys/fs/file-max ",
      "language": "shell"
    }
  ]
}
[/block]
Alternatively, you may execute the following command:
[block:code]
{
  "codes": [
    {
      "code": "sysctl fs.file-max\n",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": ""
}
[/block]
##Process File Descriptor Limit

By default, Linux OS has a relatively small number of file descriptors available and max user processes (1024) configured. It is important that you use a user account which has its maximum open file descriptors (open files) and max user processes configured to an appropriate value. 
[block:callout]
{
  "type": "info",
  "body": "A good maximum value for open file descriptors is 32768"
}
[/block]
Use the following command to set the maximum open file descriptors and maximum user processes:
[block:code]
{
  "codes": [
    {
      "code": "ulimit -n 32768 -u 32768",
      "language": "shell"
    }
  ]
}
[/block]
Alternatively, you may modify the following files accordingly:
[block:code]
{
  "codes": [
    {
      "code": "/etc/security/limits.conf\n\n- soft    nofile          32768\n- hard    nofile          32768\n\n/etc/security/limits.d/90-nproc.conf\n\n- soft nproc 32768\n",
      "language": "text"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "See [increase-open-files-limit](https://easyengine.io/tutorials/linux/increase-open-files-limit/) for more details."
}
[/block]