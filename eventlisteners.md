# 事件监听器

使用 PySpigot 的 ListenerManager，您有能力注册事件监听器。类似于 Bukkit 的监听器系统，当事件触发时，您的脚本中的相应函数会被调用，您可以在脚本中处理该事件。

请参阅[一般信息](writingscripts#pyspigot-的管理器)页面了解如何将监听器管理器导入到您的脚本中的说明。

这不是一个关于 Bukkit 事件的全面指南。要了解更完整的事件指南，请参阅 Spigot 关于使用事件 API 的教程[这里](https://www.spigotmc.org/wiki/using-the-event-api/)。

# 监听器管理器使用方法

如果您希望对事件有更大的控制权或需要更高级的事件处理，监听器管理器中有五个函数可供您在脚本中使用：

- `registerListener(function, event)`: 接受触发事件时要调用的函数以及要监听的事件。返回一个 `ScriptEventListener`，可用于稍后取消注册该事件。
- `registerListener(function, event, priority)`: 与上面相同，但还允许您定义事件优先级（相对于同一事件的其他监听器，您的事件监听器应该何时触发）。优先级是一个字符串，代表一个事件优先级。返回一个 `ScriptEventListener`，可用于稍后取消注册该事件。
    - 事件优先级与 Bukkit 的[EventPriority 类](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/EventPriority.html)中的优先级相同。
- `registerListener(function, event, ignoreCancelled)`: 允许您“忽略”已被其他事件监听器取消的事件。如果事件在其他地方被取消，则不会调用脚本中的监听器。返回一个 `ScriptEventListener`，可用于稍后取消注册该事件。
    - 这只适用于可以被[取消](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Cancellable.html)的事件。
- `registerListener(function, event, priority, ignoreCancelled)`: 允许您注册一个被取消时忽略并且具有优先级的事件（结合了前面两个函数的功能）。返回一个 `ScriptEventListener`，可用于稍后取消注册该事件。
- `unregisterListener(event_listener)`: 允许您从脚本中取消注册事件监听器。接受注册监听器时返回的事件监听器。

?> 当您的脚本停止或卸载时，**不需要**取消注册事件监听器。PySpigot 会为您处理这一点。

# 代码示例

让我们看看下面这段定义并注册事件监听器的代码：

```python
import pyspigot as ps
from org.bukkit.event.player import AsyncPlayerChatEvent

def player_chat(event):
    print('Player sent a chat! Their message was: ' + event.getMessage())

listener = ps.listener.registerListener(player_chat, AsyncPlayerChatEvent)
```

第 1 行，我们导入 PySpigot 作为 `ps` 以便使用监听器管理器 (`listener`)。

第 2 行，有一个适当的导入语句用于 Bukkit 的 `AsyncPlayerChatEvent`。您希望监听的所有事件**必须**被导入！

第 4 行，我们定义了一个名为 `player_chat` 的函数，该函数接受一个事件作为参数（在这种情况下是一个 AsyncPlayerChatEvent）。这是当发生 AsyncPlayerChatEvent 时将被调用的函数。第 5 行，我们在控制台上打印一条简单消息，包含聊天中发送的消息。

所有事件监听器都必须使用 PySpigot 的监听器管理器进行注册。在这个例子中，我们使用 `registerListener(function, event)` 函数。它接受两个参数：

- 第一个参数接受当事件触发时应调用的函数。
- 第二个参数接受要监听的事件。

`registerListener` 函数返回一个 `ScriptEventListener`，表示已注册的事件。这可用于稍后取消注册事件监听器。

因此，在第 7 行，我们调用监听器管理器来注册我们的事件，传递我们在第 5 行定义的函数 `player_chat` 和我们想要监听的事件 `AsyncPlayerChatEvent`，并将返回的值赋给 `listener`。

有关可用监听器及其提供的函数/方法的完整文档，请参阅[Spigot Javadocs](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/event/Event.html)。

## 总结：{docsify-ignore}

- 您希望使用的所有事件都应该使用 Python 的导入语法导入。
- 所有事件监听器都应定义为您的脚本中的函数，接受一个参数，即事件（参数名称可以是您喜欢的任何名称）。
- 所有事件监听器都必须使用 PySpigot 的监听器管理器通过 `listener.registerListener(function, event)` 进行注册。
- 在注册事件监听器时，所有 `registerListener` 函数都会返回一个 `ScriptEventListener`，可用于稍后取消注册监听器。
