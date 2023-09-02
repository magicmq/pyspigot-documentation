# External Libraries

There are many open source Java libraries that greatly simplify writing more complicated code. Many libraries also add additional functionalities that would otherwise take time to write yourself. The [Apache Commons Libraries](https://commons.apache.org/) are one such example. Normally, these libraries/dependencies would be "shaded" into a plugin (with the end result sometimes called a "fat" Jar or "Uber" Jar) so that the plugin has access to the dependency at runtime. We obviously can't do this with scripts.

PySpigot scripts have access to the entire Java class path at runtime. This means that a script can access any Java class that is loaded and running on the server. As a consequence, if we manually load a Jar file (and the class files contained within) into the Java classpath, then a script should be able to access it. Fortunately, PySpigot has a library manager that does just that, so that you may use external libraries/dependencies in your scripts.

If you find a library/dependency you would like to use within your script, but it is not loaded onto the server already (another plugin isn't already using it), you can load the library/dependency using PySpigot's library manager.

## Loading and Using Libraries

!> It's possible that another plugin on the server is already using the library/dependency that you'd like to use. You should check if this is the case before you use PySpigot to load the library yourself. You can do this by trying to `import` the dependency into your script. If you're able to import it, stop here! You won't need to load it yourself, as it is already loaded (likely by another plugin).

All Java libraries/dependencies *should* have a Jar file available for download somewhere, perhaps on its Github repository or on its official website. You may need to download it manually from the repository where the dependency is hosted. If you can't find the Jar file, reach out on PySpigot's Discord for help.

Once you have the physical Jar file for the dependency you'd like to use, drag and drop it into the `libs` folder of the main PySpigot plugin folder. The dependency will be automatically loaded when the PySpigot plugin is loaded and enabled (on server startup). From here, you can use `import` statements in your script to import the desired code from your dependency.

!> All Jar files within the `libs` folder will be automatically loaded. If you're no longer using a library, be sure to delete it from the `libs` folder to avoid unnecessary use of server memory.

### Loading a library when PySpigot is already running

To load a library/dependency if PySpigot is already running, follow these steps:

1. Drag and drop the Jar into the `libs` folder, located within PySpigot's main plugin folder.
2. Load the library with the command `/loadlibrary <filename>`, where `<filename>` is the name of the Jar file library you would like to load (including the .Jar extension).
3. Import the desired code in your script from the library/dependency with Python `import` statements, as you normally would with the Bukkit API.

!> PySpigot does not have a way to unload previously loaded libraries due to limitations within Java's class loading system. Nevertheless, a library *will not* be loaded more than once; if it already loaded, PySpigot will not load it again.

## Jar Relocation Rules

You may need to change the name/classpath of classes within a library/dependency. This is called "relocation". There are many reasons why you might want to do this, most of which are too complicated to explain here. If you need to relocate the classpath of your library/dependency, you'll likely know that already. In that case, read on for information on how to specify relocation rules.

Within PySpigot's `config.yml`, you'll find a `library-relocations` value. You may specify your own relocations rules here. Relocation rules should be in the format `<pattern>|<relocation>`. For example, if we want to relocate `com.apache.commons.lang3.stringutils` to `lib.com.apache.commons.lang3.stringutils`, we would specify this in the config like this:

```yaml
...
# List of relocation rules for libraries in the libs folder. Format as <pattern>|<relocated pattern>
library-relocations: ['com.apache.commons.lang3.stringutils|lib.com.apache.commons.lang3.stringutils']
...
```

### Jar Relocation Example

Coming soon