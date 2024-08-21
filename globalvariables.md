# 全局变量

PySpigot 包含了一个全局变量系统，该系统在运行时可供脚本访问。在 Java 端，这个系统依赖于 `HashMap`，这是一种存储键值对数据的数据结构，类似于 Python 中的字典。这个系统的目的是允许脚本间共享变量。如果你正在编写一组需要从其他脚本获取信息才能正确运行的脚本，这个功能可能会很有用。

插入到这个全局集合中的变量的变化会自动对所有脚本可见。如果变量的值发生了变化，无需再次将其插入到全局变量集合中。

# 访问全局变量系统

PySpigot 的全局变量系统就像 PySpigot 的其他管理器一样进行访问。有三种访问方式：

## 通过 pyspigot 辅助模块

```python
import pyspigot as ps

global_vars = ps.global_variables()

global_vars.<function>
```

pyspigot 辅助模块还定义了一些常用的别名以方便使用。`ps.global_vars` 和 `ps.gv` 都可以使用。例如：

```python
import pyspigot as ps

ps.global_vars.<function>
ps.gv.<function>
```

## 通过 PySpigot 类

```python
from dev.magicmq.pyspigot import PySpigot as ps

global_vars = ps.global_vars

global_vars.<function>
```

## 通过直接导入

```python
from dev.magicmq.pyspigot.manager.script import GlobalVariables as global_vars

global_vars.get().<function>
```

**注意**：如果你直接导入，必须调用 `get()`！

# 使用方法

全局变量系统的基本使用涉及调用 `set`、`get` 和 `remove` 函数。下面将详细介绍这些函数。这里有一个简短的例子：

脚本 A:

```python
import pyspigot as ps

global_vars = ps.global_variables()

test = 'Global variable'

global_vars.set('test', test)
```

脚本 B:

```python
import pyspigot as ps

global_vars = ps.global_variables()

test = global_vars.get('test')

print(test)

global_vars.remove('test')
```

在上面的代码中，我们在脚本 A 中将变量 `test` 设置到全局变量系统中。然后，我们从脚本 B 中检索并打印 `test` 的值，并在检索后移除该变量。

**提示**：一旦全局变量被设置到全局变量系统中，如果其值发生变化，会自动更新，所有脚本都能看到这个变化。如果变量的值发生变化，无需再次将其设置到系统中。

**注意**：名称是唯一的。如果使用与现有值相同的名称向全局值集合中插入新值，则旧值将被覆盖并最终丢失。

## 避免内存泄漏

全局变量系统并不智能。它会无限期地存储变量，除非它们被覆盖或有意清除。系统不会自动删除变量，除非明确被告知这样做。考虑以下例子：脚本 A 将一个变量设置到全局变量系统中供脚本 B 使用。脚本 B 检索该变量并使用它。一段时间后，脚本 B 完成任务并关闭。但是，脚本 B 使用的变量仍然存在于全局变量系统中。在这种情况下，该变量是“泄漏”的：它仍然存在并且可以读取，但不再被使用。在编程中，这种情况称为 [内存泄漏](https://en.wikipedia.org/wiki/Memory_leak)，这是不受欢迎的，因为它会导致不必要的内存使用。这也是可以避免的。

回到上面的例子：因为脚本 B 是使用数据的脚本，脚本 A 无法知道脚本 B 何时已经看到了该数据。因此，当脚本 B 完成检索后，有责任从全局变量系统中移除该变量。你可以看到在上面的代码中，一旦 `test` 变量被打印出来，就会被移除。

一般来说，当你使用全局变量系统时，应确保在代码的某个地方，不再需要的旧变量会被移除。

## 高级使用

全局变量系统包含一个名为 `getHashMap()` 的函数，该函数返回存储变量的底层 Java HashMap。你可以使用此函数获取底层的哈希表，并访问更高级的功能。有关 Java HashMap 类中可用函数的完整列表，请参阅 [JavaDocs for HashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html)。

# 可用函数

以下是全局变量系统中可用的函数列表：

`set(key, value)`:
- 使用给定的键（名称）将新值插入全局变量集合。总是会覆盖具有相同名称的现有变量。
- 参数：
    - key: 要设置的全局变量的名称
    - value: 要设置的变量的值
- 返回：先前使用给定名称设置的值，如果没有则返回 `None`

**注意**：上述函数会覆盖具有相同名称的现有变量！

`set(key, value, override)`:
- 使用给定的键（名称）将新值插入全局变量集合，可以选择是否覆盖现有值。
- 参数：
    - key: 要设置的全局变量的名称
    - value: 要设置的变量的值
    - override: 如果具有给定名称的现有变量应该被覆盖，则为 `True`；否则为 `False`
- 返回：先前使用给定名称设置的值，如果没有则返回 `None`

`remove(key)`:
- 移除具有给定键（名称）的全局变量。
- 参数：
    - key: 要移除的全局变量的名称
- 返回：被移除的变量的值，如果没有则返回 `None`

`get(key)`:
- 获取具有给定键（名称）的全局变量。
- 返回：具有给定名称的变量，如果没有则返回 `None`

`getKeys()`:
- 获取当前设置的所有全局变量的键（名称）列表。
- 返回：当前存储的所有全局变量名称的不可变集合。如果没有全局变量则返回空集合

`getValues()`:
- 获取当前设置的所有全局变量的值列表。
- 返回：当前存储的所有全局变量值的不可变集合。如果没有全局变量则返回空集合

`getHashMap()`:
- 获取存储所有全局变量的底层 Java HashMap 数据结构。
- 返回：底层 HashMap，它是可变的（可以对其进行更改，这些更改会在全局变量系统中反映）

`contains(key)`:
- 检查当前是否存储了具有给定键（名称）的全局变量。
- 返回：如果有具有给定名称的全局变量则返回 `True`，否则返回 `False`

`containsValue(value)`:
- 检查当前是否存储了具有给定值的全局变量。
- 返回：如果有具有给定值的全局变量则返回 `True`，否则返回 `False`

`purge()`:
- 清除全局变量系统中的所有变量。相当于重置。

## 总结：

- 全局变量系统允许脚本间共享变量。
- 全局变量系统像 PySpigot 的其他管理器一样进行访问。
- 基本使用涉及 `set`、`get` 和 `remove` 函数。
- 全局变量的名称是唯一的。如果使用与现有变量相同的名称插入新变量，则旧值将丢失。
- 存储在全局变量系统中的变量值会自动更新，变化会自动对访问它们的脚本可见，无需再次调用 `set` 函数。
- 当脚本完成使用全局变量时，应移除它们以避免内存泄漏。
- 全局变量系统使用 Java HashMap，并且可以通过 `getHashMap` 函数访问底层的 HashMap。
