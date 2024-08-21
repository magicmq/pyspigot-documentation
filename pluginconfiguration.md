# PySpigot 配置

以下是 PySpigot 的配置文件。注释应该足以解释每个值的意义：

```yaml
# 如果设为 false，则禁用 bStats 为 PySpigot 收集度量信息的功能。你也可以在 /plugins/bStats 下的 bStats config.yml 中全局禁用 bStats
metrics-enabled: true
# 服务器完成加载后加载脚本的延迟（以刻为单位）
script-load-delay: 20
# libs 文件夹中库的重定位规则列表。格式为 <pattern>|<relocated pattern>
library-relocations: []
# 脚本日志文件中时间戳的日期/时间格式，采用 Java 的 SimpleDateFormat 模式：https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html
log-timestamp-format: 'MMM dd yyyy HH:mm:ss'
# 如果设为 true，则每次加载、运行和卸载脚本时都会将日志消息打印到控制台
script-action-logging: true
# 如果设为 true，则会将所有 Redis 事件的日志记录到控制台和脚本的日志记录器。如果设为 false，则仅记录重连事件（重连尝试和失败）
verbose-redis-logging: true
# 如果设为 true，当脚本依赖的插件被卸载时，脚本也会自动卸载。这对于确保依赖于某个插件的脚本关闭任务能够成功完成尤其有用（在插件卸载之前）
script-unload-on-plugin-disable: true
# 脚本选项的默认值。如果脚本的 script_options.yml 中未定义一个或多个选项，则 PySpigot 将回退到这些值
script-option-defaults:
  # 脚本是否启用
  enabled: true
  # 脚本依赖的其他脚本列表
  depend: []
  # 脚本依赖的插件列表
  plugin-depend: []
  # 是否应将脚本日志消息记录到相应的日志文件
  file-logging-enabled: true
  # 记录到控制台和脚本日志文件的最低日志级别
  min-logging-level: 'INFO'
  # 权限的默认级别
  permission-default: 'op'
# A脚本的高级调试选项
debug-options:
  # 如果设为 true，则将所有与脚本相关的异常的堆栈跟踪打印到服务器控制台
  print-stack-traces: false
  # 如果设为 true，则会抑制插件版本过时的通知
  suppress-update-messages: false
  # 如果设为 false，则在检测到更改时，pyspigot.py 文件将 不会 自动替换为 pyspigot JAR 中的最新版本。如果插件更新附带了更新的 pyspigot.py 文件，建议保持此选项启用状态
  auto-pyspigot-lib-update-enabled: true
messages:
  plugin-prefix: '&8[&6PySpigot&8] &r'
```