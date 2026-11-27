---
icon: cards-blank
---

# 模板

多件物品共用材质、描述或属性时，可以把重复的配置放进模板。更新模板并重载后，重新领取物品就能看到变化。

## 用一份模板创建两把剑

先在 `plugins/Sertraline/templates/practice.yml` 定义一个模板。顶层名称 `practice:blade` 是模板 ID。

```yaml
practice:blade:
  xbuilder:
    material: "%{material?:IRON_SWORD}"
    name: "%{name?:<!i><green>练习剑}"
    unbreakable: true
    lore:
      - "<!i><gray>练习装备"
```

再在 `plugins/Sertraline/workspace/practice_blades.yml` 创建两件物品。

```yaml
practice_iron_blade:
  $template: practice:blade

practice_gold_blade:
  $template:
    use: practice:blade
    arguments:
      material: GOLDEN_SWORD
      name: "<!i><gold>黄金练习剑"
  xbuilder:
    lore:
      - "<!i><gray>活动奖励"
```

执行 `/sertraline reload`，再分别用 `/sertralineitem give practice_iron_blade` 和 `/sertralineitem give practice_gold_blade` 领取。铁剑使用模板的默认名称与材质，金剑使用 `arguments` 提供的名称与材质。两把剑都无法损坏。金剑的描述还会在模板原有描述后多出一行。

## 调整参数和覆盖方式

* `%{material}` 会读取 `arguments` 中的 `material`，`%{material?:IRON_SWORD}` 在参数缺失时使用 `IRON_SWORD`。文本、数字和布尔参数都可用于替换。
* `%{_ID}` 是当前文件的顶层条目 ID。例如金剑使用模板时，该值为 `practice_gold_blade`。
* 如果只想替换这个条目的描述，把 `$list-mode: replace` 放在与 `$template` 同级的位置。条目自己的同名字段优先，列表默认追加。
* 模板可通过 `$template` 引用其他模板。每个模板使用唯一名称，引用带命名空间的模板时写完整 ID。你也可以在支持模板的目录中复用它，例如 `recipes/` 或 `crafting-stations/`。

## 在填入模板前处理参数

可以在 `arguments` 里使用 `$t`。例如，把金剑原有的 `arguments` 替换为下面的配置，重载后它的名称会变成大写。

```yaml
arguments:
  name:
    $t:
      type: upper
      value: "gold practice blade"
  material: GOLDEN_SWORD
```

`$t` 还支持 `lower`、`when`、`condition` 和 `self_increase`，也可用 `kether`、`js`、`graaljs`、`fluxon` 或 `jexl` 计算结果。转换器按声明顺序求值，先写它需要引用的参数。`self_increase` 在每次模板展开时分配一个数值，同一次展开引用多次时值相同，重载会重置序列。

对于带 `%{damage}` 参数的模板，可在 `arguments` 下写成这样。这里 `ceil` 把 `10.2` 向上取整为 `11`，另有 `floor` 和 `round`。

```yaml
  damage:
    $c:
      type: int
      value: "10.2"
      round: ceil
```

`$c` 的输入依次取自同条参数上的 `$t` 运算结果、`$c.value` 或外层同名参数，转换失败可用 `fallback`。可用类型包括 `int`、`double` 和 `string`。在模板值的引号内使用这些参数时，下游配置仍会按各字段的规则解析。

完整示例见随附的 `templates/basic_gear.yml` 和 `workspace/default/example_templated_items.yml`。加载时报模板名重复、参数未使用或占位符未解析时，可从重载日志定位具体文件。开发者可以查阅 [SertralineAPI 的模板接口](../can-kao/sertralineapi.md)。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：`$t` 按声明顺序执行，可读取此前声明的参数和 `_ID`；`$c` 转换同一参数的 `$t` 结果、显式 `value` 或外层模板继承的参数。示例符合当前求值顺序。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: cards-blank
---

