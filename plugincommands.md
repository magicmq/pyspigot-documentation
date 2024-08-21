# PySpigot 命令

PySpigot 提供了多个命令，主要用于加载、卸载和重载脚本。以下是这些命令的文档。

通常情况下，当一个命令需要脚本名称时，指的是文件名本身。即使脚本文件位于子文件夹中也是如此。例如，如果你有一个路径为 `scripts/test/test.py` 的脚本，你应该只通过文件名（`test.py`）来引用它。要加载它，你可以运行命令 `/pyspigot load test.py`。PySpigot 会根据你提供的名称自动在 `scripts` 文件夹中搜索匹配的脚本文件，因此不需要指定脚本文件的完整路径。

PySpigot 的基础命令是 `/pyspigot`。

- 语法: `/pyspigot <argument>`
- 别名: `ps`
- 权限: `pyspigot.command.listcmds`

## 帮助命令

显示多个有用链接，包括文档（本网站）、Spigot 上的 PySpigot 插件页面、官方 Discord 频道以及在 GitHub 上报告问题/错误的链接。

- 语法: `/pyspigot help`
- 别名: `gethelp`
- 权限: `pyspigot.command.help`

## 信息命令

显示有关脚本的详细信息，包括运行时间、已注册的监听器、命令、任务等更多信息。

- 语法: `/pyspigot info <scriptname>`
- 别名: `scriptinfo`
- 权限: `pyspigot.command.info`

## 列出脚本命令

列出所有脚本，包括未加载和已加载的。已加载的脚本显示为绿色，未加载的脚本显示为红色。如果有多个页面可用，可以使用 `[page]` 跳转到其他页面。

- 语法: `/pyspigot listscripts [page]`
- 别名: `list`, `scriptslist`, `ls`
- 权限: `pyspigot.command.listscripts`

## 加载命令

加载并运行一个脚本。

- 语法: `/pyspigot load <scriptname>`
- 别名: `start`
- 权限: `pyspigot.command.load`

## 加载库命令

加载你希望在脚本中使用的 Java 库。更多信息请参见 [外部库](librarymanager.md)。

- 语法: `/pyspigot loadlibrary <filename>`
- 别名: `loadlib`
- 权限: `pyspigot.command.loadlibrary`

## 重载全部命令

执行 PySpigot 的完全重载，包括 `config.yml`、`script_options.yml`、库管理器和所有脚本。

- 语法: `/pyspigot reloadall`
- 别名: `reset`, `restart`, `reboot`, `resetall`
- 权限: `pyspigot.command.reloadall`

## 重载命令

重载一个脚本。如果你在脚本运行时对其进行了修改，并希望重新加载它，这将非常有用。

- 语法: `/pyspigot reload <scriptname>`
- 别名: 无
- 权限: `pyspigot.command.reload`

## 重载配置命令

重载 PySpigot 的 `config.yml` 和 `script_options.yml`。

- 语法: `/pyspigot reloadconfig`
- 别名: `configreload`
- 权限: `pyspigot.command.reloadconfig`

!> 这个命令不会重载脚本！使用 `/pyspigot reload <scriptname>` 来重载脚本。

## 卸载命令

停止并卸载一个脚本。

- 语法: `/pyspigot unload <scriptname>`
- 别名: `stop`
- 权限: `pyspigot.command.unload`
