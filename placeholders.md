# 占位符管理器

!> 占位符管理器是一个 *可选* 的管理器。只有当 PySpigot 插件启用时服务器上安装了 PlaceholderAPI 插件，才应访问此管理器。

如果你想在脚本中注册占位符扩展，PySpigot 包含了一个与 PlaceholderAPI 交互的管理器。

请参阅 [一般信息](writingscripts#PySpigot-的管理器) 页面上的说明来了解如何将占位符管理器导入到你的脚本中。

# 占位符管理器使用方法

所有由脚本创建的占位符都会遵循这样的通用格式：`%script:<scriptname>_<placeholder>%`，其中 `<scriptname>` 是你的脚本名称（不含 .py 扩展名），而 `<placeholder>` 是具体的占位符，你需要在占位符的“替换”函数中自行处理。详情请参见下面的代码示例。

占位符管理器提供了三个函数用于注册和注销占位符：

- `registerPlaceholder(placeholder_function)`: 使用默认作者 ("Script Author") 和版本 ("1.0.0") 注册一个新的占位符扩展。当使用占位符时，会调用 `placeholder_function`。它应该返回用来替换占位符的文本。返回一个 `ScriptPlaceholder` 对象，表示已注册的占位符。
- `registerPlaceholder(placeholder_function, author, version)`: 使用自定义作者和版本注册一个新的占位符扩展。返回一个 `ScriptPlaceholder` 对象，表示已注册的占位符。
- `unregisterPlaceholder(placeholder)`: 注销先前注册的占位符。需要一个之前由 `registerPlaceholder` 函数返回的 `ScriptPlaceholder` 对象。

?> 当你的脚本停止或卸载时，**无需**注销你的占位符。PySpigot 会为你处理这些事情。

!> 脚本一次只能注册一个占位符扩展。关于如何为一个脚本定义多个占位符，请参见下面的代码示例。

# 代码示例

让我们来看一段定义并注册占位符的代码：

```python
import pyspigot as ps

def replace(offline_player, placeholder):
	if placeholder == 'placeholder1':
		return "Replace placeholder 1!"
	elif placeholder == 'placeholder2':
		return "Replace placeholder 2!"

placeholder = ps.placeholder.registerPlaceholder(replace)
```

在第 1 行，我们导入 PySpigot 并将其别名为 `ps`，以便使用占位符管理器 (`placeholder`)。

在第 3 行，我们定义了一个 `replace` 函数，该函数会在使用占位符时被调用。这个函数有两个参数：`offline_player` 是一个代表与占位符关联的玩家的 Bukkit API 的 `OfflinePlayer` 对象；`placeholder` 是具体占位符的文本。

!> 如果占位符与玩家无关，则 `offline_player` 可能为 `None`。

在第 4 行，我们检查 `placeholder` 是否等于我们想要定义的具体占位符 `placeholder1`。然后，在第 5 行，我们返回被替换的文本。

在第 6 行，我们使用 `elif` 定义另一个具体的占位符 `placeholder2`。我们在第 6 行返回该占位符的文本。

如你所见，你无需为每个具体占位符单独注册新的占位符扩展。相反，你只需要检查替换函数中的 `placeholder` 参数是否等于你想定义的占位符即可。可以使用 `if` 和 `elif` 语句来实现这一点。

如果我们把这个脚本命名为 `test.py`，上面代码中定义的两个占位符分别是 `%script:test_placeholder1%` 和 `%script:test_placeholder2%`。

## 总结：{docsify-ignore}

- 由脚本定义的占位符遵循格式 `%script:<scriptname>_<placeholder%>`。
- 你的占位符替换函数应该接受两个参数：`offline_player` 和 `placeholder`。你可以给它们命名任何你喜欢的名字。当占位符被使用时，会调用这个函数。`offline_player` 是与占位符相关的玩家（如果存在的话）。`placeholder` 是使用的具体占位符。
- 使用 PySpigot 的占位符管理器通过 `registerPlaceholder(placeholder_function)` 或 `registerPlaceholder(placeholder_function, author, version)` 注册你的占位符。
- 每个脚本只需要注册一个占位符扩展。在替换函数中检查 `placeholder` 参数来处理具体占位符。
- 在注册占位符扩展时，注册函数都会返回一个 `ScriptPlaceholder` 对象，稍后可以使用它来注销占位符扩展。
