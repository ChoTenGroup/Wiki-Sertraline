---
icon: tent-double-peak
---

# 描述格式

你可以用同一套描述格式为多件物品显示统一的类型、品级和等级。把下面的配置保存为 `plugins/Sertraline/lore-formats/practice.yml`。

```yaml
practice_lore:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - key: "xbuilder:lore"
      content: "{value}"
    - key: "sertraline:type"
      content: "<!i><gray>类型 {type:name}"
    - key: "sertraline:tier"
      content: "<!i><gray>品级 {tier:format}"
    - key: "sertraline:level"
      content: "<!i><gray>等级 {level:name}"
```

将[物品品级示例](item-tier.md)中 `classified_sword` 的 `lore-format: loreformat` 改为 `lore-format: practice_lore`。重载后重新领取这把剑。原描述和三项分类信息会按 `elements` 中的顺序显示。

## 调整显示的内容

- 只填 `content` 可添加固定文本行。指定 `key` 后，用 `{value}` 显示该键的值。如果值是列表，列表的每项各生成一行。
- `key: "sertraline:vals.note"` 读取物品的 `vals.note`。`key: "*sertraline.vals.note"` 则从原物品配置直接读取，前缀 `*` 后的层级用点号分隔。读取 MMO 属性时使用转换后的大写键，例如 `mmo:ATTACK_DAMAGE`。
- 物品行也可用 `{val:note}` 引用固定值。要把同一描述应用于多件物品，在每件物品的 `sertraline` 段填写相同的 `lore-format` ID。

| `settings` 选项 | 默认值 | 作用 |
| --- | --- | --- |
| `overwrite` | `true` | 覆盖原描述。设为 `false` 时在原描述后追加 |
| `visual` | `false` | 开启后通过发包显示，需要 PacketEvents 和发包描述开关 |
| `auto-skip-blank` | `true` | 合并连续空行 |

## 仅在有数据时显示一行

把下面的条目加到 `practice_lore` 的 `elements` 列表末尾。然后在剑的 `sertraline` 下添加 `vals: { note: "活动奖励" }`。重载后，只有提供了 `note` 的物品才会出现提示。

```yaml
- content: "<!i><yellow>备注 {val:note}"
  lineMode: ALL
  lineRequire:
    - "sertraline:vals.note"
```

当 `lineRequire` 中有多个键时，`ANY` 表示至少有一项存在，`ALL` 表示全部存在，`NOT` 表示全部缺失，`NOT_ALL` 表示至少缺失一项。

## 格式化数值

| 写法 | 显示结果 |
| --- | --- |
| `{value}` | 保留值的原有文本 |
| `{auto}` | 移除末尾的 `.0`，例如 `15.0` 显示为 `15` |
| `{plus}{auto}` | 为正数添加加号，例如 `+15`。负数保留已有的负号 |
| `{minus}` | 在值为负数时单独显示负号 |
| `{round:2}` | 在当前版本显示两位小数，例如 `15.567` 显示为 `15.57`。冒号后的数字是格式宽度 |

用 `config.yml` 中的 `placeholders.plus` 和 `placeholders.minus` 可调整专用符号。有玩家上下文时，`content` 可使用 PlaceholderAPI 占位符，也可用 `{{ ... }}` 插入 Kether。随附的 `lore-formats/compare_example.yml` 和 `lore-formats/switchable_example.yml` 提供装备比较和切换描述的示例。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：`{round:N}` 中的 `N` 是格式宽度，实际固定显示两位小数。`lineRequire` 根据键名是否带 `*` 检查物品数据或原配置路径，再按 `ANY`、`ALL`、`NOT` 或 `NOT_ALL` 决定是否显示。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: tent-double-peak
---

# 描述格式

你可以用同一套描述格式为多件物品添加统一的类型、品级和等级行。先把下面的内容保存为 `plugins/Sertraline/lore-formats/practice.yml`：

```yaml
practice_lore:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - key: "xbuilder:lore"
      content: "{value}"
    - key: "sertraline:type"
      content: "<!i><gray>类型 {type:name}"
    - key: "sertraline:tier"
      content: "<!i><gray>品级 {tier:format}"
    - key: "sertraline:level"
      content: "<!i><gray>等级 {level:name}"
```

把[物品品级示例](item-tier.md)里 `classified_sword` 的 `lore-format: loreformat` 改为 `lore-format: practice_lore`。重载后重新领取这把剑，检查原描述和三项分类信息是否按顺序显示。

## 自定义各行

- 只填 `content` 就能添加一行固定文本。指定 `key` 后，插件按对应数据生成内容。若值是列表，`{value}` 会逐项生成一行。
- `key: "sertraline:vals.note"` 可以读取物品 `vals` 中的 `note`。`key: "*sertraline.vals.note"` 直接读取配置文件中的值。前者用冒号区分配置段，后者用点号分隔 YAML 层级。
- 用 `lineRequire` 列出条件键，再用 `lineMode` 选择 `ANY`（至少一个存在）、`ALL`（全部存在）、`NOT`（全部缺失）或 `NOT_ALL`（至少一个缺失）。
- 数值行可用 `{plus}`、`{minus}` 显示符号，用 `{auto}` 显示不带符号的数值，或用 `{round:2}` 舍入。要保留原值可用 `{value}`。有玩家上下文时还可以使用 PlaceholderAPI 占位符和内嵌 Kether。

| `settings` 字段 | 默认值 | 用途 |
| --- | --- | --- |
| `overwrite` | `true` | 覆盖原描述。设为 `false` 时追加 |
| `visual` | `false` | 默认直接写入物品。开启后通过发包显示，需要 PacketEvents 和 `packet.packet-lore` |
| `auto-skip-blank` | `true` | 合并连续空行 |

自带的 `lore-formats/compare_example.yml` 和 `lore-formats/switchable_example.yml` 展示了对比描述和切换描述。

#### LoreSetting 属性 <a href="#loresetting-properties" id="loresetting-properties"></a>

| 属性    | 类型      | 默认值 | 说明                                                                           |
| ----------- | --------- | ------- | ------------------------------------------------------------------------------------- |
| `overwrite` | `Boolean` | `true`  | 为 true 时覆盖原描述；为 false 时追加到原描述                              |
| `visual`    | `Boolean` | `false`  | 为 true 时应用于服务端到客户端的视觉转换；为 false 时应用于服务端物品 |
| `auto-skip-blank` | `Boolean` | `true`  | 如果为 true，则将连续的空行合并为单个空行                        |

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

如果 `sertraline:vals.test` 包含 `[10, 20, 30]`，生成三行，每行都使用占位符替换值。

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

描述格式依次替换行内的数值或分类占位符，再处理玩家上下文与内嵌脚本。

**源码：** [project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt54-101](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/7fbdc299/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt#L54-L101)

#### 内置占位符类型 <a href="#built-in-placeholder-types" id="built-in-placeholder-types"></a>

`performNormalPlaceholders()` 函数根据迭代值处理特殊占位符：

| 占位符 | 行为                                | 示例                           |
| ----------- | --------------------------------------- | --------------------------------- |
| `{plus}`    | 显示 `+` 如果值 > 0，则为空 | `15` → `+`                        |
| `{minus}`   | 值小于 0 时显示 `-`，否则为空 | `-5` → `-`                        |
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

</details>