# 模板

模板可以让多件物品复用配置。插件自带的 `templates/basic_gear.yml` 定义了 `basic_melee_weapon`；你可以在 `workspace` 下新建物品文件，使用它来创建一把剑：

```yaml
my_template_sword:
  $template:
    use: basic_melee_weapon
    arguments:
      material: IRON_SWORD
      name: "<!i><green>模板练习剑"
      damage: 10
      speed: 1.6
  sertraline:
    tier: C
```

保存后执行 `/sertraline reload`，再用 `/sertraline item give my_template_sword` 领取。`use` 是模板 ID；`arguments` 只会替换模板中对应的 `%{参数名}`，因此传入模板未使用的参数不会改变物品。模板默认值及更多可运行示例请看 `templates/basic_gear.yml` 和 `workspace/default/example_templated_items.yml`。

## 模板系统

Sertraline 的模板系统允许使用可重用的 YAML 片段，并支持参数替换、值转换（`$t`），以及类型转换（`$c`）。模板从 `templates/` 文件夹中，并且可以使用以下方式从任何配置文件中引用： `$template` 指令引用。

***

### 定义模板

将 YAML 文件放置在 `templates/` 目录。每个顶级键都是一个模板名称。

```yaml
# templates/example.yml
my_weapon_template:
  xbuilder:
    material: "%{material?:IRON_SWORD}"
    name: "%{name?:Unnamed Weapon}"
    lore:
      - "%{lore_line?:}"
    unbreakable: true
    attributes:
      ATTACK_DAMAGE:
        amount: "%{damage?:10}"
        operation: ADD_NUMBER
        slot: HAND
      ATTACK_SPEED:
        amount: "%{speed?:1.6}"
        operation: ADD_NUMBER
        slot: HAND
```

模板可以引用其他模板（链式）：

```yaml
# templates/tiers.yml
iron_melee:
  $template:
    use: my_weapon_template
    arguments:
      material: IRON_SWORD
      damage: 8
      speed: 1.6
      name: Iron Sword
```

***

### 使用模板

在任何配置文件（物品、配方等）中，使用 `$template` 指令：

```yaml
my_custom_sword:
  $template:
    use: iron_melee
    arguments:
      name: My Custom Sword
  sertraline:
    tier: C
```

同级物品字段会递归合并到模板结果中，并覆盖同名字段。列表默认追加。把 `$list-mode: replace` 写在 `$template` 同一级可替换重叠的列表。

#### 自动参数 `_ID`

每个根条目都会自动获得 `_ID`，其值为该条目的 ID：

```yaml
flame_sword:
  $template:
    use: my_weapon_template
    arguments:
      material: NETHERITE_SWORD
      name: "Flame Sword of %{_ID}"
# _ID = "flame_sword" → name becomes "Flame Sword of flame_sword"
```

***

### 参数替换

#### 语法

| 格式             | 说明                                              |
| ------------------- | -------------------------------------------------------- |
| `%{param}`          | 替换为参数 `param`. 缺失时保留原样。  |
| `%{param?:default}` | 替换为参数 `param`, 缺失时使用 `default`。 |

#### 默认值

```yaml
name: "%{name?:Unnamed Weapon}"     # default: "Unnamed Weapon"
color: "%{color?:WHITE}"            # default: "WHITE"
lore: "%{lore?:}"                   # default: empty string
```

#### 转义序列

| 转义 | 输出       |
| ------ | ------------ |
| `\{`   | 字面值 `{`  |
| `\}`   | 字面值 `}`  |
| `\?:`  | 字面值 `?:` |

***

### 转换器（`$t`）

转换器在替换之前处理参数值。请将它们放在 `$template.arguments` 在任何键下：

```yaml
arguments:
  name:
    $t:
      type: upper
      value: "sword"
```

#### 缩写

```yaml
$t: upper
```

当值为字符串（而非映射）时，其将直接用作类型。

