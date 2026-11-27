---
icon: percent
---

# PlaceholderAPI

安装 PlaceholderAPI 后，Sertraline 会自动注册 `%sertraline_...%` 扩展。

## 语法与测试

使用 `%sertraline_<前缀>:<参数>%`。前缀选择要执行的功能，冒号后的内容是该功能的输入。例如，在游戏中执行以下命令测试随机整数：

```text
/papi parse me %sertraline_random:40to80%
```

## 支持的前缀

| 前缀 | 用途 |
| --- | --- |
| `kether`、`ke` | 在上下文中以 `player` 和 Sertraline 默认脚本工具运行 Kether；若 PAPI 传入的玩家在线，该玩家也会作为 Kether 执行者。 |
| `javascript`、`js` | 使用 `player` 和默认脚本工具运行 JavaScript；已编译脚本会缓存。 |
| `jexl`、`je` | 使用 `player` 和默认脚本工具运行 JEXL；已编译脚本会缓存。 |
| `graaljs`、`gjs` | 使用 `player` 和默认脚本工具执行 GraalJS。 |
| `fluxon`、`fl` | 使用 `player` 和默认脚本工具运行 Fluxon。 |
| `config` | 按键读取 Sertraline 配置中的原始值，例如 `config:debug`。 |
| `random`、`rand` | 按 `起始值to结束值` 生成随机数，例如 `40to80` 或 `1.5to3.5`；可用 `:2` 指定小数位数。 |
| `data` | 按键读取玩家已保存的原始数据。 |
| `data1`、`data2` | 读取玩家数值数据，分别舍入到一位或两位小数。 |
| `data1a`、`data2a` | 与对应的数值格式相同，但会去掉末尾的 `.0`。 |
| `cdleftmil` | 按键读取剩余冷却时间，单位为毫秒。 |
| `cdleftsec`、`cdleftsec1`、`cdleftsec2` | 读取剩余冷却秒数，分别舍入为整数、保留一位或两位小数。 |
| `cdleftsec1a`、`cdleftsec2a` | 一位或两位小数的冷却秒数格式，并去掉末尾的 `.0`。 |

多个返回单一值的前缀支持在 `?:` 后指定缺省文本，例如 `%sertraline_data:my_key?:未设置%`。这是值类前缀的缺省值语法，不是所有脚本语言通用的功能。

{% hint style="warning" %}
脚本类前缀会执行代码，并可访问 Sertraline 的脚本工具和当前玩家上下文。只使用可信脚本；不要假定每次 PlaceholderAPI 调用都在服务器主线程执行。
{% endhint %}
