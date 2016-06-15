Visor Command Line can be start in batch mode (run several commands).

Run ignitevisorcmd.{sh|bat} -? it will show available options:
[block:code]
{
  "codes": [
    {
      "code": "Usage:\n    ignitevisorcmd.bat [? | -help]|[{-v}{-np} {-cfg=<path>}]|[{-b=<path>} {-e=command1;command2;...}]\n    Where:\n        ?, /help, -help      - show this message.\n        -v                   - verbose mode (quiet by default).\n        -np                  - no pause on exit (pause by default).\n        -cfg=<path>          - connect with specified configuration.\n        -b=<path>            - batch mode with file.\n        -e=cmd1;cmd2;...     - batch mode with commands.\n",
      "language": "text",
      "name": null
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Batch mode using file with commands"
}
[/block]
First batch mode "-b=<path>" will read commands from file. All commands must start from a new line.
[block:code]
{
  "codes": [
    {
      "code": "open\n0\nstatus",
      "language": "text",
      "name": null
    }
  ]
}
[/block]
Usage: ignitevisorcmd.{sh|bat} -np -b=sample.txt

This will open configuration with index "0" and execute "status" command.
[block:api-header]
{
  "type": "basic",
  "title": "Batch mode using list of commands"
}
[/block]
Second batch mode will take a list of commands separated by semicolons.

Usage: ignitevisorcmd.{sh|bat} -np -e=open;0;status

This will do the same as previous example.
[block:callout]
{
  "type": "info",
  "body": "Please note, that in batch mode Visor Command Line simply run supplied commands one by one as if they were entered from keyboard."
}
[/block]