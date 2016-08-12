Visor Command Line can be start in batch mode (run several commands).

Run ignitevisorcmd.{sh|bat} -help it will show available options:
[block:code]
{
  "codes": [
    {
      "code": "Usage:\n    ignitevisorcmd [? | -help]|[{-v}{-np} {-cfg=<path>}]|[{-b=<path>} {-e=command1;command2;...}]\n    Where:\n        ?, /help, -help      - show this message.\n        -v                   - verbose mode (quiet by default).\n        -np                  - no pause on exit (pause by default).\n        -cfg=<path>          - connect with specified configuration.\n        -b=<path>            - batch mode with file.\n        -e=cmd1;cmd2;...     - batch mode with commands.\n",
      "language": "shell",
      "name": "ignitevisorcmd.{sh|bat} -?"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Batch Mode Using File with Commands"
}
[/block]
In this mode commands will be read from file. All commands must start from a new line.
[block:code]
{
  "codes": [
    {
      "code": "open\n0\nstatus",
      "language": "text",
      "name": "commands.txt"
    }
  ]
}
[/block]
Then need to specify path to file with commands using **-b** option.
[block:code]
{
  "codes": [
    {
      "code": "ignitevisorcmd.{bat|sh} -np -b=commands.txt",
      "language": "shell",
      "name": "Usage"
    }
  ]
}
[/block]
This example will connect to grid using configuration with index "0" and execute "status" command.
[block:api-header]
{
  "type": "basic",
  "title": "Batch Mode Using List of Commands"
}
[/block]
In this mode commands will be read from -e option. Commands must be separated by semicolons.
[block:code]
{
  "codes": [
    {
      "code": "ignitevisorcmd.{bat|sh} -np -e=\"open;0;status\"",
      "language": "text",
      "name": "Usage"
    }
  ]
}
[/block]
When commands have space symbols they should be additionally quoted.
[block:code]
{
  "codes": [
    {
      "code": "ignitevisorcmd.{bat|sh} -np -e=\"'open -cpath=config/default-config.xml;status'\"",
      "language": "text",
      "name": "Command quoting"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Please note, that in batch mode Visor Command Line simply run supplied commands one by one as if they were entered from keyboard."
}
[/block]