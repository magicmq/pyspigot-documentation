# Defining Tasks

Through PySpigot, you can interact with Bukkit's task scheduler and schedule/run synchronous and asynchronous tasks. These allow you to run code on a thread other than the main thread as well as run code repeatedly at a fixed interval.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the task manager into your script.

This is not a comprehensive guide to scheduling tasks. For a more complete guide to tasks and scheduler programming, see Bukkit's tutorial on using the scheduler [here](https://bukkit.fandom.com/wiki/Scheduler_Programming). Note that much of this information will not be useful because it pertains to Java, but the background information is nevertheless helpful.

# Task Manager Usage

There are several functions in the task manager available for you to use in your script:

- `runTask(function, functionArgs)`: Run a synchronous task as soon as possible. Takes the function to call when the task runs. Also takes any number of arguments that should be passed to the function when the task runs.
- `runTaskAsync(function, functionArgs)`: Run an asychronous task (a task on a thread other than the main server thread). Takes the function to call when the task runs. Also takes any number of arguments that should be passed to the function when the task runs.
- `runTaskLater(function, delay, functionArgs)`: Run a synchronous task at some point in the future after the specified delay. Takes the function to call when the task runs and the delay to wait (in ticks) before running the task. Also takes any number of arguments that should be passed to the function when the task runs.
- `runTaskLaterAsync(function, delay, functionArgs)`: Run an asynchronous task at some point in the future after the specified delay. Takes the function to call when the task runs and the delay to wait (in ticks) before running the task. Also takes any number of arguments that should be passed to the function when the task runs.
- `scheduleRepeatingTask(function, delay, interval, functionArgs)`: Run a synchronous repeating task that repeats every specified interval. Takes the function to call each time the task runs, the delay to wait (in ticks) before running the task, and the interval (in ticks) at which the task should be run. Also takes any number of arguments that should be passed to the function when the task runs.
- `scheduleAsyncRepeatingTask(function, delay, interval, functionArgs)`: Run an asynchronous repeating task that repeats every specified interval. Takes the function to call each time the task runs, the delay to wait (in ticks) before running the task, and the interval (in ticks) at which the task should be run. Also takes any number of arguments that should be passed to the function when the task runs.
- `stopTask(id)`: Stop/Cancel a task. Takes the id of the task to stop.

?> In the above functions, `functionArgs` is an optional argument. If the function to call does not accept any arguments, you do not need to specify any.

20 ticks of in-game time is one real-world second (assuming there is no server tick lag).

# Code Example

Let's take a look at the following code that defines and starts a task:

```python
import pyspigot as ps

a_string = 'Test'

def run_task(arg):
    #Do something...

task_id = ps.scheduler.scheduleRepeatingTask(run_task, 0, 100, a_string)
```

On line 1, we import PySpigot as `ps` to utilize the task manager (`scheduler`).

On line 3, we define a string called `a_string`.

On line 5, we define a function called `run_task` that takes one argument.

Like listeners, all tasks must be registered and run with PySpigot's task manager. There are many different ways to start tasks depending on if we want the task to be synchronous, ascynchronous, and/or repeating, but here we want our task to be synchronous and repeating, so we use `scheduleRepeatingTask(function, delay, interval, functionArgs)`, which in this case takes four arguments:

- The first argument accepts the function that should be called when the task runs (either once or repeatedly at a fixed interval).
- The second argument is the delay (in ticks) that the scheduler should wait before starting the task when it is registered.
- The third argument is the interval (in ticks) that the task should be run.
- The final arguments (fourth only in this case) are used to specify the arguments that should be passed to the function.

Therefore, on line 8, we register our task as a synchronous repeating task with the task manager. This will return a task id, which we then store as a variable called `task_id`. We store this task id in case we want to cancel our task later. Cancelling a task requires the task ID.

## Passing Multiple Arguments to a Task's Function

If we wanted `run_task` in the above example to take two arguments instead of one, we would modify the code like so:

```python
import pyspigot as ps

a_string = 'Test'
another_string = 'Test 2'

def run_task(arg, arg2):
    #Do something...

task_id = ps.scheduler.scheduleRepeatingTask(run_task, 0, 100, a_string, another_string)
```

This example is very similar to the first, except that we define another string (`another_string`) and pass it (as well as `a_string`) to the task manager when we register the task on line 8. These two strings are then passed, in order, to `run_task` when it is called by the task.

As you can see, all arguments to be passed are added at the end of the function that registers the task, in the appropriate order that they should be passed.

## To summarize: {docsify-ignore}

- Like listeners, tasks are defined as functions in your script. Task can take any number of arguments, including zero.
- Tasks can pass arguments on to the function they call. Specify these arguments when you register your task with the task manager.
- All tasks must be registered with PySpigot's task manager. For example, to schedule and run a synchronous repeating task, use `scheduler.scheduleRepeatingTask(function, delay, interval, functionArgs)`.