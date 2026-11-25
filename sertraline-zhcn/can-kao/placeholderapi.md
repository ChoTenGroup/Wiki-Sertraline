---
icon: percent
---

# PlaceholderAPI

安装 PlaceholderAPI 后，可以使用 `/papi parse me %sertraline_PREFIX:PARAM%` 测试占位符。例如，在游戏里执行 `/papi parse me %sertraline_random:40to80%` 可测试随机数。脚本工具也可在支持的 PAPI 脚本中使用。

| 参数前缀 | 用途 |
| --- | --- |
| `kether:`、`ke:` | 执行 Kether 脚本 |
| `javascript:`、`js:` | 执行 JavaScript 脚本 |
| `jexl:`、`je:` | 执行 JEXL 脚本 |
| `fluxon:`、`fl:` | 执行 Fluxon 脚本 |
| `graaljs:`、`gjs:` | 执行 GraalJS 脚本。3.13.2.2 会执行脚本，展示用文本请使用能返回结果的占位符 |
| `config:` | 按键读取配置值 |
| `random:`、`rand:` | 在指定范围生成随机数，如 `40to80` |
| `data:` | 读取在线玩家数据 |
| `data1:`、`data2:` | 读取数值数据，分别按一位和两位小数舍入 |
| `data1a:`、`data2a:` | 同上，去除末尾的 `.0` |
| `cdleftmil:` | 获取剩余冷却时间，单位为毫秒 |
| `cdleftsec:` | 获取剩余秒数，四舍五入为整数 |
| `cdleftsec1:`、`cdleftsec2:` | 获取剩余秒数，分别保留一位和两位小数 |
| `cdleftsec1a:`、`cdleftsec2a:` | 同上，去除末尾的 `.0` |

脚本会获得相应上下文，具体可用变量取决于脚本类型和调用位置。

例如，在游戏里执行 `/papi parse me %sertraline_random:40to80%` 可验证占位符是否已启用。`random:1.5to3.5:2` 则按最多两位小数显示，末尾的零会省略。部分占位符可用 `?:` 指定缺省文本，例如 `%sertraline_data:my_key?:未设置%`。
