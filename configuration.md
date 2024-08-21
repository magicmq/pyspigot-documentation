# 配置文件

使用 PySpigot，您的脚本可以加载、访问和保存到配置文件。所有使用配置管理器访问的配置文件都会自动存储在 PySpigot 插件文件夹内的 `config` 文件夹中。

有关如何将配置管理器导入到您的脚本中的说明，请参见 [一般信息](writingscripts#PySpigot-的管理器) 页面。

这不是一个全面的配置文件操作指南。对于可用方法/函数的更完整文档，请参阅 [Javadocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html)。这里列出的所有方法都可以在您的脚本中调用。

# 配置管理器用法

以下是从配置管理器中可用的函数：

- `doesConfigExist(filePath)`: 检查是否有一个配置文件存在给定的名称或路径下。如果文件存在则返回 `True`，不存在则返回 `False`。
- `loadConfig(filePath)`: 加载配置文件，并且如果文件尚不存在则自动创建。接受您希望加载和/或创建的文件的名称或路径。返回一个表示已加载/创建的配置的 `ScriptConfig` 对象。不指定默认值。
- `loadConfig(filePath, defaults)`: 加载配置文件，并且如果文件尚不存在则自动创建。接受您希望加载和/或创建的文件的名称或路径。返回一个表示已加载/创建的配置的 `ScriptConfig` 对象。还接受一个 YAML 格式的字符串 (`defaults`)，用于指定配置所需的默认值。
- `reloadConfig(config)`: 如果文件有任何需要加载的变化，则重新加载配置。接受要重新加载的配置（一个 `ScriptConfig` 对象）。返回另一个表示已重新加载的配置的 `ScriptConfig` 对象。
  - *注意:* `reloadConfig` 函数将在未来的 PySpigot 版本中移除。相反，应在 `ScriptConfig` 中使用 `reload` 函数（下面详细说明）。
- `deleteConfig(filePath)`: 删除具有指定名称或路径的配置文件。如果删除了文件则返回 `True`，如果之前提供的名称或路径下不存在文件则返回 `False`。
- `getConfigFolder()`: 返回一个指向配置文件所在的文件夹（位于 PySpigot 插件文件夹内）的路径。

?> 配置管理器允许在处理脚本配置文件时使用子文件夹以达到组织的目的。只需将配置文件作为路径传递给上述接受 `filePath` 参数的方法即可。例如，`ps.config_manager.loadConfig('test_script/config.yml')` 将加载 `configs` 文件夹内的 `test_script` 子文件夹中的 `config.yml` 文件。

# ScriptConfig 用法

加载/重新加载配置返回一个 `ScriptConfig` 对象。这个对象有许多您可以使用的方法/函数：

- 有许多函数可用于从您的配置中获取数据。要获取从配置文件中检索数据可以使用的完整方法/函数列表，请参阅 [Spigot JavaDocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html)
- `set(key, value)`: 在配置中设置给定键处的值。接受一个键，表示要写入的键，以及一个值，表示要写入的值。
- `setIfNotExists(path, value)`: 只有在给定路径处尚未设置值的情况下才在配置中设置值。如果路径被设置为值（路径之前未设置），则返回 `True`；如果路径 *未* 被设置为给定的值（路径之前已设置），则返回 `False`。如果您希望向配置文件添加新的键，但不想覆盖它们如果它们已经设置的话，这很有用。
- `getConfigFile()`: 获取与配置对应的文件。
- `getConfigPath()`: 获取与配置对应的文件的路径。
- `load()`: 加载配置文件。在正常情况下，不应调用此函数，因为它在从 ScriptManager 调用 `loadConfig` 时完成。
- `reload()`: 从文件重新加载配置。如果自上次加载/重新加载以来对文件进行了更改，则很有用，因为这些更改不会自动反映出来，直到配置被重新加载。
- `save()`: 保存配置，使您设置的任何值都是持久的。保存后会自动重新加载配置。

!> 对配置所做的更改不会自动保存到文件！您 *必须* 调用 `save` 以将更改写入配置文件。

# 默认配置值

Bukkit 配置库（PySpigot 的配置系统基于此构建）内置了一套处理默认配置值的系统。默认值是在加载配置时可以指定的值，当配置文件中没有定义某个值时，配置系统会回退使用这些默认值。这些默认值仅在配置文件中不存在相应的路径/键时使用。请看以下示例：

配置文件：

```yaml
key-1: 'Key 1 is set'
key-2: 'Key 2 is set'
```

脚本：

```python
import pyspigot as ps

config_defaults = """
key-1: 'Key 1'
key-2: 'Key 2'
key-3: 'Key 3'
"""

config = ps.config.loadConfig('script_config.yml', config_defaults)

print(config.getString('key-1'))
print(config.getString('key-2'))
print(config.getString('key-3'))
```

控制台输出：

```
[14:17:22 INFO]: [PySpigot/test.py] [STDOUT] Key 1 is set
[14:17:22 INFO]: [PySpigot/test.py] [STDOUT] Key 2 is set
[14:17:22 INFO]: [PySpigot/test.py] [STDOUT] Key 3
```

在上面的例子中，定义了一个 YAML 格式的多行字符串，其中包含了所有期望的默认值。然后加载配置文件，并适当地传递默认配置字符串。接着打印出 `key-1`、`key-2` 和 `key-3` 的值。从控制台输出可以看出，`key-1` 和 `key-2` 分别打印出 "Key 1 is set" 和 "Key 2 is set"，因为这些值实际上在配置文件中定义了。然而，由于 `key-3` 在配置文件中未定义，所以它默认使用了 `key-3` 的默认值 "Key 3"，这是在 `config_defaults` 字符串中预先指定的。

指定默认值特别有用，如果你想要向现有的配置文件中添加新的配置值，但又不想删除并重新生成整个配置。这也非常有用，如果用户不小心删除了一个配置值，但你不希望脚本因找不到配置值而中断（因为它会回退到你指定的默认值）。

## 另一种指定默认值的方式

Bukkit 配置系统还内置了一些函数，允许你在获取值时指定一个默认值。例如：

```python
import pyspigot as ps

config = ps.config.loadConfig('script_config.yml', config_defaults)

print(config.getString('key-1', 'Key 1 default'))
print(config.getString('key-2', 'Key 2 default'))
print(config.getString('key-3', 'Key 3 default'))
```

在上面的代码中，在 `getString` 函数中传递了默认值，只有在配置中不存在该值时才会返回这些默认值。获取其他类型的值时也有类似的函数，包括 `getInt`、`getDouble`、`getLong`、`getBoolean` 等。

# 特殊的 Python 数据类型

## 列表、集合和元组

列表、集合和元组都以 YAML 语法中的列表形式保存，看起来像这样：

```yaml
list:
  - 'item1'
  - 'item2'
  - 'item3'
```

## 字典

字典在 YAML 语法中以 `key: value` 的形式表示。例如，考虑以下代码：

```python
thisdict = {
  "brand": "Ford",
  "model": "Mustang",
  "year": 1964
}

script_config.set('dict', thisdict)
```

这将以以下格式输出到 YAML 语法中：

```yaml
dict:
  year: 1964
  model: Mustang
  brand: Ford
```

当然，你也可以在字典中嵌套列表、集合、元组和其他字典，并相应地保存它们。字典对于同时设置多个配置值特别有用，无需逐个设置每个值。

# 代码示例

让我们来看一个加载配置文件、从中读取一个数字和一个字符串、写入值并保存的示例代码。

```python
import pyspigot as ps

script_config = ps.config.loadConfig('test.yml')

a_number = script_config.getInt('test-number')
a_string = script_config.getString('test-string')

script_config.set('test-set', 1337)
script_config.save()
```

第 1 行，我们导入 PySpigot 并将其命名为 `ps` 以便使用配置管理器 (`config`)。

第 3 行，我们使用配置管理器加载配置。幸运的是，配置管理器很容易从 `pyspigot` 辅助模块中通过变量名 `config` 访问。`loadConfig` 函数接受一个代表要加载的配置文件名称的字符串。如果文件不存在，它会自动为你创建。

第 5 和 6 行，我们分别使用 `getInt` 和 `getString` 从配置中读取一个数字和一个字符串。

最后，在第 7 和 8 行，我们首先将值 1337 设置为名为 `test-set` 的配置键。然后，我们使用 `script_config.save()` 保存配置。

!> 配置文件不是每个脚本独有的！任何脚本都可以访问任何配置文件。请使用独特的名称。

## 总结: {docsify-ignore}

- 脚本可以加载和保存自动存储在 PySpigot 插件文件夹的 `configs` 文件夹中的配置文件。
- 要加载配置，使用 `config.load(filePath)`。`filePath` 参数是你希望加载的配置文件的名称 *或* 路径（包括 `.yml` 扩展名）。如果配置文件不存在，它将自动为你创建。这返回一个 `ScriptConfig` 对象，用于访问配置的内容和写入配置。
- 要获取已加载配置的所有可用函数/方法，请参阅 [Javadocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/configuration/MemorySection.html)。
- 要在配置中设置值，使用 `script_config.set(key, value)`，其中 `key` 是你要写入的键，`value` 是要写的值。
- 你可以通过向 `loadConfig` 函数传递包含默认配置值的 YAML 格式字符串来设置配置默认值。
- 最后，要保存配置，使用 `script_config.save()`。
