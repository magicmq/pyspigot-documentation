# The `functional` Decorator Module

PySpigot includes a helper module called `functional`, found at `decorators/functional.py`. This module provides Python **decorators** that wrap Python functions as Java [functional interfaces](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/function/package-summary.html) (also known as SAMs — Single Abstract Methods), making it much easier to use Java APIs that expect them.

Many Java APIs, including [NBT-API](https://www.spigotmc.org/resources/nbt-api.7939/) and Java's own `Stream` and `CompletableFuture` APIs, accept functional interface arguments such as `Consumer`, `Function`, `Predicate`, and `Supplier`. Without this module, bridging Python functions to these interfaces requires verbose boilerplate. The `functional` module removes that friction entirely.

## Importing

Import individual decorators directly from the module:

``` py
from decorators.functional import function
from decorators.functional import consumer, predicate, supplier
```

You may import as many decorators as you need in a single statement:

``` py
from decorators.functional import function, consumer, predicate, supplier, runnable
```

## Decorating a Function

Apply a decorator to a Python function using the standard `@decorator` syntax. This wraps your Python function as the corresponding Java functional interface and attaches the Java object to your function as two equivalent attributes:

- `.java` — the canonical accessor for the wrapped Java object
- `.j` — a shorter alias for convenience

The decorated function itself is **unchanged** as a Python callable — you can still call it normally. Only the `.java` / `.j` attributes are added.

``` py linenums="1"
from decorators.functional import function

@function # (1)!
def double(x):
    return x * 2

double(5)           # (2)!

some_api.process(double.java) # (3)!
some_api.process(double.j)    # (4)!
```

1. Apply the `@function` decorator. This wraps `double` as a Java `Function<T,R>` and attaches the Java object as `double.java` and `double.j`.

2. The Python function is unchanged and can still be called normally.

3. Pass the Java functional interface to a Java API using `.java`.

4. `.j` is an identical shorthand for `.java`.

## The `.into()` Method

Every decorated function also gains an `.into(registrar, *args, **kwargs)` convenience method. Calling `fn.into(registrar, ...)` is exactly equivalent to calling `registrar(fn.java, ...)` — it passes the Java object as the first argument to the registrar, followed by any additional arguments you provide.

``` py linenums="1"
from decorators.functional import consumer

@consumer
def on_join(player):
    player.sendMessage('Welcome!')

# These two lines are equivalent:
some_api.register(on_join.java, extra_arg)
on_join.into(some_api.register, extra_arg)
```

???+ note

    `.into()` always passes the Java object as the **first** argument to the registrar. If the API you are using expects the functional interface argument in a different position, use `.java` directly instead.

## `_into` Decorator Factories

For every decorator (e.g. `consumer`), there is a corresponding decorator **factory** named with an `_into` suffix (e.g. `consumer_into`). A `_into` factory takes a registrar function (and any extra arguments) and returns a decorator that both wraps the function *and* immediately calls the registrar on decoration.

This lets you define and register in a single step:

``` py linenums="1"
from decorators.functional import consumer_into
from java.util import Arrays

fruits = Arrays.asList('apple', 'banana', 'cherry')

@consumer_into(fruits.forEach) # (1)!
def print_fruit(name):
    print(name)
```

1. When Python processes this `@` line, `print_fruit` is wrapped as a `Consumer<T>` and `fruits.forEach(print_fruit.java)` is called immediately. No separate call to `forEach` is needed.

The above is exactly equivalent to:

``` py linenums="1"
@consumer
def print_fruit(name):
    print(name)

fruits.forEach(print_fruit.java)
```

`_into` factories accept extra arguments that are forwarded to the registrar after the Java object:

``` py linenums="1"
# Calls: some_api.register(fn.java, 'extra', 123)
@consumer_into(some_api.register, 'extra', 123)
def my_handler(value):
    ...
```

???+ note

    Because the registrar is called at decoration time, the `_into` factories are best suited for situations where the registrar is already available when the function is defined — for example, during module-level setup or script initialization.

## Available Decorators

### Core Decorators

These are the most commonly used decorators, covering the general-purpose functional interfaces in `java.util.function`:

| Decorator | Java Interface | Args | Returns |
|---|---|---|---|
| `runnable` | `Runnable` | none | nothing |
| `consumer` | `Consumer<T>` | one | nothing |
| `bi_consumer` | `BiConsumer<T,U>` | two | nothing |
| `function` | `Function<T,R>` | one | a value |
| `bi_function` | `BiFunction<T,U,R>` | two | a value |
| `predicate` | `Predicate<T>` | one | `boolean` |
| `bi_predicate` | `BiPredicate<T,U>` | two | `boolean` |
| `supplier` | `Supplier<T>` | none | a value |
| `boolean_supplier` | `BooleanSupplier` | none | `boolean` |
| `unary_operator` | `UnaryOperator<T>` | one (type `T`) | same type `T` |
| `binary_operator` | `BinaryOperator<T>` | two (type `T`) | same type `T` |

### Primitive Specializations

The module also covers all primitive-specialized functional interfaces. These are typically only needed when a Java API explicitly requires one (e.g. `IntConsumer` rather than `Consumer<Integer>`):

| Decorator | Java Interface |
|---|---|
| `int_consumer` | `IntConsumer` |
| `int_function` | `IntFunction<R>` |
| `int_predicate` | `IntPredicate` |
| `int_supplier` | `IntSupplier` |
| `int_unary_operator` | `IntUnaryOperator` |
| `int_binary_operator` | `IntBinaryOperator` |
| `int_to_double_function` | `IntToDoubleFunction` |
| `int_to_long_function` | `IntToLongFunction` |
| `double_consumer` | `DoubleConsumer` |
| `double_function` | `DoubleFunction<R>` |
| `double_predicate` | `DoublePredicate` |
| `double_supplier` | `DoubleSupplier` |
| `double_unary_operator` | `DoubleUnaryOperator` |
| `double_binary_operator` | `DoubleBinaryOperator` |
| `double_to_int_function` | `DoubleToIntFunction` |
| `double_to_long_function` | `DoubleToLongFunction` |
| `long_consumer` | `LongConsumer` |
| `long_function` | `LongFunction<R>` |
| `long_predicate` | `LongPredicate` |
| `long_supplier` | `LongSupplier` |
| `long_unary_operator` | `LongUnaryOperator` |
| `long_binary_operator` | `LongBinaryOperator` |
| `long_to_double_function` | `LongToDoubleFunction` |
| `long_to_int_function` | `LongToIntFunction` |
| `obj_int_consumer` | `ObjIntConsumer<T>` |
| `obj_long_consumer` | `ObjLongConsumer<T>` |
| `obj_double_consumer` | `ObjDoubleConsumer<T>` |
| `to_int_function` | `ToIntFunction<T>` |
| `to_long_function` | `ToLongFunction<T>` |
| `to_double_function` | `ToDoubleFunction<T>` |
| `to_int_bi_function` | `ToIntBiFunction<T,U>` |
| `to_long_bi_function` | `ToLongBiFunction<T,U>` |
| `to_double_bi_function` | `ToDoubleBiFunction<T,U>` |

### `_into` Variants

Every decorator in both tables above has a corresponding `_into` factory variant. The naming pattern is simply `<decorator_name>_into` — for example:

- `consumer` → `consumer_into`
- `bi_function` → `bi_function_into`
- `int_predicate` → `int_predicate_into`

Refer to the [section above](#into-decorator-factories) for usage details.

## Code Examples

### NBT-API

The following example uses [NBT-API](https://www.spigotmc.org/resources/nbt-api.7939/) to read and write custom NBT data on items. NBT-API uses `Function`-style interfaces for reading (since a value is returned) and `Consumer`-style interfaces for writing (since only a side-effect is needed, with no return value).

``` py linenums="1"
import pyspigot as ps
from decorators.functional import function, consumer # (1)!
from de.tr7zw.nbtapi import NBT
from org.bukkit.event.player import PlayerInteractEvent

def on_interact(event):
    item = event.getItem()
    if item is None:
        return

    @function # (2)!
    def read_nbt(nbt):
        return nbt.getString('my_key') # (3)!

    stored = NBT.get(item, read_nbt.java) # (4)!
    if stored:
        event.getPlayer().sendMessage(f'Item NBT value: {stored}')

ps.listener_manager().registerListener(on_interact, PlayerInteractEvent)


def stamp_item(item):
    @consumer # (5)!
    def write_nbt(nbt):
        nbt.setString('my_key', 'hello from PySpigot') # (6)!

    NBT.modify(item, write_nbt.java) # (7)!
```

1. Import `function` for reading NBT (one argument, returns a value) and `consumer` for writing NBT (one argument, no return value).

2. `@function` wraps `read_nbt` as a Java `Function<ReadableNBT, T>`. NBT-API will call this function, passing the item's NBT compound as `nbt`.

3. Read and return the string stored under `'my_key'`. The returned value is what `NBT.get` hands back to us on line 9.

4. Call `NBT.get`, passing the item and `read_nbt.java` (the wrapped Java function). The return value is whatever `read_nbt` returned.

5. `@consumer` wraps `write_nbt` as a Java `Consumer<ReadWriteNBT>`. No value is returned — NBT-API just calls the function to let us mutate the NBT compound in place.

6. Set the string `'my_key'` to `'hello from PySpigot'` on the mutable NBT compound.

7. Call `NBT.modify`, passing the item and `write_nbt.java`. NBT-API invokes `write_nbt`, which sets the NBT data, and then saves the changes back to the item.

### Java Streams

Java's Stream API makes heavy use of functional interfaces. The following example streams a list of online players, filters for operators, maps each to their name, and prints every result.

``` py linenums="1"
from decorators.functional import predicate, function, consumer # (1)!
from org.bukkit import Bukkit
from java.util import ArrayList

players = ArrayList(Bukkit.getOnlinePlayers()) # (2)!

@predicate # (3)!
def is_op(player):
    return player.isOp()

@function # (4)!
def get_name(player):
    return player.getName()

@consumer # (5)!
def log_name(name):
    print(f'Online operator: {name}')

players.stream() \
    .filter(is_op.java) \   # (6)!
    .map(get_name.java) \   # (7)!
    .forEach(log_name.java) # (8)!
```

1. Import `predicate` (one arg, returns boolean), `function` (one arg, returns value), and `consumer` (one arg, no return).

2. Wrap the online player collection in an `ArrayList` so it exposes `.stream()`.

3. `@predicate` wraps `is_op` as a `Predicate<Player>`. `stream.filter` will call this for each player, keeping only those for whom it returns `True`.

4. `@function` wraps `get_name` as a `Function<Player, String>`. `stream.map` will call this on each remaining player to transform it into their name string.

5. `@consumer` wraps `log_name` as a `Consumer<String>`. `stream.forEach` calls this for each name, printing it.

6. Filter the stream to only operator players.

7. Map each operator `Player` object to their name string.

8. Print each name.
