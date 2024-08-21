# 脚本

本教程仅提供 PySpigot 的概述，并不详细涵盖 Bukkit/Spigot API 或编写 Python 代码的内容。有关 Python 语法或编写 Python 代码的一般问题应转至相应的论坛，因为本教程不会提供编写基本 Python 代码的介绍。

编写 PySpigot 脚本时，有几点基本事项需要注意：

- PySpigot 需要 Java 17 或更高版本。
- PySpigot 正式支持 Minecraft 版本 1.16 及以上版本的 Spigot 和 Paper。
- 在底层，PySpigot 使用 Jython，即 Python 的 Java 实现。目前，Jython 只实现了 Python 2，因此在编写 PySpigot 脚本时应使用 Python 2 语法。
- 脚本必须使用 Python 语法编写，脚本文件应以 `.py` 结尾。不以 `.py` 结尾的文件将不会被加载。
- 脚本放置在 PySpigot 插件目录下的 `scripts` 文件夹中。PySpigot 支持在 `scripts` 文件夹内创建子文件夹以方便组织，但所有子文件夹中的脚本名称必须唯一。
- 避免使用变量名 `global` 和 `logger`。这些变量名会在运行时自动分配。更多关于这些变量的信息如下所述。
- 脚本在功能上相互隔离。除了 `global` 变量（见下面的 [全局变量](#全局变量) 部分）外，脚本之间不共享任何内容。
- 若要使用 PySpigot 提供的任何管理器（如注册监听器、任务等），必须将它们导入到你的脚本中。有关如何利用 PySpigot 的管理器的详细信息，请参阅下面的“利用 PySpigot 的管理器”部分。
- 如果你正在使用除 ProtocolLib 或 PlaceholderAPI 以外的任何插件的 API，请确保在 `script_options.yml` 文件中指定了该插件为依赖项。有关更多信息，请参阅 [脚本选项](scriptoptions.md#plugin-depend) 页面。

## 关于 Jython 的说明

在底层，PySpigot 使用 Jython，即 Python 的 Java 实现。PySpigot 的 jar 文件与其他 Spigot 插件相比相当大，因为 Jython（及其依赖项）都被打包到了 PySpigot 中。

Jython 的设计使得脚本完全在 Java 中编译和解释。这意味着脚本在运行时具有对整个 Java 类路径的原生访问权限，使得与 Spigot API 和服务器的其他方面一起工作变得非常容易。考虑以下示例：

```python
from org.bukkit import Bukkit
from org.bukkit import Location

teleport_location = Location(Bukkit.getWorld('world'), 0, 64, 0)

online_players = Bukkit.getOnlinePlayers()

for player in online_players:
    player.teleport(teleport_location)
```

从上面的代码块可以看出，与 Java 类/对象一起工作非常直观。如果你在与 Java 交互时遇到任何困难，Jython 有一份写得很好的文档，你可以在此查阅：[这里](https://jython.readthedocs.io/en/latest/)。

目前，Jython 的最新版本实现了 Python 2。因此，目前 PySpigot 脚本是用 Python 2 编写的。虽然有些人可能认为这是一个缺点，但 Python 2 对于 PySpigot 的绝大多数用途来说通常是足够的，而且我还没有发现任何需要 Python 3 功能来实现脚本功能的情况。Jython 的开发人员打算在未来版本的 Jython 中实现 Python 3，但更新的时间框架尚不清楚。相关工作正在进行中，可以在 [Jython GitHub 仓库](https://github.com/jython/jython) 查看。

要了解更多信息，请访问 [jython.org](https://www.jython.org/)。

## 标准 Python 库

Jython *不支持* 许多内置的 Python 模块（即那些为 Python 用 C 语言编写的模块）。这些模块需要移植到 Java 或使用 JNI 桥接实现。一些内置模块已经被移植到 Jython，最显著的是 `cStringIO`、`cPickle`、`struct` 和 `binascii`。Jython 的文档指出，不太可能将 JNI 模块包含在 Jython 本身中。

Jython 现在支持标准 Python 库的大部分内容。然而，Jython 的文档在跟上这些新增内容方面进展缓慢，因此如果 Jython 的文档没有提及某个库，它可能仍然被支持。

如果你想在脚本中使用标准 Python 模块，尝试导入它。如果可以导入，那么你可能就可以使用了。你也可以调用 `dir()` 方法来检查模块实现的函数列表。

## 基础脚本信息

所有 PySpigot 脚本都是设计为 *自包含* 的单个文件。这意味着每个脚本最多只包含一个文件。此外，脚本之间是 *隔离* 的，意味着它们不共享变量、函数或作用域。脚本可以通过多种方式相互交互（下面会有更多细节），但可以将 `scripts` 文件夹中的每个 `.py` 文件视为一个独立的实体，在自己的环境中执行。

PySpigot 脚本放置在 `scripts` 文件夹中，该文件夹位于 PySpigot 的主插件文件夹中。支持在 `scripts` 文件夹内创建子文件夹以方便组织。PySpigot 将尝试加载 `scripts` 文件夹（包括子文件夹）中所有以 `.py` 扩展名结尾的文件。`scripts` 文件夹中不以 `.py` 结尾的任何文件都不会被加载。

!> 脚本名称必须唯一，因为它们的名称用于在运行时识别它们。如果你在 `scripts` 文件夹中使用子文件夹，这一点同样适用。例如，`scripts/folder1/test.py` 和 `scripts/folder2/test.py` 会产生冲突，但 `scripts/folder1/test.py` 和 `scripts/folder2/test2.py` 不会。

## 脚本选项

对于每个脚本，可以设置各种选项，包括是否启用、加载优先级和日志记录选项。这些选项在 PySpigot 主插件文件夹中的 `script_options.yml` 文件中设置。有关脚本选项的更多信息，请参阅 [脚本选项](scriptoptions.md) 页面。

!> 为每个脚本定义脚本选项是 *可选的*；如果没有明确定义选项，脚本仍能正常运行。

## 脚本加载

PySpigot 在插件加载或服务器启动时自动加载和运行 `scripts` 文件夹（包括子文件夹中的脚本）中的所有脚本。脚本加载顺序由 `script_options.yml` 文件中定义的加载优先级确定。未列出任何加载优先级的脚本将继承 `config.yml` 中指定的默认加载优先级。具有相同加载优先级的脚本将按字母顺序加载。

你也可以使用 `/pyspigot load <scriptname>` 手动加载脚本，如果你想要在服务器启动/插件加载后加载/启用脚本。如果你在运行时对脚本进行了更改，则必须重新加载脚本才能使更改生效。使用 `/pyspigot reload <scriptname>` 重新加载脚本。

有一个与加载脚本相关的配置选项：

- `script-load-delay`：这是 PySpigot 在 **服务器加载完成** 后等待加载脚本的延迟（以刻为单位）。一秒钟有 20 个服务器刻。例如，如果值为 20，则 PySpigot 将在服务器加载完成后等待 20 刻（或 1 秒）再加载脚本。

稍后将添加更多关于此模块的文档。

## 脚本权限

PySpigot 允许脚本定义它们使用的权限列表。这对于限制对某些功能的访问非常有用。脚本权限在解析和执行脚本代码之前初始化和加载，并在脚本停止后立即移除。

脚本权限在 `script_options.yml` 文件中定义。有关如何定义权限的更多信息，请参阅 [脚本选项文档](scriptoptions.md#permissions)。

## 启动和停止函数

你可以包含两个特殊的函数在你的 PySpigot 脚本中：`start` 和 `stop`。这两个函数都不接受参数。

如果你的脚本中定义了一个 `start` 函数，当脚本启动时，PySpigot 将调用它。

如果你的脚本中定义了一个 `stop` 函数，当你的脚本停止/卸载时，PySpigot 将调用它。

?> `start` 和 `stop` 都是可选的，如果不需要的话，你不必在脚本中定义它们。

## PySpigot 辅助模块

从版本 0.5.0 开始，PySpigot 包含了一个名为 `pyspigot.py` 的辅助模块，其中包含各种有用的函数，用于访问 PySpigot 的管理器类。此模块会在插件加载时自动放置到 `python-libs` 文件夹中。

PySpigot 包括一个自动化系统，当检测到变化时会自动更新 `pyspigot.py` 辅助模块。如果需要，可以通过将 `config.yml` 文件中的 `debug-options` 下的 `auto-pyspigot-lib-update-enabled` 选项设置为 `false` 来禁用此功能。建议保持启用状态以确保任何更改、新增功能和修复都能在用户端得到反映。

[点击此处](https://github.com/magicmq/pyspigot/blob/master/src/main/resources/python-libs/pyspigot.py) 查看 `pyspigot.py` 模块的源代码。

## PySpigot 的管理器

PySpigot 提供了一系列管理器，以便更轻松地处理 Bukkit/Spigot API 的各个部分。有关如何将这些管理器导入到你的脚本中的说明，请参阅下面的内容。PySpigot 当前提供的管理器包括：

- ScriptManager，用于从另一个脚本中加载和卸载脚本。
- ListenerManager，用于注册事件监听器。
- CommandManager，用于注册和处理命令。
- TaskManager，用于注册各种重复、延迟和异步任务。
- ConfigManager，用于处理配置文件。
- ProtocolManager，用于与 ProtocolLib 工作。
- PlaceholderManager，用于与 PlaceholderAPI 工作。
- DatabaseManager，用于连接和交互 SQL 类型和 MongoDB 数据库。
- RedisManager，用于连接和交互 Redis 服务器实例。

为了能够访问这些管理器，你需要将它们导入到你的脚本中。下表总结了如何访问管理器，但请阅读下面的部分以获取更多关于如何导入它们的详细信息：

| 管理器      | 通过辅助模块访问         | 通过 PySpigot 访问       | 单独导入                                                         |
|----------| ------------------------ | ------------------------ | ---------------------------------------------------------------- |
| 脚本管理器    | `pyspigot.script_manager()` | `PySpigot.script`         | `from dev.magicmq.pyspigot.manager.script import ScriptManager`   |
| 事件管理器    | `pyspigot.listener_manager()` | `PySpigot.listener`       | `from dev.magicmq.pyspigot.manager.listener import ListenerManager` |
| 命令管理器    | `pyspigot.command_manager()` | `PySpigot.command`        | `from dev.magicmq.pyspigot.manager.command import CommandManager` |
| 任务管理器    | `pyspigot.task_manager()`   | `PySpigot.scheduler`      | `from dev.magicmq.pyspigot.manager.task import TaskManager`       |
| 配置管理器    | `pyspigot.config_manager()` | `PySpigot.config`         | `from dev.magicmq.pyspigot.manager.config import ConfigManager`   |
| 协议管理器    | `pyspigot.protocol_manager()` | `PySpigot.protocol`       | `from dev.magicmq.pyspigot.manager.protocol import ProtocolManager` |
| 占位符管理器   | `pyspigot.placeholder_manager()` | `PySpigot.placeholder` | `from dev.magicmq.pyspigot.manager.placeholder import PlaceholderManager` |
| 数据库管理器   | `pyspigot.database_manager()` | `PySpigot.database`       | `from dev.nagicmq.pyspigot.manager.database import DatabaseManager` |
| Redis管理器 | `pyspigot.redis_manager()`   | `PySpigot.redis`          | `from dev.magicmq.pyspigot.manager.redis import RedisManager`     |

!> Protocol Manager 和 Placeholder Manager 是 *可选* 管理器。只有在 ProtocolLib 和/或 PlaceholderAPI 插件已加载并启用的情况下才应访问这些管理器。

为了使用这些管理器，必须将它们导入到你的脚本中。这可以通过三种方式完成：

### 通过 `pyspigot.py` 辅助库导入管理器

从版本 0.5.0 开始，PySpigot 包含了一个 `pyspigot.py` 辅助模块，该模块会在插件初始化时自动放置到 `python-libs` 文件夹中。此模块包含多个函数，允许访问所有管理器。

```python
import pyspigot as ps

ps.script_manager().<function>
ps.listener_manager().<function>
ps.command_manager().<function>
ps.task_manager().<function>
ps.config_manager().<function>
ps.protocol_manager().<function>
ps.placeholder_manager().<function>
ps.database_manager().<function>
ps.redis_manager().<function>
```

在上面的代码中，PySpigot 库被导入为 `ps`。然后，通过调用库内的函数来获取每个管理器。当然，你也可以将所需的管理器赋值给变量，以便在代码的多个位置方便使用，如下所示：

```python
import pyspigot as ps

script = ps.script_manager()
listener = ps.listener_manager()
command = ps.command_manager()

...
script.<function>
listener.<function>
command.<function>
...
```

PySpigot 辅助模块还包括一系列方便的变量，以便于访问管理器。你会在 PySpigot 文档中的示例代码中看到这些常见的别名。

```python
import pyspigot as ps
```

- 脚本管理器: `ps.script`, `ps.scripts`, `ps.sm`
- 事件管理器: `ps.listener`, `ps.listeners`, `ps.lm`, `ps.event`, `ps.events`, `ps.em`
- 命令管理器: `ps.command`, `ps.commands`, `ps.cm`
- 任务管理器: `ps.scheduler`, `ps.scm`, `ps.tasks`, `ps.tm`
- 配置管理器: `ps.config`, `ps.configs`, `ps.com`
- 协议管理器: `ps.protocol`, `ps.protocol_lib`, `ps.protocols`, `ps.pm`
- 占位符管理器: `ps.placeholder`, `ps.placeholder_api`, `ps.placeholders`, `ps.plm`
- 数据库管理器: `ps.database`
- Redis管理器: `ps.redis`

### 使用 PySpigot 类一次性导入所有管理器

这曾经是访问管理器的首选方法，但从版本 0.5.0 开始不再作为首选方法。不过，这种方法仍然有效，可以根据需要使用。

```python
from dev.magicmq.pyspigot import PySpigot as ps

ps.script.<function>
ps.listener.<function>
ps.command.<function>
ps.scheduler.<function>
ps.config.<function>
ps.protocol.<function>
ps.placeholder.<function>
```

在上面的代码中，PySpigot 被导入为 `ps`。使用简化的名称来调用管理器，如 `script` 对应 ScriptManager，`listener` 对应 ListenerManager，`command` 对应 CommandManager，`scheduler` 对应 TaskManager，`config` 对应 ConfigManager，以及 `protocol` 对应 ProtocolManager。

### 单独导入每个管理器

你也可以直接从 PySpigot 的 Java 代码中单独导入每个管理器类。

```python
from dev.magicmq.pyspigot.manager.script import ScriptManager as script
from dev.magicmq.pyspigot.manager.listener import ListenerManager as listener
from dev.magicmq.pyspigot.manager.command import CommandManager as command
from dev.magicmq.pyspigot.manager.task import TaskManager as scheduler
from dev.magicmq.pyspigot.manager.config import ConfigManager as config
from dev.magicmq.pyspigot.manager.protocol import ProtocolManager as protocol

script.get().<function>
listener.get().<function>
command.get().<function>
...
```

!> 如果单独导入管理器，则每次调用管理器时 *必须* 使用 `get()`！

## 全局变量

PySpigot 为所有加载的脚本分配了一个称为 `global` 的本地命名空间变量。在 Java 端，这个变量是一个 `HashMap`，存储键值对数据，类似于 Python 中的字典。这个系统的目的是充当一组全局变量。如果你希望在多个不同的脚本之间共享变量/值，这是一个很实用的功能。

有关如何使用全局变量系统的详细信息，请参阅 [全局变量](globalvariables.md) 页面。

## 脚本错误

脚本可能会产生错误/异常。PySpigot 将尝试处理这些错误以防止脚本的其他部分崩溃。如果脚本加载时产生了未处理的错误/异常，脚本将自动卸载以防止进一步的问题。如果在稍后的某个时刻，在调用事件监听器或命令函数时发生未处理的错误/异常，脚本将继续加载，但在函数中的后续代码将不会被执行。无论如何，错误/异常都会被记录到控制台和相应的脚本日志文件中（如果在 `config.yml` 中启用了文件日志记录）。你可以使用 Python 的 `try:` 和 `except:` 语法来自己处理异常。这对于 Java 异常也同样适用。

脚本可能产生的错误有两种类型：

### Python 异常

这些是脚本 Python 代码内在的异常。这些异常会产生一条带有 Python 跟踪回溯的日志条目，指示导致异常的脚本文件和行。因为这些异常源自 Python 代码，所以应该比较容易调试。它们看起来像这样：

![Python 异常在服务器控制台中的示例](images/python_exception.png)

方框中的文本是 Python 的跟踪回溯。

### Java 异常

这些异常发生在脚本调用 Java 代码时，并且异常发生在 Java 代码中的某个地方（但不是在脚本内部）。这些异常也会产生一条带有 Python 跟踪回溯的日志条目，指示导致异常的脚本文件和行。这些异常可能更难调试，因为异常的原因可能不明显。脚本日志/控制台应该能给你一些关于发生了什么问题的线索。它们看起来像这样：

![Java 异常在服务器控制台中的示例](images/java_exception.png)

你会注意到这些异常看起来非常类似于 Python 异常。唯一的区别是会有伴随的 Java 异常 (`java.lang.<exception>`) 以及关于异常发生原因的简短消息。

### 关于异常的最终说明

由于 PySpigot 是一个处于早期开发阶段的活跃项目，你可能会遇到由 PySpigot 自身的 bug 所引起的异常。如果你的脚本出现问题，并且你的调试努力没有成功，请在 [GitHub 上提交问题](https://github.com/magicmq/PySpigot/issues)。

## 脚本日志记录

就像脚本本身一样，脚本的日志记录也是独立的。每个脚本都有自己的日志记录器，它是 [java.util.logging.Logger](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Logger.html) 的子类。

PySpigot 为每个运行中的脚本创建一个新的日志记录器。脚本的日志记录器自动分配给其全局命名空间下的变量 `logger`。要访问你的脚本的日志记录器，请使用 `logger` 变量。

如果你想在脚本中打印日志消息，强烈建议使用脚本相应的日志记录器。使用 Python 的 `print` 函数也可以工作，但它不会自动表明消息来自哪个脚本（除非你自己在消息中添加这些信息）。`print` 也不会将消息记录到脚本的日志文件中。

当你访问脚本的日志记录器时，你可以使用 [这里](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Logger.html) 列出的所有函数。PySpigot 添加了两个额外的函数以方便使用：

- `logger.print(message)`：用于快速向脚本添加调试消息
- `logger.debug(message)`：底层功能与 `logger.print` 相同

### 脚本日志文件

如上所述，每个脚本都有自己的日志文件，这些文件可以在 PySpigot 插件文件夹内的 `logs` 文件夹中找到。所有与脚本相关的消息都将记录在这里。如果你想禁用脚本的日志文件记录，可以将 PySpigot `config.yml` 中的 `log-to-file` 值设置为 `false`。

你也可以改变哪些消息会被记录到脚本的日志文件中。为此，请编辑 `config.yml` 中的 `min-log-level` 值。使用 Java 的 [日志级别](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/Level.html)。

你还可以改变脚本日志文件中时间戳的格式。为此，请编辑 `config.yml` 中的 `log-timestamp-format` 值。使用符合 Java 的 [DateTimeFormatter](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html) 的格式。

## 脚本文件中的非 ASCII 字符

Jython 使用 ASCII 编码读取和编译脚本文件。这意味着它不会识别文件中的非 ASCII 字符。当你加载包含非 ASCII 字符的脚本时，可能会看到 `SyntaxError`。此外，在 Python 2 中，`str` 类型是一组 8 位字符。因此，英语字母表中的所有字符（以及一些基本符号）都可以使用这些 8 位字符表示，但特殊符号和非拉丁字母表的字符则不能。有几种方法可以解决这两个限制：

### 解决方案 1

Jython 允许你指定脚本文件的编码。这是通过在脚本中指定 [编码声明](https://docs.python.org/2/reference/lexical_analysis.html#encoding-declarations) 来实现的。这确保了当 Jython 读取和编译文件时，它将识别非 ASCII 字符。将以下内容添加到脚本文件的 *第一行或第二行*：

`#coding: utf-8`

当然，你可以将 `utf-8` 替换为你希望 Jython 使用的任何字符编码标准。支持的编码方案列表，请参见 [此页面](https://docs.python.org/2/library/codecs.html#standard-encodings)。

!> 必须将编码声明放在脚本的第一行或第二行，因为 Jython 只在这个区域搜索编码声明。

Python 2 包含一个 `unicode` 类型，支持所有 UTF-8 字符（符号、非拉丁字母等）。你可以在字符串前面加上一个前置 `u` 来指定你想使用 `unicode` 类型（而不是 `str`）。例如：

```python
#coding: utf-8

from org.bukkit import Bukkit

Bukkit.broadcastMessage(u'Привет, мир!')
```

注意字符串前面有一个 `u`。这表示字符串应该被视为 unicode 字符串。

### 解决方案 2

你也可以使用你想写的 unicode 字符的十六进制代码。你仍然需要通过在字符串前面加上一个前置 `u` 来标明字符串是 unicode 字符串，但是你不需要在脚本文件顶部添加编码声明，因为你实际上并没有在文件中写入任何非 ASCII 字符。例如：

```python
from org.bukkit import Bukkit

Bukkit.broadcastMessage(u'\u041f\u0440\u0438\u0432\u0435\u0442, \u043c\u0438\u0440!')
```