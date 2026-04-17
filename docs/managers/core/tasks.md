# Defining Tasks

Through PySpigot, scripts can schedule and run tasks using the platform's task scheduler. Tasks allow you to run code at a later time, at a fixed interval, and/or on a thread other than the main server thread. The task manager also has built-in support for asynchronous tasks with synchronous callbacks (on Bukkit).

For instructions on importing the task manager into your script, visit the [General Information](../usage.md) page.

???+ info

    This is not a comprehensive guide to scheduling tasks. For a more complete guide to scheduler programming, see Bukkit's tutorial on using the scheduler [here](https://bukkit.fandom.com/wiki/Scheduler_Programming). Note that much of this information pertains to writing plugins in Java, but the general ideas are nevertheless helpful to understand.

???+ note

    20 ticks of in-game time is one real-world second (in a server without TPS lag). Therefore, one tick is equal to 1/20 of a second.

    On BungeeCord and Velocity, ticks do not apply. Delays and intervals on these platforms can optionally be specified using a Java [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html).

## Task Decorators

PySpigot ships with a `decorators/task.py` helper module that provides Python **decorators** for scheduling tasks. Using the decorators is the recommended way to schedule tasks, as it is cleaner and more Pythonic than calling the task manager directly.

### Importing

=== "Bukkit"

    ``` py
    from decorators.task import task           # synchronous task
    from decorators.task import async_task     # asynchronous task
    from decorators.task import sync_callback_task  # (1)!
    ```

    1. Only needed for asynchronous tasks with a synchronous callback. See [`@sync_callback_task`](#sync_callback_task-bukkit-only) below.

=== "Velocity"

    ``` py
    from decorators.task import async_task
    ```

=== "BungeeCord"

    ``` py
    from decorators.task import async_task
    ```

### Scheduling Rules

The `@task` and `@async_task` decorators follow a simple set of rules to determine how the task is scheduled, based on the `delay` and `interval` parameters:

| `delay` | `interval` | Behavior |
|---|---|---|
| `0` | `0` | Runs immediately (once). |
| `> 0` | `0` | Runs once after the specified delay. |
| `0` | `> 0` | Repeats at the specified interval, starting immediately. |
| `> 0` | `> 0` | Repeats at the specified interval, starting after the initial delay. |

### `@task` (Bukkit only)

The `@task` decorator schedules a **synchronous** task. Synchronous tasks run on the main server thread.

``` py linenums="1"
from decorators.task import task

@task() # (1)!
def my_task():
    print('Running immediately on the main thread!')
```

1. `@task()` with no arguments runs the task once, immediately.

Delayed and repeating examples:

``` py linenums="1"
from decorators.task import task

@task(delay=40) # (1)!
def delayed_task():
    print('Runs after 2 seconds.')

@task(interval=20) # (2)!
def repeating_task():
    print('Runs every second.')

@task(delay=20, interval=20) # (3)!
def delayed_repeating_task():
    print('Runs every second, starting after 1 second.')
```

1. Runs once, 40 ticks (2 seconds) after the script loads.
2. Runs every 20 ticks (1 second), starting immediately.
3. Runs every 20 ticks (1 second), starting after an initial 20-tick delay.

### `@async_task`

The `@async_task` decorator schedules an **asynchronous** task. Asynchronous tasks run on a thread other than the main server thread.

???+ warning

    On Bukkit, never interact with the Bukkit API from an asynchronous task. Bukkit API calls must happen on the main server thread. Use a [sync callback task](#sync_callback_task-bukkit-only) if you need to bring asynchronous results back to the main thread.

=== "Bukkit"

    The Bukkit `@async_task` decorator accepts `delay` and `interval` (in ticks) only:

    ``` py linenums="1"
    from decorators.task import async_task

    @async_task() # (1)!
    def my_async_task():
        print('Running asynchronously!')

    @async_task(delay=40) # (2)!
    def delayed_async_task():
        print('Runs asynchronously after 2 seconds.')

    @async_task(interval=20) # (3)!
    def repeating_async_task():
        print('Runs asynchronously every second.')
    ```

    1. Runs once, immediately, on a separate thread.
    2. Runs once asynchronously after a 40-tick delay.
    3. Repeats asynchronously every 20 ticks.

=== "Velocity"

    On Velocity, `@async_task` supports optional `TimeUnit` parameters for the delay and interval:

    - `delay` — how long to wait before first execution. Defaults to `0`.
    - `delay_time_unit` — the [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html) for `delay`. If omitted, ticks are used.
    - `interval` — how long to wait between executions. Defaults to `0`.
    - `interval_time_unit` — the [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html) for `interval`. If omitted, ticks are used.
    - `time_unit` — a convenience parameter that sets both `delay_time_unit` and `interval_time_unit` at once.

    ``` py linenums="1"
    from decorators.task import async_task
    from java.util.concurrent import TimeUnit

    @async_task(delay=5, time_unit=TimeUnit.SECONDS) # (1)!
    def delayed_async_task():
        print('Runs asynchronously after 5 seconds.')

    @async_task(delay=2, delay_time_unit=TimeUnit.SECONDS, interval=30, interval_time_unit=TimeUnit.SECONDS) # (2)!
    def repeating_async_task():
        print('Repeats every 30 seconds, starting after 2 seconds.')
    ```

    1. Runs once, 5 seconds after the script loads. `time_unit` sets both delay and interval time units in one step.
    2. Repeats every 30 seconds, starting after an initial 2-second delay. Delay and interval have independent `TimeUnit` values.

=== "BungeeCord"

    On BungeeCord, `@async_task` supports an optional `time_unit` parameter that applies to both `delay` and `interval`:

    - `delay` — how long to wait before first execution. Defaults to `0`.
    - `interval` — how long to wait between executions. Defaults to `0`.
    - `time_unit` — the [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html) for both `delay` and `interval`. If omitted, ticks are used.

    ``` py linenums="1"
    from decorators.task import async_task
    from java.util.concurrent import TimeUnit

    @async_task(delay=5, time_unit=TimeUnit.SECONDS) # (1)!
    def delayed_async_task():
        print('Runs asynchronously after 5 seconds.')

    @async_task(delay=2, interval=30, time_unit=TimeUnit.SECONDS) # (2)!
    def repeating_async_task():
        print('Repeats every 30 seconds, starting after 2 seconds.')
    ```

    1. Runs once, 5 seconds after the script loads.
    2. Repeats every 30 seconds, starting after an initial 2-second delay. The same `time_unit` applies to both.

### `@sync_callback_task` (Bukkit only)

The `@sync_callback_task` decorator schedules an **asynchronous** task with a **synchronous callback**. The async portion runs first on a background thread; when it finishes, the callback runs synchronously on the main server thread. Any value *returned* from the async function is automatically passed as an argument to the callback.

This is useful when you need to perform I/O work (such as a database query) asynchronously to avoid server lag, and then apply the result back on the main thread.

Attach the callback by applying the decorated function's `.callback` attribute as a decorator to the synchronous callback function:

``` py linenums="1"
from decorators.task import sync_callback_task

@sync_callback_task() # (1)!
def fetch_data():
    print('Asynchronous!')
    data = 'some data'
    return data # (2)!

@fetch_data.callback # (3)!
def apply_data(data): # (4)!
    print('Synchronous!')
    print(data)
```

1. `@sync_callback_task()` schedules `fetch_data` to run asynchronously. An optional `delay` (in ticks) parameter can be passed to delay the start.

2. Any value returned from the async function is automatically forwarded as an argument to the callback.

3. `@fetch_data.callback` registers `apply_data` as the synchronous callback. It is called after `fetch_data` completes.

4. `apply_data` receives the return value of `fetch_data` as its argument.

The following console output is observed when this code runs:

```
[STDOUT] Asynchronous!
[STDOUT] Synchronous!
[STDOUT] some data
```

### Passing Arguments to the Task Function

Extra positional arguments passed to any decorator are forwarded to the task function each time it executes:

=== "Bukkit"

    ``` py linenums="1"
    from decorators.task import task

    message = 'Hello!'
    count = 42

    @task(0, 20, message, count) # (1)!
    def my_task(msg, n):
        print(msg, n)
    ```

    1. The third and subsequent arguments (`message`, `count`) are passed through to `my_task` on every execution.

    ```
    [STDOUT] Hello! 42
    ```

=== "Velocity"

    ``` py linenums="1"
    from decorators.task import async_task

    message = 'Hello!'

    @async_task(0, None, 20, None, None, message) # (1)!
    def my_task(msg):
        print(msg)
    ```

    1. On Velocity, the signature is `async_task(delay, delay_time_unit, interval, interval_time_unit, time_unit, *args)`. Extra positional arguments follow after the time unit parameters.

    ```
    [STDOUT] Hello!
    ```

=== "BungeeCord"

    ``` py linenums="1"
    from decorators.task import async_task

    message = 'Hello!'

    @async_task(0, 20, None, message) # (1)!
    def my_task(msg):
        print(msg)
    ```

    1. On BungeeCord, the signature is `async_task(delay, interval, time_unit, *args)`. Extra positional arguments follow after `time_unit`.

    ```
    [STDOUT] Hello!
    ```

### Cancelling a Task

When a function is decorated with any task decorator, two attributes are attached to it:

- `.scheduled_task` — the `Task` object representing the scheduled task.
- `.cancel()` — a convenience method that cancels the task.

``` py linenums="1"
my_task.cancel() # (1)!
```

1. Cancels `my_task`. After this call, the task will no longer execute.

???+ note

    When a script is stopped, any tasks belonging to that script are stopped/cancelled automatically. You do not need to cancel them yourself.

## Task Manager Usage

If you prefer to schedule tasks manually without using the decorators, the task manager functions are available as an alternative:

=== "Bukkit"

    - `runTask(function, functionArgs)`: Run a synchronous task as soon as possible.
    - `runTaskAsync(function, functionArgs)`: Run an asynchronous task as soon as possible.
    - `runTaskLater(function, delay, functionArgs)`: Run a synchronous task after the specified delay (in ticks).
    - `runTaskLaterAsync(function, delay, functionArgs)`: Run an asynchronous task after the specified delay (in ticks).
    - `scheduleRepeatingTask(function, delay, interval, functionArgs)`: Run a synchronous repeating task.
    - `scheduleAsyncRepeatingTask(function, delay, interval, functionArgs)`: Run an asynchronous repeating task.
    - `runSyncCallbackTask(function, callback, functionArgs)`: Schedule an asynchronous task with a synchronous callback.
    - `runSyncCallbackTaskLater(function, callback, delay, functionArgs)`: Schedule an asynchronous task with a synchronous callback, starting after a delay.
    - `stopTask(task)`: Stop/cancel a task. Takes the `Task` object returned when the task was scheduled.

    In the above functions, `functionArgs` is optional — if the function takes no arguments, it can be omitted.

=== "Velocity"

    Velocity does not support synchronous tasks. Only asynchronous scheduling functions are available:

    - `runTaskAsync(function, functionArgs)`: Run an asynchronous task as soon as possible.
    - `runTaskLaterAsync(function, delay, functionArgs)`: Run an asynchronous task after the specified delay (in ticks).
    - `runTaskLaterAsync(function, delay, timeUnit, functionArgs)`: Run an asynchronous task after the specified delay, using a [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html).
    - `scheduleAsyncRepeatingTask(function, delay, interval, functionArgs)`: Run an asynchronous repeating task (delay and interval in ticks).
    - `scheduleAsyncRepeatingTask(function, delay, delayTimeUnit, interval, intervalTimeUnit, functionArgs)`: Run an asynchronous repeating task with independent time units for delay and interval.
    - `stopTask(task)`: Stop/cancel a task. Takes the `Task` object returned when the task was scheduled.

=== "BungeeCord"

    BungeeCord does not support synchronous tasks. Only asynchronous scheduling functions are available:

    - `runTaskAsync(function, functionArgs)`: Run an asynchronous task as soon as possible.
    - `runTaskLaterAsync(function, delay, functionArgs)`: Run an asynchronous task after the specified delay (in ticks).
    - `runTaskLaterAsync(function, delay, timeUnit, functionArgs)`: Run an asynchronous task after the specified delay, using a [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html).
    - `scheduleAsyncRepeatingTask(function, delay, interval, functionArgs)`: Run an asynchronous repeating task (delay and interval in ticks).
    - `scheduleAsyncRepeatingTask(function, delay, interval, timeUnit, functionArgs)`: Run an asynchronous repeating task with a shared [`TimeUnit`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/TimeUnit.html) for both delay and interval.
    - `stopTask(task)`: Stop/cancel a task. Takes the `Task` object returned when the task was scheduled.

Any time a task is scheduled using the task manager functions, a `Task` object is returned. This can be used to cancel the task later either by calling `task.cancel()` on the `Task` object, or by passing it to `stopTask(task)`.

### Code Example

``` py linenums="1"
import pyspigot as ps

a_string = 'Test'

def run_task(arg): # (1)!
    print(arg)

task = ps.task_manager().scheduleRepeatingTask(run_task, 0, 100, a_string) # (2)!

# Some time passes...

task.cancel() # (3)!
```

1. Define the task function. It accepts one argument, which will be the value of `a_string`.

2. Schedule a synchronous repeating task with a 0-tick initial delay and 100-tick interval. The `a_string` variable is passed through as an argument to `run_task` each time it executes.

3. Cancel the task by calling `cancel()` on the returned `Task` object. Alternatively, `ps.task_manager().stopTask(task)` has the same effect.

## Summary

- Tasks are defined as functions in your script. Task functions can take any number of arguments, including zero.
- The recommended way to schedule tasks is via the `@task`, `@async_task`, or `@sync_callback_task` decorators from `decorators/task.py`.
- Decorator scheduling rules: no delay/interval → runs once immediately; `delay > 0` → delayed single run; `interval > 0` → repeating task.
- On BungeeCord and Velocity, only asynchronous tasks are supported. Delays and intervals can be specified using a Java `TimeUnit` instead of ticks.
- Decorated task functions gain a `.scheduled_task` attribute (the `Task` object) and a `.cancel()` method.
- Alternatively, tasks can be scheduled manually via the task manager functions such as `scheduleRepeatingTask(function, delay, interval)`.
- When a script stops, all of its tasks are cancelled automatically.
