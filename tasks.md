# 定义任务

通过 PySpigot，你可以与 Bukkit 的任务调度器进行交互并安排/运行同步和异步任务。这些功能允许你在主线程之外的线程上运行代码，以及按照固定间隔重复运行代码。

请参阅 [一般信息](writingscripts#pyspigot-的管理器) 页面以获取如何将任务管理器导入到你的脚本中的说明。

这不是一个全面的任务调度指南。对于更完整的任务和调度编程指南，请参阅 Bukkit 关于使用调度器的教程 [这里](https://bukkit.fandom.com/wiki/Scheduler_Programming)。请注意，这些信息大部分并不适用，因为它们与 Java 相关，但是背景信息仍然很有帮助。

# 任务管理器使用

任务管理器中有几个函数可供你在脚本中使用：

- `runTask(function, functionArgs)`: 尽快运行一个同步任务。接受当任务运行时要调用的函数。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `runTaskAsync(function, functionArgs)`: 运行一个异步任务（在主线程之外的线程上运行）。接受当任务运行时要调用的函数。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `runTaskLater(function, delay, functionArgs)`: 在指定延迟之后的某个时刻运行一个同步任务。接受当任务运行时要调用的函数和等待运行任务前的延迟（以刻为单位）。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `runTaskLaterAsync(function, delay, functionArgs)`: 在指定延迟之后的某个时刻运行一个异步任务。接受当任务运行时要调用的函数和等待运行任务前的延迟（以刻为单位）。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `scheduleRepeatingTask(function, delay, interval, functionArgs)`: 按照指定的间隔重复运行一个同步任务。接受每次任务运行时要调用的函数、运行任务前的延迟（以刻为单位）以及任务应运行的间隔（以刻为单位）。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `scheduleAsyncRepeatingTask(function, delay, interval, functionArgs)`: 按照指定的间隔重复运行一个异步任务。接受每次任务运行时要调用的函数、运行任务前的延迟（以刻为单位）以及任务应运行的间隔（以刻为单位）。还可以接受任何数量的参数，在任务运行时传递给该函数。
- `runSyncCallbackTask(function, callback, functionArgs)`: 安排一个带有同步回调的异步任务。接受用于异步部分的函数和用于同步部分的另一个函数。还可以接受任何数量的参数，在任务运行时传递给该函数（异步部分）。
- `runSyncCallbackTaskLater(function, callback, delay, functionArgs)`: 在指定延迟之后安排一个带有同步回调的异步任务。接受用于异步部分的函数、用于同步部分的另一个函数以及等待运行任务前的延迟（以刻为单位）。还可以接受任何数量的参数，在任务运行时传递给该函数（异步部分）。
- `stopTask(id)`: 停止/取消一个任务。接受要停止的任务的 ID。

每当安排一个任务时，都会返回一个任务 ID（整数）。如果需要的话，可以使用这个 ID 来取消任务。

在上述函数中，`functionArgs` 是一个可选参数。如果要调用的函数不接受任何参数，你不需要指定任何参数。

?> 游戏中的 20 刻等于现实世界的一秒（假设没有服务器刻的延迟）。因此，一刻等于 1/20 秒。

# 代码示例

让我们看看以下定义并启动任务的代码：

```python
import pyspigot as ps

a_string = 'Test'

def run_task(arg):
    #Do something...

task_id = ps.scheduler.scheduleRepeatingTask(run_task, 0, 100, a_string)
```

第 1 行，我们导入 PySpigot 作为 `ps` 以便使用任务管理器 (`scheduler`)。

第 3 行，我们定义一个字符串 `a_string`。

第 5 行，我们定义一个名为 `run_task` 的函数，它接受一个参数。

像监听器一样，所有任务都必须通过 PySpigot 的任务管理器注册和运行。根据我们想要任务是同步的、异步的和/或是重复的，启动任务的方式有很多种，但在这里我们希望我们的任务是同步的并且重复的，所以我们使用 `scheduleRepeatingTask(function, delay, interval, functionArgs)`，这需要四个参数：

- 第一个参数接受当任务运行时（一次或定期）应调用的函数。
- 第二个参数是注册任务时调度器应在开始任务前等待的延迟（以刻为单位）。
- 第三个参数是任务应运行的间隔（以刻为单位）。
- 最后的参数（在这个例子中只有第四个）用于指定应传递给函数的参数。

因此，在第 8 行，我们使用任务管理器将我们的任务注册为一个同步重复任务。这将返回一个任务 ID，我们将其存储为一个名为 `task_id` 的变量。我们存储这个任务 ID 以防我们以后想要取消我们的任务。取消一个任务需要任务 ID。

## 向任务函数传递多个参数

如果我们希望上面的例子中的 `run_task` 接受两个参数而不是一个，我们可以这样修改代码：

```python
import pyspigot as ps

a_string = 'Test'
another_string = 'Test 2'

def run_task(arg, arg2):
    #Do something...

task_id = ps.scheduler.scheduleRepeatingTask(run_task, 0, 100, a_string, another_string)
```

这个例子与第一个非常相似，只是我们定义了另一个字符串 `another_string` 并在第 8 行注册任务时将其（以及 `a_string`）传递给任务管理器。这两个字符串在 `run_task` 被任务调用时按顺序传递给它。

正如你所见，所有要传递的参数都在注册任务的函数末尾添加，按适当的顺序排列。

## 回调任务

任务管理器还包括一个回调任务，这是一种特殊的任务，它异步运行，然后在异步任务完成后运行另一个同步任务。这种任务类型在你想异步执行工作，然后在同步上下文中处理工作的场景中非常有用。例如，考虑服务器有一个存储玩家数据的 SQL 数据库的情况。当玩家加入时，应该异步地从数据库获取数据以避免服务器延迟，但它不能异步地应用于玩家，因为所有与 Bukkit 和服务器的交互都应该同步进行。因此，在这种情况下，同步回调是有用的，它可以将从数据库获得的数据带回主服务器线程进行进一步处理，并将其应用于玩家。

### 代码示例

以下是使用回调任务的一个简单示例：

```python
import pyspigot as ps

def async_task():
    print('Asynchronous!')
    data = 'some data'
    return data

def sync_task(data):
    print('Synchronous!')
    print(data)

ps.scheduler.runSyncCallbackTask(async_task, sync_task)
```

观察到的控制台输出如下：

```
[STDOUT] Asynchronous!
[STDOUT] Synchronous!
[STDOUT] some data
```

这个例子有几个需要注意的地方：

- 首先，为回调任务的异步和同步部分分别定义了不同的函数。
- 其次，异步任务首先发生，而同步任务直到异步任务完成后才开始执行。
- 第三，从异步部分的任务返回的任何数据（例如上面示例中的第 6 行的返回语句）作为函数参数传递给同步部分的任务（`sync_task` 上面接受参数 `data`）。这使得可以在同步上下文中处理异步部分的任务检索到的任何数据。

## 总结：

- 像监听器一样，任务被定义为脚本中的函数。任务可以接受任意数量的参数，包括零。
- 任务可以将参数传递给它们调用的函数。在你使用任务管理器注册任务时指定这些参数。
- 所有任务都必须通过 PySpigot 的任务管理器注册。例如，为了安排和运行一个同步重复任务，使用 `scheduler.scheduleRepeatingTask(function, delay, interval, functionArgs)`。
