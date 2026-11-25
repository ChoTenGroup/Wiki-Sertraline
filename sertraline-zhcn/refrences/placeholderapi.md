---
icon: percent
---

# PlaceholderAPI

安装 PlaceholderAPI 后，可以使用 `/papi parse me %sertraline_PREFIX:PARAM%` 测试占位符。例如，默认物品使用 `%sertraline_random:40to80%` 生成随机值。脚本工具也可在支持的 PAPI 脚本中使用。

| 参数前缀 | 用途 |
| --- | --- |
| `kether:`、`ke:` | 执行 Kether 脚本 |
| `javascript:`、`js:` | 执行 JavaScript 脚本 |
| `jexl:`、`je:` | 执行 JEXL 脚本 |
| `fluxon:`、`fl:` | 执行 Fluxon 脚本 |
| `graaljs:`、`gjs:` | 执行 GraalJS 脚本 |
| `config:` | 按键读取配置值 |
| `random:`、`rand:` | 在指定范围生成随机数，如 `40to80` |
| `data:` | 读取玩家数据 |
| `cdleftmil:` | 获取剩余冷却时间，单位为毫秒 |
| `cdleftsec:` | 获取剩余秒数，四舍五入为整数 |
| `cdleftsec1:`、`cdleftsec2:` | 获取剩余秒数，分别保留一位和两位小数 |
| `cdleftsec1a:`、`cdleftsec2a:` | 同上，去除末尾的 `.0` |

脚本会获得相应上下文，具体可用变量取决于脚本类型和调用位置。
