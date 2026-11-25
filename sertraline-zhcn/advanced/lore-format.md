---
icon: tent-double-peak
---

# 描述格式

#### LoreSetting 属性 <a href="#loresetting-properties" id="loresetting-properties"></a>

| 属性    | 类型      | 默认值 | 说明                                                                           |
| ----------- | --------- | ------- | ------------------------------------------------------------------------------------- |
| `overwrite` | `Boolean` | `true`  | 为 true 时覆盖原描述；为 false 时追加到原描述                              |
| `visual`    | `Boolean` | `true`  | 为 true 时应用于服务端到客户端的视觉转换；为 false 时应用于服务端物品 |
| `skipBlank` | `Boolean` | `true`  | 如果为 true，则将连续的空行合并为单个空行                        |

#### 关键迭代示例 <a href="#key-iteration-examples" id="key-iteration-examples"></a>

**不指定键（静态行）：**

```
content: "<gray>═══════════════════"
```

生成一行固定文本。

**指定键（动态行）：**

```
key: "sertraline:vals.test"
content: "<green>{plus}{value} {name}"
```

如果 `sertraline:attributes` 包含 `[10, 20, 30]`，生成三行，每行都使用占位符替换值。

{% hint style="warning" %}
有些键与配置的区块不同。例如，MMO属性： `attack-damage` -> `ATTACK_DAMAGE`
{% endhint %}

**使用配置键：**

```
key: "*xbuilder.lore"
content: "{value}"
```

从物品配置段而非运行时数据中读取描述。

### 示例

```yaml
default:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - content: ""
    - key: "mmo:ATTACK_DAMAGE"
      content: "<!i><#777777>| <#ff6936>Attack Damage <#aabbcc>{plus}{minus}{auto}"
    - key: "mmo:ATTACK_SPEED"
      content: "<!i><#777777>| <#26cc79>Attack Speed <#aabbcc>{plus}{minus}{auto}"
    - content: ""
    - key: "sertraline:vals.lore"
      content: "<!i><#777777>| <#cccccc>{value}"
    - key: "sertraline:vals.graylore"
      content: "<!i><#777777>| {value}"
```

```yaml
wooden_sword:
  sertraline:
    tier: C
    lore-format: default
    vals:
      lore: |
        Common wood sword.
      graylore: |
        Just another description
  xbuilder:
    name: "<!i><#bb9900>Wooden Sword"
    material: WOODEN_SWORD
    flags: [HIDE_ATTRIBUTES]
    item-model: "kangelitem:wooden_sword"
  mmo:
    fix-attack-speed: true
    slot: "MAIN_HAND"
    allowed:
      - "hand"
    attack-damage: 5
    attack-speed: 1.6
```

### 进阶

### 条件行渲染 <a href="#conditional-line-rendering" id="conditional-line-rendering"></a>

`LineMode` 决定如何判断多个必需键；系统使用 `handleLoreExists()` 根据 `lineRequire` 列表判断是否显示 `LoreElement`。

#### 行模式行为表 <a href="#linemode-behavior-table" id="linemode-behavior-table"></a>

| LineMode  | Condition                                        | Result       |
| --------- | ------------------------------------------------ | ------------ |
| `ANY`     | `lineRequire` 中至少有一个键存在         | 显示该行 |
| `ALL`     | `lineRequire` 中所有键均存在                  | 显示该行 |
| `NOT`     | `lineRequire` 中没有键存在                   | 显示该行 |
| `NOT_ALL` | 至少一个键在 `lineRequire` 不存在 | 显示该行 |

`handleExistLore()` 检查键是否存在：以 `*` 开头时查询物品配置段，否则通过 `getDeepData()` 查询物品数据。

### 占位符处理系统 <a href="#placeholder-processing-system" id="placeholder-processing-system"></a>

Lore 格式化系统支持按顺序应用三层占位符处理。

```
```

**源码：** [project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt54-101](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/7fbdc299/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt#L54-L101)

#### 内置占位符类型 <a href="#built-in-placeholder-types" id="built-in-placeholder-types"></a>

`performNormalPlaceholders()` 函数根据迭代值处理特殊占位符：

| 占位符 | 行为                                | 示例                           |
| ----------- | --------------------------------------- | --------------------------------- |
| `{plus}`    | 显示 `+` 如果值 > 0，则为空 | `15` → `+`                        |
| `{minus}`   | 显示 `-` 如果值 < 0，则为空，否则不为空 | `-5` → `-`                        |
| `{value}`   | 字符串值                        | `15.5` → `15.5`                   |
| `{auto}`    | 移除 `.0` 整数的后缀       | `15.0` → `15`                     |
| `{round:N}` | 格式化为 N 位小数             | 对 `15.567` 使用 `{round:2}` → `15.57` |

可在 `config.yml` 中通过 `placeholders.plus` 和 `placeholders.minus` 自定义加减号。

#### PlaceholderAPI 集成 <a href="#placeholderapi-integration" id="placeholderapi-integration"></a>

内置占位符之后， `performPlaceholders()` 如果玩家上下文可用，则函数应用PlaceholderAPI替换。

这允许使用标准的 PAPI 占位符，例如 `%player_name%`, `%player_health%`, etc.

#### 内嵌 Kether 求值 <a href="#inline-kether-evaluation" id="inline-kether-evaluation"></a>

内容部分包含： `{{`，系统会将以下内容中的文本视为 `{{...}}` 作为内联Kether脚本。

例如：  `{{ player name }}`&#x20;

这样可以在描述模板中动态计算。

