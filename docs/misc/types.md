# Python/Java Type Coercion

Jython handles type coercion automatically between Java and Python. However, it is useful to know the equivalency of Python types to Java types. The following table describes the equivalency of all types available in Python to their Java counterparts.

| **Python Type**            | **Jython PyObject Subclass**   | **Java Type**                         | **Description**                                    |
| -------------------------- | ------------------------------ | ------------------------------------- | -------------------------------------------------- |
| `int`                      | `PyInteger`                    | `java.lang.Integer`                   | Maps Python `int` to a Java `Integer`.             |
| `float`                    | `PyFloat`                      | `java.lang.Double`                    | Maps Python `float` to Java `Double`.              |
| `str`                      | `PyString`                     | `java.lang.String`                    | Maps Python `str` to Java `String`.                |
| `bool`                     | `PyBoolean`                    | `java.lang.Boolean`                   | Maps Python `True`/`False` to Java `Boolean`       |
| `None`                     | `PyNone`                       | `null`                                | Python `None` translates to Java `null`.           |
| `long` (Python 2 only)     | `PyLong`                       | `java.math.BigInteger`                | Maps Python `long` to `BigInteger`.                |
| `complex`                  | `PyComplex`                    | `org.python.core.PyComplex`           | Represents complex numbers.                        |
| `list`                     | `PyList`                       | `java.util.List`                      | Maps Python `list` to Java `List`.                 |
| `tuple`                    | `PyTuple`                      | `java.util.List` (immutable)          | Maps Python `tuple` to Java `List` but immutable.  |
| `dict`                     | `PyDictionary`                 | `java.util.Map`                       | Maps Python `dict` to Java `Map`.                  |
| `set`                      | `PySet`                        | `java.util.Set`                       | Maps Python `set` to Java `Set`.                   |
| `frozenset`                | `PyFrozenSet`                  | `java.util.Set` (immutable)           | Maps Python `frozenset` to immutable Java `Set`.   |
| `bytes`                    | `PyString`                     | `byte[]`                              | Maps Python `bytes` to Java `byte[]`.              |
| `bytearray`                | `PyByteArray`                  | `byte[]`                              | Mutable version of `bytes` mapped to `byte[]`      |
| `memoryview`               | `PyMemoryView`                 | `org.python.core.PyMemoryView`        | Maps Python `memoryview` to Java PyMemoryView.     |
| `function`                 | `PyFunction`                   | `org.python.core.PyFunction`          | Python functions map to PyFunction in Java.        |
| `method`                   | `PyMethod`                     | `org.python.core.PyMethod`            | Python methods map to PyMethod.                    |
| `class`                    | `PyClass`                      | `org.python.core.PyClass`             | Represents Python classes in Java.                 |
| `instance`                 | `PyInstance`                   | `org.python.core.PyInstance`          | Maps Python instances to PyInstance.               |
| `module`                   | `PyModule`                     | `org.python.core.PyModule`            | Represents Python modules in Jython.               |
| `callable` (e.g., lambdas) | `PyCallable`                   | `org.python.core.PyObject` (callable) | Maps callable Python objects like lambdas.         |
| `object` (general)         | `PyObject`                     | `java.lang.Object`                    | Generic Python object maps to `Object` in Java.    |
| `iter` (iterator)          | `PyIterator`                   | `java.util.Iterator`                  | Python `iterator` maps to Java `Iterator`.         |
| `generator`                | `PyGenerator`                  | `org.python.core.PyGenerator`         | Maps Python generator objects.                     |
| `Exception`                | `PyException`                  | `java.lang.Throwable`                 | Python exceptions map to Java `Throwable`.         |
| `file` (Python 2 only)     | `PyFile`                       | `java.io.InputStream`/`OutputStream`  | Maps Python file objects to Java IO streams.       |
| `ellipsis` (`...`)         | `PyEllipsis`                   | `org.python.core.PyEllipsis`          | Represents the `...` object in Python.             |
| `NotImplemented`           | `PyNotImplemented`             | `org.python.core.PyNotImplemented`    | Represents `NotImplemented` in Python.             |
| `slice`                    | `PySlice`                      | `org.python.core.PySlice`             | Represents Python slices.                          |
| `buffer` (Python 2 only)   | `PyBuffer`                     | `org.python.core.PyBuffer`            | Represents Python buffer objects.                  |
| `type` (metaclasses)       | `PyType`                       | `org.python.core.PyType`              | Maps Python metaclasses (`type`) in Java.          |
| `custom objects`           | User-defined `PyObject`        | User-defined Java class or interface  | Custom Python classes map to Java `PyObject`.      |