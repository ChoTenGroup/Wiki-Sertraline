---
icon: sensor-cloud
---

# 物品文本标签

用 `${标签:参数}$` 把数据插入名称、描述或其他支持标签的位置。下例会显示比赛轮数、奖品和联系人。把配置保存到 `plugins/Sertraline/workspace/event_pass.yml`，重载后运行 `/sertralineitem give event_pass`。

```yaml
event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动通行证"
    lore:
      - "<!i><gray>轮数 ${val:roundsAsIn}$"
      - "<!i><gray>奖品 ${val:rewardsAsLiSplitBy / }$"
      - "<!i><gray>联系人 ${val:contact?:—}$"
  sertraline:
    vals:
      rounds: 2.7
      rewards: ["Gold", "Paper"]
```

描述会显示轮数 `3`、奖品 `Gold / Paper`。由于没有定义 `contact`，联系人一栏会显示 `—`。如果在 `vals` 中添加 `contact: "Steve"`，重载后重新领取即可看到新值。

## 选择和格式化数据

- `val` 读取 `sertraline.vals`，`var` 读取物品存储的变量，`dynamic` 用于按需计算的值。见[物品数据](item-structures/section-sertraline/item-data.md)。
- 在键后面加 `?:默认值` 可提供缺省文本，加 `!!` 可用空文本代替 `null`。如果对应数据段整个为空，标签可能原样保留。示例中的 `vals` 已包含其他键。
- 可在键后附加 `AsIn`（四舍五入为整数）、`AsSt`（文本）、`AsDo` / `AsFl`（浮点数）、`AsBy`（字节）、`AsLo`（长整数）、`AsBo`（布尔值）、`AsBd`（高精度小数）、`AsLi`（拼接列表）或 `AsMa`（拼接键值对）。

## 修改列表和键值对的分隔符

列表默认用逗号拼接。用 `AsLiSplitBy / ` 时，用斜杠两边各带一个空格的分隔符。用 `AsLiSplitByBLANK` 可以直接连写。单个空格也需要写在 `SplitBy` 后面。

