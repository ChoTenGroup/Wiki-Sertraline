---
icon: subscript
---

# 脚本工具

<a id="utility-reference-table"></a>

当脚本上下文注入 Sertraline 默认脚本变量时，可使用下列内置名称。集成插件还可以添加其他变量；实际可用对象仍取决于执行上下文。

| 变量名 | 类型与用途 |
| --- | --- |
| `mmUtil` | MiniMessage 工具：将 MiniMessage 字符串反序列化为 Component。 |
| `mmJsonUtil` | MiniMessage JSON 工具。 |
| `mmLegacySectionUtil` | 将传统 `§` 颜色代码转换为 Component。 |
| `mmLegacyAmpersandUtil` | 将传统 `&` 颜色代码转换为 Component。 |
| `jsonUtils` | Gson JSON 序列化与反序列化工具。 |
| `ItemStackUtil` | ItemStack 操作辅助工具。 |
| `EventUtil` | 事件辅助工具。 |
| `ThreadUtil` | 线程管理辅助工具。 |
| `PlayerUtil` | 玩家相关辅助工具。 |
| `SertralineAPI` | SertralineAPI 实例，用于访问插件公开 API。 |
| `DataUtil` | 玩家数据存储与冷却管理工具。 |
| `Math` | `java.lang.Math` 类，提供数学操作。 |
| `System` | `java.lang.System` 类。 |
| `Bukkit` | `org.bukkit.Bukkit` 类及服务器 API。 |
| `Gson` | `Class<Gson>` 类引用，并非预先配置好的 Gson 实例。 |
| `ExternalItemHelper` | 外部物品辅助工具，可解析或构建受支持的第三方物品引用。 |
| `AttributeAPI` | Sertraline 属性操作 API 对象。 |
| `bukkitPlugin` | Bukkit 插件实例。 |
| `scheduler` | Bukkit 调度器，可用于安排服务器任务。 |
| `SertralineObj` | Sertraline 插件单例对象。 |
| `FluxonShell` | Fluxon 脚本操作辅助工具。 |
| `UUIDUtil` | UUID 相关辅助工具。 |
| `LoreFormatUtil` | 物品描述格式工具。 |
| `TierUtil` | 品级数据查询工具。 |
| `TypeUtil` | 类型数据查询工具。 |

{% hint style="warning" %}
部分脚本上下文会暴露 `System`、`Bukkit` 等 Java 或服务器类。请只运行可信脚本；玩家或世界相关操作应在服务器主线程执行。
{% endhint %}

更多细节见[脚本工具和 API 参考](script-util-refs.md)。
