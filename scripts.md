# 脚本管理器

脚本管理器可以说是 PySpigot 的核心大脑。它包含了加载、运行和卸载脚本的代码。它还充当了一个管理者，监督脚本命令、监听器和任务等。在正常情况下，你不应该需要直接与脚本管理器交互，因为所有这些都是在后台自动完成的。然而，如果你需要从一个脚本内部加载/运行或卸载另一个脚本，脚本管理器是可以供你使用的。

请参阅 [一般信息](writingscripts#PySpigot-的管理器) 页面以获取如何将脚本管理器导入到你的脚本中的说明。

# 使用脚本管理器

使用脚本管理器来加载/运行和卸载脚本相对简单。有三个可用的函数来实现这一点：

- `script.loadScript(name)`: 加载具有给定名称的脚本。返回一个表示加载脚本结果的 `RunResult`。有关 `RunResult` 的更多信息，请参见以下部分。
- `script.loadScript(path)`: 加载具有给定文件路径的脚本。返回一个表示加载脚本结果的 `RunResult`。有关 `RunResult` 的更多信息，请参见以下部分。
- `script.unloadScript(name)`: 卸载具有给定名称的脚本。如果脚本成功卸载，此函数将返回 `True`；如果脚本未能成功卸载，则返回 `False`。不成功的卸载通常会伴随着打印到控制台和脚本日志文件（如果启用了文件日志记录）的错误消息。

!> *永远不要* 从脚本内部卸载自身。这 *将会* 导致意外行为和错误。

## 运行结果

`RunResult` 表示运行脚本的结果。有六种可能的结果：

- `RunResult.SUCCESS`: 运行脚本成功，并且脚本当前正在运行。你可以安全地假设如果这是 `RunResult`，那么脚本正在无错误地运行。
- `RunResult.FAIL_DISABLED`: 运行脚本失败，因为脚本在其选项中（在 `script_options.yml` 中）被禁用。
- `RunResult.FAIL_PLUGIN_DEPENDENCY`: 运行脚本失败，因为它依赖于一个或多个未运行的 *插件*。
- `RunResult.FAIL_ERROR`: 运行脚本失败，因为在运行时出现了错误。此错误将被打印到服务器控制台以及脚本的日志文件（如果启用了文件日志记录）。
- `RunResult.FAIL_DUPLICATE`: 运行脚本失败，因为已经有一个同名的脚本正在运行。
- `RunResult.FAIL_SCRIPT_NOT_FOUND`: 运行脚本失败，因为没有找到具有给定名称的脚本。

`RunResult` 类可以从 `dev.magicmq.pyspigot.manager.script.RunResult` 访问。

# 代码示例

让我们看一些加载、运行和卸载脚本的代码：

## 加载和运行脚本

```python
import pyspigot as ps
from dev.magicmq.pyspigot.manager.script import RunResult

run_result = ps.script.loadScript('script.py')

if (run_result == RunResult.SUCCESS):
	print('The script is running!')
elif (run_result == RunResult.FAIL_ERROR):
	print('The script did not load due to an error!')
```

第 1 行，我们导入 PySpigot 作为 `ps` 以便使用脚本管理器 (`script`)。

第 2 行，我们导入 `RunResult` 以便稍后确定加载脚本的结果。

第 4 行，我们加载名为 `script.py` 的脚本。我们将返回的值赋给 `run_result`。

第 6 行，我们检查加载脚本的结果是否为 `RunResult.FAIL_ERROR`（这意味着由于错误脚本未能加载），如果是这种情况，则向控制台打印一条消息。

第 8 行，我们检查加载脚本的结果是否为 `RunResult.SUCCESS`（这意味着脚本成功加载并且当前正在运行），如果是这种情况，则向控制台打印一条消息。

## 卸载脚本

```python
import pyspigot as ps

unload_result = ps.script.unloadScript('script.py')

if (unload_result == True):
	print('The script unloaded successfully without errors!')
```

第 1 行，我们导入 PySpigot 作为 `ps` 以便使用脚本管理器 (`script`)。

第 3 行，我们卸载名为 `script.py` 的脚本。我们将返回的值（根据结果为 `True` 或 `False`）赋给 `unload_result`。

第 5 行，我们检查卸载脚本的结果是否为 `True`（这意味着脚本已成功无误地卸载），如果是这种情况，则进入 if 语句。

!> 在通过其他脚本加载/卸载脚本时要 *极其小心*。这是一个强大但潜在破坏性的特性，可能会导致意外的行为和错误。

## 总结：

- 使用脚本管理器从一个脚本内部加载/运行和卸载另一个脚本。
- 必须使用 `loadScript` 和 `runScript` 来加载和运行脚本，因为加载和运行是两个独立的操作。
- `loadScript` 返回一个脚本对象，该对象应该传递给 `runScript`。如果脚本未能成功加载，则返回 `None`。
- `runScript` 返回一个 `RunResult`，可以用来确定运行脚本的结果。
- `unloadScript` 返回 `True` 或 `False`，其中 `True` 表示脚本成功卸载，`False` 表示未能成功卸载。
- *永远不要* 尝试从脚本内部卸载自身。这会导致 bug 和错误。
