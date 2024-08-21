# 脚本选项

在 PySpigot 的主插件文件夹中，你会找到一个名为 `script_options.yml` 的文件。这个文件使你能够为脚本指定各种特定于脚本的选项，所有这些选项都在此文档中涵盖。

!> 为每个脚本定义脚本选项是 *可选的*；更多详情请参见[默认值](#默认值)部分。

## 基本格式

如果你想为脚本指定选项，可以这样做：

``` yaml
test.py:
  enabled: true
```

`script_options.yml` 文件中的每一节都应该是脚本的名称。名称必须 *完全* 匹配脚本的名称（包括 `.py`），否则这些选项不会被解析。这一节中的所有选项都适用于该节命名的脚本。在上面的例子中，选项 `enabled` 适用于脚本 `test.py`。

如果你想为多个脚本定义选项，可以这样做：

``` yaml
test.py:
  enabled: true
  file-logging-enabled: true
test2.py:
  enabled: true
  depend: ['test.py']
```

如你所见，在上面的代码片段中，我们定义了一个新的节名为 `test2.py`，这一节中的所有选项都将适用于脚本 `test2.py`。`test.py` 脚本中的所有选项都适用于脚本 `test.py`。

## 默认值

为每个脚本指定脚本选项并不是必需的。此外，如果你不希望设置某些脚本选项，则可以省略它们。所有选项都有默认值，这些默认值可以在 PySpigot 的 `config.yml` 文件中，在 `script-option-defaults` 部分进行自定义。

如果脚本在 `script_options.yml` 中没有任何脚本选项定义，那么 PySpigot 将回退到 `config.yml` 中定义的任何默认值。如果 `config.yml` 中没有定义默认值，则使用内部回退值。每个选项的这些内部回退值在下面列出。

## 选项

### `enabled`

用于启用或禁用脚本。要禁用脚本，请将此值设置为 `false`。

默认值：`true`（启用）
用法：

``` yaml
test.py:
  enabled: true
```

### `load-priority`

用于为脚本指定整数加载优先级。脚本按照从高到低的加载优先级顺序加载。换句话说，具有较高加载优先级的脚本较早加载，而具有较低加载优先级的脚本较晚加载。如果有多个脚本具有相同的加载优先级，则按字母顺序加载。要列出依赖项，请使用脚本的全名（包括 `.py`）。

默认值：1
用法：

``` yaml
test.py:
  depend: ['test1.py', 'test2.py']
```

### `depend`

用于指定脚本依赖的其他脚本列表。脚本将按照依赖关系的顺序加载。如果依赖的脚本未加载，则当前脚本也不会加载。

?> 如果你在脚本中使用了 ProtocolLib 或 PlaceholderAPI，**不需要**在这里指定它们。PySpigot 内置支持这两个插件，并且依赖管理是在内部处理的。

?> 此值是一个列表（即使只有一个项目！），并且应该格式化为 `depend: ['item1', 'item2', 'item3', ...]`

默认值：无
用法：

``` yaml
test.py:
  plugin-depend: ['test1.py', 'test2.py']
```

### `plugin-depend`

用于指定脚本需要加载的一系列插件。如果这些插件依赖项没有在服务器上加载并运行，脚本将不会加载。此外，当插件卸载/禁用时，任何依赖于该插件的脚本（如在此选项下指定）将自动卸载（如果 `config.yml` 中的 `script-unload-on-plugin-disable` 选项设置为 `true`）。

?> 如果你在脚本中使用了 ProtocolLib 或 PlaceholderAPI，**不需要**在这里指定它们。PySpigot 内置支持这两个插件，并且依赖管理是在内部处理的。

?> 此值是一个列表（即使只有一个项目！），并且应该格式化为 `plugin-depend: ['item1', 'item2', 'item3', ...]`

默认值：无
用法：

``` yaml
test.py:
  plugin-depend: ['Essentials', 'WorldGuard']
```

### `file-logging-enabled`

指定是否为脚本启用文件日志记录。如果此选项设置为 `true`，则会为脚本生成一个日志文件，并且任何错误消息（以及发送到脚本日志记录器的打印消息）都将记录到此文件中。如果此选项设置为 `false`，则不会将任何消息记录到文件中，但是消息仍然会打印到服务器控制台。

默认值：PySpigot 的 `config.yml` 中的 `log-to-file` 值，默认为 `true`
用法：

``` yaml
test.py:
  file-logging-enabled: true
```

### `min-logging-level`

指定应记录到脚本的日志文件和控制台的最小日志级别。选项可以在 [Javadocs](https://docs.oracle.com/javase/8/docs/api/java/util/logging/Level.html) 中找到。

默认值：PySpigot 的 `config.yml` 中的 `min-log-level` 值，默认为 `INFO`
用法：

``` yaml
test.py:
  min-log-level: 'INFO'
```

### `permissions`

指定脚本使用的权限列表。这对于想要限制访问某些功能的脚本很有用。此部分的定义方式与在 Bukkit 插件的 `plugin.yml` 文件中定义权限的方式 [完全相同](https://docs.papermc.io/paper/dev/plugin-yml#permissions)。请参阅下面的用法代码示例了解如何定义权限、默认值和子权限。

默认值：未定义权限（空列表）
用法：

```yaml
test.py:
  permissions:
    permission.node.*:
      description: 'This is a permission node'
      default: op
      children:
        permission.node.child: true
    permission.node.child:
      description: 'This is a child permission node'
      default: true
    another.permission.node:
      description: 'This is another permission node'
      default: not op
```

- `description` 是权限节点的描述，这将在权限列表中显示。默认值是权限节点的名称。
- `default` 是权限节点的默认值，或者换句话说，谁应该默认拥有权限节点。`default` 的可能值有四个：
  - `op`：只有服务器管理员默认拥有权限节点。
  - `not op`：非管理员玩家默认拥有权限节点。
  - `true`：所有玩家都有此权限（即，它是默认权限）。
  - `false`：没有玩家拥有此权限（即，它 *不是* 默认权限）。默认值是 `default_permission`（如下所述）的值。
- `children` 是应该继承父权限的一系列子权限。每个权限节点都可以有子权限。当设置为 `true` 时，子权限将继承父权限。

### `permission-default`

指定权限如果没有定义 `default` 值时应具有的默认值。

默认值：`op`
用法：

```yaml
test.py:
  permission-default: true
```

`permission-default` 的允许值为 `op`、`not op`、`true` 和 `false`：

- `op`：只有服务器管理员默认拥有权限节点。
- `not op`：非管理员玩家默认拥有权限节点。
- `true`：所有玩家都有此权限（即，它是默认权限）。
- `false`：没有玩家拥有此权限（即，它 *不是* 默认权限）。
