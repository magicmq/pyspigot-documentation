# The `function.py` Helper Module

For convenience, PySpigot includes a helper module called `function.py`. This module wraps several [functional interfaces](https://www.geeksforgeeks.org/functional-interfaces-java/) contained within the [`java.util.function` package](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html) into a Python-friendly format.

This module makes it significantly easier to work with Java's functional interfaces when Java APIs/libraries utilize them. For example, [NBT-API](https://www.spigotmc.org/resources/nbt-api.7939/) makes heavy use of functional interfaces, and the `function.py` helper module makes it much less cumbersome. See the [usage example section](#usage-example) for an example of how to use this module with NBT-API.

## Code

``` py linenums="1"
"""
This is a helper module that wraps a variety of functional interfaces in the java.util.function package, for easier use in Python.

This module should be particularly useful when working with some libraries that make use of functional interfaces (such as NBT-API).
"""
import java.util.function.BiConsumer
import java.util.function.BiFunction
import java.util.function.BiPredicate
import java.util.function.Consumer
import java.util.function.Function
import java.util.function.Predicate
import java.util.function.Supplier

"""
Represents an operation that accepts two input arguments and returns no result. This is the two-arity specialization of Consumer. Unlike most other functional interfaces, BiConsumer is expected to operate via side-effects.
"""
class BiConsumer(java.util.function.BiConsumer):

    """
    Initialize a new BiConsumer.

    Arguments:
        function (callable): A function that accepts two arguments and returns nothing.
    """
    def __init__(self, function):
        self.accept = function


"""
Represents a function that accepts two arguments and produces a result. This is the two-arity specialization of Function. 
"""
class BiFunction(java.util.function.BiFunction):

    """
    Initializes a new BiFunction.

    Arguments:
        function (callable): A function that accepts two arguments and returns a value.
    """
    def __init__(self, function):
        self.apply = function


"""
Represents a predicate (boolean-valued function) of two arguments. This is the two-arity specialization of Predicate. 
"""
class BiPredicate(java.util.function.BiPredicate):

    """
    Initializes a new BiPredicate.

    Arguments:
        function (callable): A function that accepts two arguments and returns either True or False.
    """
    def __init__(self, function):
        self.test = function


"""
Represents an operation that accepts a single input argument and returns no result. Unlike most other functional interfaces, Consumer is expected to operate via side-effects.
"""
class Consumer(java.util.function.Consumer):

    """
    Initializes a new Consumer.

    Arguments:
        function (callable): A function that accepts one argument and returns nothing.
    """
    def __init__(self, function):
        self.accept = function


"""
Represents a function that accepts one argument and produces a result.
"""
class Function(java.util.function.Function):

    """
    Initializes a new Function.

    Arguments:
        function (callable): A function that accepts one argument and returns a value.
    """
    def __init__(self, function):
        self.apply = function


"""
Represents a predicate (boolean-valued function) of one argument.
"""
class Predicate(java.util.function.Predicate):

    """
    Initializes a new Predicate.

    Arguments:
        function (callable): A function that accepts one argument and returns either True or False.
    """
    def __init__(self, function):
        self.test = function


"""
Represents a supplier of results.

There is no requirement that a new or distinct result be returned each time the supplier is invoked.
"""
class Supplier(java.util.function.Supplier):

    """
    Initializes a new Supplier.

    Arguments:
        function (callable): A function that accepts no arguments and returns a value.
    """
    def __init__(self, function):
        self.get = function
```

## Usage Example

The following code utlizes NBT-API to set a custom NBT string to an item using a command, and then fetches the custom NBT string when the player interacts with the item in their hand.

``` py linenums="1"
import pyspigot as ps
from function import Function # (1)!
from de.tr7zw.nbtapi import NBT
from org.bukkit.event.player import PlayerInteractEvent

def interact_event(event):
    item = event.getItem()
    if item is not None:
        def nbt_get(nbt): # (2)!
            return nbt.getString("Test_String")
        nbt_get_function = Function(nbt_get) # (3)!

        nbt_data = NBT.get(item, nbt_get_function) # (4)!
        print(nbt_data)

ps.listener_manager().registerListener(interact_event, PlayerInteractEvent)

def command(sender, label, args):
    item = sender.getItemInHand()

    to_set = 'This is a test string'
    def set_function(nbt): # (5)!
        nbt.setString("Test_String", to_set)
    set_function = Function(set_function) # (6)!

    NBT.modify(item, set_function) # (7)!

ps.command_manager().registerCommand(command, 'additemnbt')
```

1. Here, we make use of the functional interface `Function`. Most of NBT-API uses this functional interface.

2. Here, we define a function that is essentially passed to NBT-API. NBT-API then calls this function, passing the NBT data for the corresponding item as an argument. Then, we fetch our desired NBT data inside the function and return it from the function.

3. Here, we initialize a new `Function`, passing our previously defined `nbt_get` function as an argument.

4. Here, we call the NBT-API, passing the item we want to get data from as well as the `Function` instance we created earlier. We then assign the returned value (the data we fetched).

5. In a similar way as before, we define another function that is essentially passed to NBT-API. NBT-API calls this function, passing the NBT data for the corresponding item. Then, we *set* the NBT data this time inside of the function. We don't return anything, since we are setting data, not getting.

6. Here, we initialize another new `Function`, passing our previously defined `set_function` function as an argument.

7. Finally, we call the NBT-API to modify the NBT data of the item, passing the item whose NBT data we want to set as well as the `Function` instance created earlier. The `NBT.modify` function doesn't return anything since it modifies values.

This is a relatively simple example. If you are having difficulties with more complex code, feel free to ask for help on the PySpigot Discord.