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

