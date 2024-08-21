# PySpigot

> 为你的 Minecraft 服务器提供的 Python 脚本引擎。

## PySpigot 是什么？

PySpigot 是一个 Python 脚本引擎，允许你编写能够无缝地与 Bukkit 和 Minecraft 服务器的其他部分交互的 Python 脚本，而无需编写插件。脚本在运行时可以完全访问 Java 类路径（意味着任何事情都是可能的！），同时也可以访问 PySpigot 的管理器，这些管理器有助于简化脚本编写过程。

**PySpigot 正式支持 Spigot 和 Paper 服务器，在 Minecraft 1.16.1 及以上版本上运行。** 我看到一些用户的反馈表明 PySpigot 也可以在其他服务器软件以及像 1.4.7 这样较旧的版本上运行，但我不能保证 PySpigot 在未经测试的环境下的兼容性。

**PySpigot 使用 JDK 17 构建。** 这意味着它只能在 Java 17 及以上版本上运行。

## 功能

- 在服务器启动时或通过命令加载脚本
- 通过命令停止、重新加载和卸载服务器脚本
- 注册事件监听器
- 注册命令
- 安排任务（同步和异步）
- 处理配置文件
- 注册 ProtocolLib 数据包监听器
- 注册 PlaceholderAPI 占位符扩展
- 每个脚本都有详细的错误和异常日志记录
- 加载你想在运行时使用的 Java 库
- 使用 Python 语法编写脚本
- 脚本可以完全访问 Bukkit/Spigot API 以及其他插件的 API，因此几乎可以实现任何功能。
- 更多！

## 有用链接

- [在 Spigot 上查看 PySpigot](https://www.spigotmc.org/resources/pyspigot.111006/)
- [在 GitHub 上查看 PySpigot](https://github.com/magicmq/pyspigot)
- [PySpigot Java 文档](https://javadocs.magicmq.dev/pyspigot/)

## 开始使用

请参阅 [快速入门指南](quickstart.md) 获取简短教程。查阅文档的其余部分以了解更高级的使用方法。

## 示例

请参阅任何 [示例](examples.md) 来查看一些 PySpigot 脚本示例。

## 更新

当 PySpigot 更新到新版本时，更新将会推送到 Spigot 以及官方 GitHub 仓库。

如果你正在运行过时版本的 PySpigot，你会在两个地方收到更新提醒：

- 在控制台中，当插件加载时（如服务器启动时）。
- 登录服务器时在聊天窗口中。这条消息只会显示给拥有 `pyspigot.admin` 权限的玩家。

?> 建议始终使用 PySpigot 的最新版本。因为插件还处于早期开发阶段，新版本很可能包含重要的 Bug 修复和改进。

### Discord

加入官方 PySpigot Discord 服务器：https://discord.gg/f2u7nzRwuk

## 统计数据

PySpigot 使用 [bStats](https://bstats.org/) 来收集匿名使用数据。如果你想禁用此功能，请在 PySpigot 的 config.yml 中将 `metrics-enabled` 设置为 `false`。或者，你可以在 /plugins/bStats/config.yml 中将 `enabled` 设置为 `false` 来全局禁用 bStats。

## 许可证

请查阅 [许可证信息](#license.md) 页面以了解许可证详情。