#### `$t: when`

通过查找表映射源值：

```yaml
$t:
  type: when
  source: "%{_ID}"
  when:
    wild_guardian: "&2Wild Guardian"
    forest_guardian: "&aForest Guardian"
  fallback: "Unknown"
```

#### `$t: upper` / `$t: lower`

大小写转换：

```yaml
$t:
  type: upper
  value: "%{name}"
  locale: en             # 可选语言区域
```

```yaml
$t:
  type: lower
  value: "HELLO"
```

别名： `uppercase`, `to_upper_case` / `lowercase`, `to_lower_case`

#### `$t: condition`

布尔条件选择：

```yaml
$t:
  type: condition
  condition: "%{enchanted?:false}"   # true/false/yes/no/1/0
  "true": "&dEnchanted Sword"
  "false": "&5Normal Sword"
```

#### `$t: self_increase`

在每次展开中评估的自动递增计数器：

```yaml
$t:
  type: self_increase
  from: 1
  to: 999
  step: 1
  step_interval: 1
```

* `step_interval`：增量之间的调用次数（例如，2 表示 0、0、1、1……）

#### `$t: kether`

评估 Kether 表达式。 `shell` 价值支持 `%{...}` 评估前的参数替换。

```yaml
$t:
  type: kether
  shell: |
    random 1 to 10
  sandbox: true # 忽略异常
  fallback: 1
  nullfallback: 2
  unitfallback: 3
```

**在 Kether 中使用参数：**

```yaml
arguments:
  min: 5
  max:
    $t:
      type: kether
      shell: |
        math add %{min} 10
      fallback: 15
  bonus_name:
    $t:
      type: kether
      shell: |
        if check "%{_ID}" == "super_sword" then "Legendary" else "Normal"
```

Kether的表达方式是通过 `KetherShell.eval` 以控制台发送者为上下文。使用 `sandbox: false` 抛出异常，而不是回退。

**备用优先事项：** `nullfallback` > `unitfallback` > `fallback` — 与 Kether 返回类型（null、Unit 或异常）进行匹配。

#### `$t: js` / `$t: graaljs` / `$t: fluxon` / `$t: jexl`

使用相同配置形状评估脚本表达式：

```yaml
$t:
  type: js
  shell: Math.max(5, 10)
  fallback: 0
```

```yaml
$t:
  type: jexl
  shell: |
    Math.min(%{min}, %{max})
  fallback: 0
```

***

### 类型转换 (`$c`)

类型转换器将字符串值转换为特定类型，并可选择是否进行四舍五入：

```yaml
arguments:
  damage:
    $c:
      type: int
      round: ceil
```

#### 缩写

```yaml
$c: double
```

#### 类型

| 类型     | 选项                                              | 说明                          |
| -------- | ---------------------------------------------------- | ------------------------------------ |
| `int`    | `round`: `ceil` / `cell`, `round`, `floor`           | 带四舍五入的整数转换     |
| `double` | —                                                    | 双精度浮点转换                    |
| `string` | `case`: `uppercase` / `upper`, `lowercase` / `lower` | 带可选大小写转换的字符串 |

#### 大小写转换（任何类型）

```yaml
$c:
  type: string
  case: uppercase
  locale: en
```

***

### 组合使用 `$t` 与 `$c`

先转换值，再转换类型：

```yaml
arguments:
  damage:
    $t:
      type: when
      source: "%{_ID}"
      when:
        shadow_blade: "15.7"
      fallback: "10"
    $c:
      type: int
      round: ceil
  name:
    $t:
      type: upper
      value: "shadow blade"
    $c:
      type: string
      case: upper
```

***

### 参数解析优先级

当模板串联（A → B → C）时，外部参数优先于内部默认值：

1. `_ID` 始终从根条目 ID注入
2. 从最外层开始的参数 `$template.arguments` 将首先应用
3. 内部模板参数仅适用于外部层尚未提供的键

