Run ignitevisorcmd.{sh|bat} -? it will show available options:
[block:code]
{
  "codes": [
    {
      "code": "Usage:\n    ignitevisorcmd.bat [? | -help]|[{-v}{-np} {-cfg=<path>}]|[{-b=<path>} {-e=command1;command2;...}]\n    Where:\n        ?, /help, -help      - show this message.\n        -v                   - verbose mode (quiet by default).\n        -np                  - no pause on exit (pause by default).\n        -cfg=<path>          - connect with specified configuration.\n        -b=<path>            - batch mode with file.\n        -e=cmd1;cmd2;...     - batch mode with commands.\n",
      "language": "text",
      "name": "ignitevisorcmd.{sh|bat} -?"
    }
  ]
}
[/block]
First batch mode "-b=<path>" will read commands from file. All commands must start from a new line.
[block:code]
{
  "codes": [
    {
      "code": "open\n0\nstatus",
      "language": "text",
      "name": "sample.txt"
    }
  ]
}
[/block]
Usage: ignitevisorcmd.{sh|bat} -np -b=sample.txt