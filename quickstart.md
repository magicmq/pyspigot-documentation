# 快速入门指南

这里是一个非常简短的指南，教你如何创建第一个 PySpigot 脚本。如果你不懂 Python 没关系！有很多很棒的在线教程可以帮助你学习。请参阅 [有用资源](externalresources.md) 页面以获取一些好的 Python 教程。

## 下载并加载 PySpigot

PySpigot 正式支持 Spigot 和 Paper 服务器，并且支持 Minecraft 1.16 及以上版本。我不能保证 PySpigot 在这些条件以外的情况也能正常工作，但是一些用户报告称在其他服务器软件或较旧的 Minecraft 版本上也能成功运行。

自 0.6.0 版本起，PySpigot 使用 Java 17 构建。这意味着对于 0.6.0 及以后的版本，PySpigot 需要 Java 17 或更高版本。

从 [GitHub](https://github.com/magicmq/pyspigot) 或者 [Spigot](https://www.spigotmc.org/resources/pyspigot.111006/) 下载 PySpigot 的最新版本。将下载的 JAR 文件放入你的插件文件夹，并启动服务器。

## 创建你的第一个脚本

在这个简短的教程中，我们将创建一个非常简单的脚本，它会向在线玩家广播一条消息。

### 创建脚本文件

所有的脚本都位于 PySpigot 主插件文件夹内的 `scripts` 文件夹中。PySpigot 允许在脚本文件夹内创建子文件夹以方便组织，但是脚本名称在整个子文件夹范围内必须是唯一的。

在这里创建一个 Python 脚本文件，并命名为你喜欢的名字。文件名将作为该脚本的名称，稍后将用于加载和卸载脚本。

!> PySpigot 只会加载和运行以 `.py` 结尾的脚本文件。你可以通过更改文件扩展名（例如，添加 `.disabled`）轻松禁用脚本而无需删除它。

### 编写脚本

使用你选择的文本编辑器打开刚刚创建的脚本文件，并编写一些代码：

```python
from org.bukkit import Bukkit

Bukkit.broadcastMessage('Hello world!')
```

第 1 行，我们从 Bukkit Java API 导入了 Bukkit 类。这是一个非常有用的类，很可能在你的许多脚本中都会用到。

第 3 行，我们调用了 `broadcastMessage` 方法/函数，它会向所有在线玩家广播一条消息。

### 运行脚本

保存文件，并启动服务器。如果你一切操作正确，你的脚本应该会在服务器启动时自动加载。这是预期的行为；PySpigot 会在插件加载时自动加载并运行 `scripts` 文件夹中的所有脚本，包括子文件夹中的脚本。

或者，如果服务器已经运行并且 PySpigot 插件已经加载并启用，你可以使用 `/pyspigot load <scriptname>` 加载并运行你的脚本。确保名称包含扩展名（.py）！如果脚本位于子文件夹中，你不需要指定完整路径。只需要指定脚本文件名即可。

## 接下来做什么

查阅文档的其余部分以了解更高级的脚本编写技巧。

如果你遇到困难需要帮助，加入 PySpigot 的 Discord 服务器（如果你还没有加入的话）：https://discord.gg/f2u7nzRwuk
