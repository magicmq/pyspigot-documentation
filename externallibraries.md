# 使用外部 Java/Python 库

PySpigot 允许你在编写脚本时使用外部 Python 和 Java 库。有许多开源的 Python 和 Java 库可以简化或加速代码编写过程。例如，[Apache Commons Libraries](https://commons.apache.org/) 是一系列 Java 实用库，提供了大量的实用工具函数。

当然，更高级的用法可能是为你经常使用的或复杂的代码编写自己的外部库。有关这方面的更多信息，请参阅下面的 [外部 Python 模块的便捷使用](#加载和使用外部-Python-模块) 部分。

## 一般信息

当 PySpigot 加载时，它会创建两个文件夹（如果它们还不存在的话），分别叫做 `python-libs` 和 `java-libs`，用于存放外部库。

PySpigot 会在插件加载时自动将一个辅助模块 `pyspigot.py` 放入 `python-libs` 文件夹中。关于这个辅助库的更多信息，请参阅 [编写脚本](writingscripts#the-pyspigot-helper-module) 页面。

## 外部 Python 库

PySpigot 目前支持使用 *单模块文件* 作为外部库。目前还不支持完全成熟的库/包（即那些带有 `__init__.py` 的），但计划在未来版本中增加对这些的支持。因此，从现在起，“外部 Python 库”将被称为 **模块**。

### 加载和使用外部 Python 模块

使用外部 Python 模块相对简单，并不需要使用 PySpigot 的管理器或任何命令，与 Java 库不同。你打算使用的外部 Python 模块应该是一个单一的 `.py` 文件。要使用它，只需将它拖放到 PySpigot 主插件文件夹内的 `python-libs` 文件夹中。然后，在你的脚本中简单地 `import` 这个模块即可。

?> 添加新模块或删除旧模块不需要重启服务器或重新加载插件。然而，如果模块的代码有所更改，则所有使用该模块的脚本都应重新加载。

### 外部 Python 模块的便捷使用

假设有一段代码是你在多个脚本中频繁使用的。这段代码可能包括将玩家的位置转换为配置友好的 `dict`，格式化一个 `str` 字符串，或者以整洁的格式向玩家发送消息。与其在不同的脚本中多次编写相同的代码，你可以考虑将这段代码拆分成自己的外部 Python 模块，然后将其添加到 `python-libs` 文件夹中，这样就可以从所有脚本中访问它了。

## 外部 Java 库

通常情况下，Java 库会被“打包”进 Bukkit/Spigot 插件中（最终结果有时称为“胖”Jar 或 “Uber”Jar），以便插件在运行时可以访问依赖项。显然我们不能直接这样做对于脚本。

相反，我们可以利用 Jython 的能力。如文档其他部分所述，Jython 提供了在运行时访问所有已加载 Java 类的能力。因此，可以通过手动在运行时将一组 Java 类（即库）加载到类路径中，从而让脚本能够访问这些类。PySpigot 的 `LibraryManager` 提供了这种功能。

当一个 JAR 库被加载时，PySpigot 会创建一个以 `-relocated` 结尾的库副本。例如，库 `jython-annotation-tools-0.9.0.jar` 会被复制为 `jython-annotation-tools-0.9.0-relocated.jar`。这样做不仅是为了适应重定位规则（见下面的 [JAR 重定位规则](#JAR-重定位规则) 部分），也是为了加快未来插件加载时外部库的加载速度。即使你不为库指定任何重定位规则，这种行为也会发生。

### 加载和使用 Java 库

!> 可能服务器上另一个插件已经在使用你想使用的库/依赖项。在尝试自己加载库之前，你应该检查是否已经有插件在使用它。你可以尝试在脚本中 `import` 该依赖项。如果能够导入成功，**就到这里为止！** 你无需自己加载它，因为它已经被加载（很可能是由另一个插件加载的）。

所有的 Java 库/依赖项 *应该* 都可以从某个地方下载到 Jar 文件，比如在其 Github 仓库或官方网站上。你可能需要从托管该依赖项的仓库手动下载它。如果你找不到 Jar 文件，请在 PySpigot 的 Discord 上寻求帮助。

一旦你有了想要使用的依赖项的实际 Jar 文件，只需将其拖放到 PySpigot 主插件文件夹中的 `java-libs` 文件夹内。当 PySpigot 插件加载并启用（服务器启动时）时，该依赖项会被加载。从这里开始，你可以在脚本中使用 `import` 语句来导入库中的所需代码。

?> `java-libs` 文件夹中的所有 Jar 文件都会在插件加载时（服务器启动时）被加载。在 PySpigot 已经运行的情况下加载新的 Jar 文件需要使用命令（见下文），或者完全重新加载插件（使用 `/ps reloadall`）。

#### 在 PySpigot 已经运行时加载 Java 库

像之前一样，获取你想要使用的库的 Jar 文件。然后，将其拖放到 PySpigot 主插件文件夹中的 `java-libs` 文件夹内。

接下来，你需要使用命令 `/ps loadlibrary <filename>` 手动加载库，其中 `<filename>` 是你要加载的 Jar 文件的名称。确保文件名中包含 `.jar` 扩展名。这将会导入库。

现在库应该已经被加载，你可以使用 `import` 语句来导入所需的代码。

!> 由于 Java 类加载系统的限制，PySpigot 没有办法卸载已经加载的库。不过，库不会被加载超过一次；如果它已经被加载过，PySpigot 将不会再次加载它。

### JAR 重定位规则

你可能需要改变库/依赖项中的类名/类路径。这称为“重定位”。有很多原因可能会让你想这样做，但这里不深入讨论。如果你需要重定位你的库/依赖项的类路径，你可能已经知道你需要这样做。在这种情况下，请继续阅读有关如何指定重定位规则的信息。

在 PySpigot 的 `config.yml` 文件中，你会找到一个 `library-relocations` 的值。你可以在这里指定自己的重定位规则。重定位规则应采用 `<path>|<relocated-path>` 的格式。例如，要将 `com.apache.commons.lang3.stringutils` 重定位到 `lib.com.apache.commons.lang3.stringutils`：

```yaml
...
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations:
  - 'com.apache.commons.lang3.stringutils|lib.com.apache.commons.lang3.stringutils'
...
```

在 YAML 语法中也允许使用简化的列表形式：

```yaml
...
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations: ['com.apache.commons.lang3.stringutils|lib.com.apache.commons.lang3.stringutils']
...
```

要在简化的列表形式中添加多个重定位规则，每个规则之间用逗号隔开。

!> 如前所述，PySpigot 会对每个外部 JAR 库创建一个以 `-relocated` 结尾的副本。这个文件代表了重定位后的 JAR，所有重定位规则都已应用到它上面。这个副本只创建一次，或者说，重定位规则只应用一次。在后续的插件加载过程中，加载的是重定位后的 JAR 库。因此，如果对重定位规则进行了更改，则有必要删除相关库的 `-relocated.jar` 副本，以便更新后的重定位规则得到应用。
