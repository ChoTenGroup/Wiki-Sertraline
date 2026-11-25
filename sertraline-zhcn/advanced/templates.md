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

{% hint style="info" %}
以下参考内容请结合安装版本核对。高级用法请同时对照安装版本自带的模板和示例。
{% endhint %}


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

外部字段 `$template` 合并到解析结果中，覆盖模板默认值。

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
  true: "&dEnchanted Sword"
  false: "&5Normal Sword"
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