***

### API（编程访问）

#### 模板访问（通过 `SertralineAPI`)

```kotlin
val api = Sertraline.api()

// Query
val template: Map<String, Any?>? = api.getTemplate("my_weapon_template")
val names: Set<String> = api.getTemplateNames()
val all: Map<String, Map<String, Any?>> = api.getAllTemplates()
val count: Int = api.getTemplateCount()

// Resolve
val resolved = api.resolveTemplate("my_weapon_template", mapOf("name" to "Test"))
```

#### 自定义处理器注册

注册自定义 `$t` 转换器类型：

```kotlin
api.registerTransformer("my_type") { type, config, resolveCtx ->
    // type: "my_type"
    // config: the full $t map
    // resolveCtx: current args including _ID, _ORIGIN
    "transformed value"
}

api.unregisterTransformer("my_type")
```

注册自定义 `$c` 转换器类型：

```kotlin
api.registerConverter("my_format") { type, config, value, resolveCtx ->
    // value: the input string to convert
    "converted value"
}

api.unregisterConverter("my_format")
```

注册自定义参数指令（与 `$t`/`$c`):

```kotlin
api.registerDirective("\$myDirective") { name, config, key, currentArgs, dynamicArgs, resolveCtx ->
    // Process like $t/$c but with full control over currentArgs/dynamicArgs
}

api.unregisterDirective("\$myDirective")
```

功能接口（定义于 `TemplateManager`):

```kotlin
fun interface TransformerProvider {
    fun process(type: String, config: Map<String, Any?>, resolveCtx: Map<String, String>): String
}

fun interface ConverterProvider {
    fun process(type: String, config: Map<String, Any?>, value: String, resolveCtx: Map<String, String>): String
}

fun interface DirectiveProvider {
    fun process(name: String, config: Map<String, Any?>, key: String, currentArgs: MutableMap<String, String>, dynamicArgs: MutableMap<String, () -> String>, resolveCtx: Map<String, String>)
}
```

***

### Kether 集成

#### Kether 操作中的模板参数

模板解析的值将用于物品操作。任何 `%{...}` 在执行前，操作脚本中的  会被替换。

```yaml
my_sword:
  $template:
    use: basic_melee_weapon
    arguments:
      damage: 20
      material: DIAMOND_SWORD
      name: "&bSword of %{_ID}"
      power: 50
  sertraline:
    tier: S
    actions:
      onRightClick:
        - kether: |
            tell inline "&e%{name} &7deals &c%{damage} &7damage!"
            tell inline "&7Power level: &a%{power}"
            if check %{power} > 30 then give %{material} 1
```

#### 在所有脚本类型中使用 `%{...}`

```yaml
actions:
  onConsume:
    - kether: |
        tell inline "&aYou consumed %{name} (+%{heal?:5} HP)"
    - js: |
        player.sendMessage("Healed by " + "%{heal?:5}");
    - jexl: |
        player.sendMessage("Heal value: " + Math.max(%{heal?:5}, 0))
```

#### 在 Kether `$t` 链中使用模板

您可以将 `$t: kether` 与其他转换器：

```yaml
arguments:
  enchant_power:
    $t:
      type: when
      source: "%{_ID}"
      when:
        flame_sword: "5"
        ice_sword: "3"
      fallback: "1"
    $c:
      type: int
  damage:
    $t:
      type: kether
      shell: |
        math multiply %{enchant_power} 4
      fallback: 10
      sandbox: true
```

***

### 其他

模板加载自 `plugins/Sertraline/templates/` 在重新加载期间。如果文件夹不存在，则会自动创建。模板加载在所有其他配置**之前**进行，因此模板可用于物品、配方等。

```
plugins/Sertraline/
├── templates/
│   ├── basic_gear.yml
│   └── material_tiers.yml
├── workspace/
├── recipes/
└── crafting-stations/
```

</details>
