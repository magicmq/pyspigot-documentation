# 定义命令

PySpigot 允许您在脚本中定义和注册新命令。这些命令的功能与游戏内的任何命令相同。

有关如何将命令管理器导入到您的脚本中的说明，请参见 [一般信息](writingscripts#pyspigot-的管理器) 页面。

这不是一个全面的 Bukkit 命令指南。对于更完整的命令指南，请参阅 Spigot 关于命令的教程 [这里](https://www.spigotmc.org/wiki/create-a-simple-command/)。

# 命令管理器用法

命令管理器提供了多个函数供您注册和注销命令，包括：

- `registerCommand(command_function, name)`: 注册命令的最基本方法。
- `registerCommand(command_function, tab_function, name)`
- `registerCommand(command_function, name, description, usage)`
- `registerCommand(command_function, tab_function, name, description, usage)`
- `registerCommand(command_function, name, description, usage, aliases)`
- `registerCommand(command_function, tab_function, name, description, usage, aliases)`
- `registerCommand(command_function, tab_function, name, description, usage, aliases, permission, permission_message)`: 注册命令的最全面方法。如果命令执行者没有权限执行命令，则显示 `permission_message`。
- `unregisterCommand(name)`: 允许您通过名称从脚本注销命令。
- `unregisterCommand(command)`: 允许您从脚本注销命令。接受 `registerCommand` 函数返回的 `ScriptCommand` 对象。

?> 当您的脚本停止或卸载时，**不需要**注销您的命令。PySpigot 会为您处理这些。

# 代码示例

让我们看看以下定义并注册命令的代码：

```python
import pyspigot as ps

def kick_command(sender, label, args):
    #Do something...
    return True

def tab_kick_command(sender, alias, args):
    #Do something...
    return ['',]

registered_command = ps.command.registerCommand(kick_command, 'kickplayer')
```

第 1 行，我们导入 PySpigot 并将其作为 `ps` 使用，以利用命令管理器 (`command`)。

第 3 行，我们定义了一个名为 `kick_command` 的函数，它接受三个参数 `sender`、`label` 和 `args`。`sender` 是执行命令的对象，`label` 是输入的确切命令（如果命令有别名，则这是 `sender` 使用的别名）。最后，`args` 是一个包含 `sender` 在基本命令后键入的每个参数的字符串列表。

第 5 行，我们从函数返回一个布尔值。命令函数必须返回 `True` 或 `False`。不确定何时应返回 `True` 或 `False`？请参阅 [Spigot 关于命令的教程](https://www.spigotmc.org/wiki/create-a-simple-command/)。

第 7 行，我们定义了一个名为 `tab_kick_command` 的函数，它接受与 `kick_command` 相同的参数。此函数作为命令的补全功能。第 9 行，我们从这个函数返回一个字符串列表，作为可以被补全的选项。补全函数必须返回一个字符串列表。

所有命令都必须使用 PySpigot 的命令管理器进行注册才能工作。命令可以通过多种方式注册。在上面的代码中，使用了 `registerCommand(function, name)` 函数（第 11 行），它接受两个参数：

- 第一个参数接受当执行命令时应该调用的函数。
- 第二个参数是命令的名称，即一个字符串。

另外，在第 7 行，我们将 `registerCommand` 返回的值赋给 `registered_command`。这是一个 `ScriptCommand` 对象，代表已注册的命令。如果稍后想要注销命令，可以使用这个对象。

## 总结：{docsify-ignore}

- 命令在脚本中定义为函数。命令函数 *必须* 接受三个参数：发送者、标签和参数（这些参数名称可以自定义）。当执行命令时会调用该函数。
- 所有命令都必须使用 PySpigot 的命令管理器通过 `command.registerCommand(function, name)` 进行注册。
