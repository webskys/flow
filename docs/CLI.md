# Flow CLI

 > 关于单个命令的附加信息

`Flow`的命令行工具简单并且易于使用。

如果`.flowconfig`文件存在，使用`flow`命令将检测当前目录。如果需要，将自动启动`Flow`服务。

命令行工具也提供了许多其他选项或命令，允许你控制服务和创建集成工具。例如，这就是[Nuclide](https://nuclide.io/)编辑器如何与`Flow`集成，以便在其UI中提供自动完成、类型错误等。

要了解关于CLI的更多信息，只需输入：

 ```
 flow --help
 ```

这将给你反馈`Flow`所能做的所有信息，运行此命令应该打印出以下内容：

```
Usage: flow [COMMAND]

Valid values for COMMAND:
  ast             Print the AST
  autocomplete    Queries autocompletion information
  check           Does a full Flow check and prints the results
  check-contents  Run typechecker on contents from stdin
  coverage        Shows coverage information for a given file
  find-module     Resolves a module reference to a file
  get-def         Gets the definition location of a variable or property
  get-importers   Gets a list of all importers for one or more given modules
  get-imports     Get names of all modules imported by one or more given modules
  init            Initializes a directory to be used as a flow root directory
  server          Runs a Flow server in the foreground
  start           Starts a Flow server
  status          (default) Shows current Flow errors by asking the Flow server
  stop            Stops a Flow server
  suggest         Shows type annotation suggestions for given files
  type-at-pos     Shows the type at a given file and position
  version         Print version information

Default values if unspecified:
  COMMAND         status

Status command options:
  --color              Display terminal output in color. never, always, auto (default: auto)
  --from               Specify client (for use by editor plugins)
  --help               This list of options
  --json               Output results in JSON format
  --no-auto-start      If the server is not running, do not start it; just exit
  --old-output-format  Use old output format (absolute file names, line and column numbers)
  --one-line           Escapes newlines so that each error prints on one line
  --retries            Set the number of retries. (default: 3)
  --retry-if-init      retry if the server is initializing (default: true)
  --show-all-errors    Print all errors (the default is to truncate after 50 errors)
  --strip-root         Print paths without the root
  --temp-dir           Directory in which to store temp files (default: /tmp/flow/)
  --timeout            Maximum time to wait, in seconds
  --version            (Deprecated, use `flow version` instead) Print version number and exit
```

在自定义项目根目录的示例

```
mydir
├── frontend
│   ├── .flowconfig
│   └── app.js
└── backend
```

```
flow check frontend
```

然后，你可以添加一个`--help`标签更深入地钻研每个详细的命令

因此，例如，如果你想了解有关`autocomplete`的工作，你可以使用这样的命令：

```
flow autocomplete --help
```