键值对可在 `AsMaSplitBy` 后用反引号隔开两种分隔符，前面用于键与值，后面用于各条目。例如标签 ``${val:statsAsMaSplitBy=` / }$`` 会把 `{ hp: 10, speed: 2 }` 显示成 `hp=10 / speed=2`。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：`AsIn` 会四舍五入为整数；列表和键值对分隔符分别由 `AsLi`、`AsMa` 转换处理。若整个数据来源段为空，标签可能保持原样，因此示例在每个引用的数据段中至少定义了一个值。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: sensor-cloud
---

# 物品文本标签

物品文本标签允许你在物品配置中使用标签。

### 用法

在物品配置中使用 `${tag:param}$` 格式。

例如：

<pre><code>sword_of_lucky:
  sertraline:
    tier: S
    vars:
<strong>      damage: "%sertraline_random:40to80%"
</strong>  xbuilder:
    name: "&#x3C;!i>&#x3C;b>&#x3C;u>&#x3C;#9999ff>Mr. BlueICE's&#x3C;/u> &#x3C;yellow>&#x3C;i>Lucky Sword"
    lore: |
      &#x3C;!i>&#x3C;#66ccff>a sword contains random damage in generation.
    material: DIAMOND_SWORD
    attributes:
      ATTACK_DAMAGE:
<strong>        amount: "${var:damage}$"
</strong>        operation: ADD_NUMBER
        slot: HAND
</code></pre>

构建物品时，`${var:damage}$` 会被替换为变量值。

### 进阶

#### 非空值与默认值

使用非空后缀以避免“null”。使用 `?:` 以获取默认值。示例：

```
vars:
  test: null
```

* `"${var:test}$"` - `null`&#x20;
* `"${var:test!!}$"` -   空字符串
* `"${var:test?:DEF}$"` - `DEF` （默认值）            &#x20;

#### 类型转换 As\<Type>

可以使用 As\<Type> 后缀转换标签的值类型：

**可用的类型转换：**

| 修饰符 | 代码 | 说明                  | 示例                                             |
| -------- | ---- | ---------------------------- | --------------------------------------------------- |
| `AsSt`   | `St` | 转换为字符串            | `${val:numberAsSt}$` → `"42"`                       |
| `AsIn`   | `In` | 转换为整数（四舍五入） | `${val:priceAsIn}$` → `100`                         |
| `AsDo`   | `Do` | 转换为双精度浮点数            | `${val:multiplierAsDo}$` → `1.5`                    |
| `AsFl`   | `Fl` | 转换为单精度浮点数             | `${val:ratioAsFl}$` → `0.75f`                       |
| `AsBy`   | `By` | 转换为字节              | `${val:flagAsByByte}$` → `1`                        |
| `AsLo`   | `Lo` | 转换为长整数              | `${val:timestampAsLo}$` → `1234567890`              |
| `AsBo`   | `Bo` | 转换为布尔值           | `${val:enabledAsBo}$` → `true`                      |
| `AsBd`   | `Bd` | 转换为高精度小数        | `${val:preciseAsBd}$` → `123.456789`                |
| `AsLi`   | `Li` | 将列表转换为字符串       | `${val:enchantmentsAsLi}$` → `"Fire,Ice,Lightning"` |
| `AsMa`   | `Ma` | 将映射转换为字符串        | `${val:statsAsMa}$` → `"STR,50,DEX,30"`             |

```
vars:
 test: 1.99
```

* `"${var:testAsIn}$"` - `2`

### 集合的 SplitBy 修饰符 <a href="#splitby-modifiers-for-collections" id="splitby-modifiers-for-collections"></a>

`SplitBy` 修饰符控制列表和映射拼接成字符串时使用的分隔符，必须与 `AsLi` 或 `AsMa` 一起使用。

**语法：**

```
${prefix:keyAsLiSplitBySeparator}$     # 用于列表
${prefix:keyAsMaSplitByKV`entry}$      # 用于映射
```

**列表示例：**

| Tag                              | 输入列表                 | 输出                           |
| -------------------------------- | -------------------------- | -------------------------------- |
| `${val:colorsAsLi}$`             | `["Red", "Blue", "Green"]` | `"Red,Blue,Green"` （默认分隔符为 `,`） |
| `${val:colorsAsLiSplitBy }$`     | `["Red", "Blue", "Green"]` | `"Red Blue Green"` （空格）       |
| `${val:colorsAsLiSplitBy-}$`     | `["Red", "Blue", "Green"]` | `"Red-Blue-Green"` （短横线）        |
| `${val:colorsAsLiSplitByBLANK}$` | `["Red", "Blue", "Green"]` | `"RedBlueGreen"` （无分隔符）  |

**映射示例：**

映射使用两个部分的分隔符，以 `` ` `` 分开（3.8.2 及更早版本使用 `|`）：

* 第一部分：将键与值分开
* 第二部分：分隔条目

| Tag                              | 输入映射            | 输出                                   |
| -------------------------------- | -------------------- | ---------------------------------------- |
| `${val:statsAsMa}$`              | `{STR: 50, DEX: 30}` | `"STR,50,DEX,30"` （两个分隔符默认为 `,`） |
| ``${val:statsAsMaSplitBy=`}$``   | `{STR: 50, DEX: 30}` | ``"STR=50`DEX=30"``                      |
| ``${val:statsAsMaSplitBy:`\n}$`` | `{STR: 50, DEX: 30}` | `"STR:50\nDEX:30"`                       |

特殊值：

* `SplitByBLANK` - 使用空字符串（无分隔符）
* `SplitBy` - 使用单空格

```
                "St" -> input.toString()
                "In" -> input.toString().toDoubleOrNull()?.roundToInt()
                "Do" -> input.toString().toDoubleOrNull()
                "Fl" -> input.toString().toFloatOrNull()
                "By" -> input.toString().toByteOrNull()
                "Lo" -> input.toString().toLongOrNull()
                "Bo" -> input.toBooleanTolerance()
                "Bd" -> input.toString().toBigDecimal()
                "Li" -> (input as? List<*>?)?.joinToString(sep)
                "Ma" -> {
                    val split = sep.split("`")
                    (input as? Map<*, *>?)?.joinToStringExtended(split[0], if (split.size > 1) split[1] else ",")
                }
```


</details>
