* [Overview](#overview)
* [Usage](#usage)
* [Commands](#commands)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Visor Command Line Interface provides scriptable monitoring capabilities for Ignite. It can be used to get statistics about nodes, caches and tasks in the grid. General details about the topology showing various metrics and node configuration properties can also be viewed here. Visor Command Line Interface also allows you to start and stop remote nodes.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/T32Eltb1SoaxDK1lEIvd_visor.png",
        "visor.png",
        "1090",
        "1106",
        "#5095c4",
        ""
      ],
      "sizing": "80"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Usage"
}
[/block]
Ignite ships with `IGNITE_HOME/bin/ignitevisorcmd.{sh|bat}` script that starts command line management interface.
To get help and get started, type `help` or `?` commands. To connect visor to the grid, type `open` command.
[block:api-header]
{
  "type": "basic",
  "title": "Commands"
}
[/block]
The following commands are available in Visor. To get full information on a command, type `help "cmd"` or `? "cmd"`.
[block:parameters]
{
  "data": {
    "h-0": "Command",
    "h-1": "Alias",
    "h-2": "Description",
    "0-2": "Acks arguments on all remote nodes.",
    "0-0": "`ack`",
    "1-2": "Alerts for user-defined events.",
    "1-0": "`alert`",
    "2-2": "Prints cache statistics, clears cache, prints list of all entries from cache.",
    "2-0": "`cache`",
    "3-0": "`close`",
    "3-2": "Disconnects Visor console from the grid.",
    "4-0": "`config`",
    "4-2": "Prints node configuration.",
    "5-2": "Copies file or folder to remote host.",
    "5-0": "`deploy`",
    "6-0": "`disco`",
    "6-2": "Prints topology change log.",
    "7-2": "Print events from a node.",
    "7-0": "`events`",
    "8-2": "Runs GC on remote nodes.",
    "8-0": "`gc`",
    "9-2": "Prints Visor console help.",
    "9-0": "`help`",
    "9-1": "?",
    "10-0": "`kill`",
    "10-2": "Kills or restarts node.",
    "11-2": "Starts or stops grid-wide events logging.",
    "11-0": "`log`",
    "12-2": "Clears Visor console memory variables.",
    "12-0": "`mclear`",
    "13-0": "`mget`",
    "13-2": "Gets Visor console memory variable.",
    "14-2": "Prints Visor console memory variables.",
    "14-0": "`mlist`",
    "15-2": "Prints node statistics.",
    "15-0": "`node`",
    "16-2": "Connects Visor console to the grid.",
    "16-0": "`open`",
    "17-0": "`ping`",
    "17-2": "Pings node.",
    "18-0": "`quit`",
    "18-2": "Quit from Visor console.",
    "19-0": "`start`",
    "19-2": "Starts or restarts nodes on remote hosts.",
    "20-2": "Prints Visor console status.",
    "20-0": "`status`",
    "20-1": "!",
    "21-0": "`tasks`",
    "21-2": "Prints tasks execution statistics.",
    "22-0": "`top`",
    "22-2": "Prints current topology.",
    "23-2": "Opens VisualVM for nodes in topology.",
    "23-0": "`vvm`"
  },
  "cols": 3,
  "rows": 24
}
[/block]