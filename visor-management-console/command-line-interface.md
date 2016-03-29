Visor Command Line Interface (CLI) provides scriptable monitoring capabilities for GridGain. It can be used to get statistics about nodes, caches and tasks in the grid. General details about the topology showing various metrics and node configuration properties can also be viewed here. CLI also allows you to start and stop remote nodes.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/VttAKhPLSR6MjxqlVZa9",
        "image2014-4-8 15-24-1.png",
        "770",
        "552",
        "#609bc7",
        ""
      ]
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
    "0-0": "`ack`",
    "0-2": "Acks arguments on all remote nodes.",
    "1-0": "`alert`",
    "1-2": "Alerts for user-defined events.",
    "2-0": "cache",
    "2-2": "Prints cache statistics, clears cache, prints list of all entries from cache.",
    "3-0": "close",
    "3-2": "Disconnects Visor console from the grid.",
    "4-2": "Prints node configuration.",
    "4-0": "config",
    "5-0": "deploy",
    "5-2": "Copies file or folder to remote host.",
    "6-2": "Prints topology change log.",
    "6-0": "disco",
    "7-2": "Print events from a node.",
    "7-0": "events",
    "8-0": "gc",
    "8-2": "Runs GC on remote nodes.",
    "9-0": "help",
    "9-1": "?",
    "9-2": "Prints Visor console help.",
    "10-0": "kill",
    "10-2": "Kills or restarts node.",
    "11-0": "log",
    "11-2": "Starts or stops grid-wide events logging.",
    "12-0": "mclear",
    "12-2": "Clears visor console memory variables."
  },
  "cols": 3,
  "rows": 25
}
[/block]