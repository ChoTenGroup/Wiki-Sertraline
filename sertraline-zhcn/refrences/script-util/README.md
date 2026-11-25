---
icon: subscript
---

# 脚本工具

<a id="utility-reference-table"></a>

脚本可使用一些工具对象处理文本、物品、玩家和数据。具体可用对象取决于脚本引擎及执行上下文，详见[脚本工具和 API 参考](script-util-refs.md)。

| 对象 | 用途 |
| --- | --- |
| `mmUtil`、`mmJsonUtil` | 处理 MiniMessage 和 JSON 文本 |
| `mmLegacySectionUtil`、`mmLegacyAmpersandUtil` | 转换传统颜色代码 |
| `jsonUtils` | JSON 序列化和反序列化 |
| `ItemStackUtil` | 操作物品 |
| `EventUtil`、`ThreadUtil` | 事件和线程辅助方法 |
| `PlayerUtil` | 玩家相关操作 |
| `SertralineAPI` | 访问插件公开接口 |
| `DataUtil` | 玩家数据和冷却时间 |
| `Math`、`System`、`Bukkit`、`Gson` | 数学、系统、服务器和 JSON 工具 |

脚本可能访问服务器资源，请先检查外部脚本的来源和内容。
