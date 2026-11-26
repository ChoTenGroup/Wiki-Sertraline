---
icon: hammer-war
---

# XBuilder

在 `workspace/` 的物品 ID 下添加 `xbuilder`。复制下方任一个完整物品配置，执行 `/sertraline reload` 后，在游戏内用 `/sertralineitem give <物品ID>` 领取。

## 材质、文本和耐久

```yaml
guide_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>训练用剑"
    lore:
      - "<!i><gray>第一行"
      - "&7第二行"
    damage: 15
    unbreakable: true
    enchants:
      SHARPNESS: 5
      UNBREAKING: 3
    flags: [HIDE_ATTRIBUTES]
    attributes:
      ATTACK_DAMAGE:
        name: "example:guide_damage"
        amount: 10
        operation: ADD_NUMBER
        slot: HAND
```

| 字段 | 怎么填 |
| --- | --- |
| `material` | 填服务端支持的材质，如 `IRON_SWORD`、`PAPER`。其他物品来源见[物品配置段](README.md) |
| `name` | 自定义显示名称。`<!i>` 取消斜体，支持 MiniMessage 和传统 `&` 颜色代码 |
| `item-name` | 在支持此字段的版本上设置物品的基础名称。若另设了 `name`，显示时优先使用 `name` |
| `lore` | 可写一行字符串、多行 YAML 列表，或写成 `lore: \|` 再逐行缩进。动态文本见[物品文本标签](../item-text-tags.md) |
| `damage` | 已消耗的耐久，用整数填写。`0` 表示满耐久 |
| `unbreakable` | 设为 `true` 后不再消耗耐久 |
| `amount` | 设置潜影盒内容等嵌套物品的堆叠数量。发放 Sertraline 物品时用命令的数量参数 |
| `enchants` | 用附魔名作为键，等级作为值，例如 `SHARPNESS: 5` |
| `stored-enchants` | 写法与 `enchants` 相同，用于 `ENCHANTED_BOOK` 上存储的附魔 |
| `flags` | 填当前版本支持的提示标记，如 `[HIDE_ATTRIBUTES, HIDE_ENCHANTS]`。单个标记也可写成字符串。用 `ALL` 一次隐藏支持隐藏的提示项 |
| `glow` | 设为 `true` 以发光，XBuilder 可通过添加并隐藏耐久附魔实现。现代服务端也可以用组件 `minecraft.enchantment_glint_override` |

`attributes` 给装备添加原版属性。每项包含 `amount`、`operation`，还可以填写 `name` 和 `slot`。`ADD_NUMBER` 加固定值，`ADD_SCALAR` 按基础值增加比例，`MULTIPLY_SCALAR_1` 对总值乘以 `1 + amount`。例如 `amount: 0.2` 配合 `ADD_SCALAR` 表示增加基础值的 20%。槽位用 `HAND`、`OFF_HAND`、`HEAD`、`CHEST`、`LEGS` 或 `FEET`。使用 Sertraline 自定义属性时，见[物品属性配置](section-attributes.md)。

## 模型、头颅和附魔书

```yaml
guide_model_number:
  xbuilder:
    material: PAPER
    custom-model-data: 10000

guide_model_lists:
  xbuilder:
    material: PAPER
    custom-model-data:
      floats: [1.5]
      strings: [guide]
      flags: [true]
      colors: ["#FFAA00"]

guide_model_key:
  xbuilder:
    material: DIAMOND
    item-model: "minecraft:iron_ingot"

guide_head:
  xbuilder:
    material: PLAYER_HEAD
    skull: "Notch"

guide_enchanted_book:
  xbuilder:
    material: ENCHANTED_BOOK
    stored-enchants:
      SHARPNESS: 4
```

- `custom-model-data` 支持传统数字写法。有扩展模型组件的版本也支持 `floats`、`strings`、`flags` 和 `colors` 列表，外层可简写为单个数字、普通字符串、颜色或同类列表，例如 `custom-model-data: [1.5, 2.5]`。布尔值写在 `flags` 下，例如 `flags: [true]`。在对象写法中，还可以用 `float: 1.5`、`string: guide`、`color: "#FFAA00"` 和 `flag: true` 分别表示单元素。填写的值要和客户端资源包对应。
- `item-model` 在 1.21.2 及以后的版本选用指定的模型定义。模型定义仍可用 `custom-model-data` 中的值选择具体外观。
- `skull` 可填玩家名、UUID、Minecraft 纹理 URL 或 Base64 纹理。外观需要固定时，可直接提供纹理值。

## 特定材质的字段

以下字段按材质生效。可以整段复制，也可以只复制需要的物品。

```yaml
guide_armor:
  xbuilder:
    material: DIAMOND_CHESTPLATE
    trim:
      material: "minecraft:gold"
      pattern: "minecraft:sentry"

guide_banner:
  xbuilder:
    material: WHITE_BANNER
    patterns:
      STRIPE_DOWNLEFT: RED

guide_leather:
  xbuilder:
    material: LEATHER_CHESTPLATE
    color: "#FFD700"

guide_axolotl:
  xbuilder:
    material: AXOLOTL_BUCKET
    color: CYAN

guide_fish:
  xbuilder:
    material: TROPICAL_FISH_BUCKET
    color: WHITE
    pattern-color: BLUE
    pattern: KOB

guide_box:
  xbuilder:
    material: SHULKER_BOX
    contents:
      "0":
        material: PAPER
        amount: 2
      "8":
        material: EMERALD

guide_spawner:
  xbuilder:
    material: SPAWNER
    spawner: ZOMBIE

guide_crossbow:
  xbuilder:
    material: CROSSBOW
    projectiles:
      first:
        material: ARROW

guide_potion:
  xbuilder:
    material: POTION
    base-type: WATER
    color: "51, 153, 255"
    effects:
      - "SPEED,30,2"

guide_stew:
  xbuilder:
    material: SUSPICIOUS_STEW
    effects:
      - "NIGHT_VISION,5,1"
```

- 盔甲纹饰的 `material` 和 `pattern` 填带命名空间的 ID。旗帜 `patterns` 用图案名称作为键、染料颜色作为值。
- 皮革和药水的 `color` 可填带引号的 RGB 字符串或十六进制颜色。美西螈填写 `LUCY`、`WILD`、`GOLD`、`CYAN` 或 `BLUE`。热带鱼的 `color` 与 `pattern-color` 填染料名，`pattern` 填鱼的花纹。
- 潜影盒 `contents` 使用带引号的槽位号 `"0"` 到 `"26"`。弩的 `projectiles` 使用自定名称，每项下再填写投射物的 XBuilder 字段。
- `spawner` 填实体类型。旧版共用 `SPAWN_EGG` 材质时可写 `creature: ZOMBIE`，现代版直接用 `ZOMBIE_SPAWN_EGG` 材质。
- 药水 `base-type` 选择原版基底类型，例如 `WATER` 或 `SWIFTNESS`。药水和谜之炖菜都能用 `effects` 加自定义效果，每项按“效果名、秒数、等级”书写。`"SPEED,30,2"` 表示 30 秒的速度 II。

```yaml
guide_book:
  xbuilder:
    material: WRITTEN_BOOK
    book:
      author: "Sertraline"
      title: "Field Guide"
      generation: ORIGINAL
      pages:
        - "Page one"
        - "Page two"

guide_map:
  xbuilder:
    material: FILLED_MAP
    map:
      scaling: false
      location: "Spawn"
      color: "#FFAA00"
      view:
        world: world
        scale: NORMAL
        center: {x: 0, z: 0}
        locked: false
        tracking-position: true
        unlimited-tracking: false

guide_compass:
  xbuilder:
    material: COMPASS
    tracked: true
    lodestone: {world: world, x: 0, y: 64, z: 0}

guide_firework:
  xbuilder:
    material: FIREWORK_ROCKET
    power: 1
    firework:
      burst:
        type: BURST
        flicker: true
        trail: true
        colors:
          base: ["#FF0000", "#FFAA00"]
          fade: ["#FFFFFF"]
```

- 成书的作者、标题、页和版本分别填在 `book` 下的 `author`、`title`、`pages`、`generation`。版本可用 `ORIGINAL`、`COPY_OF_ORIGINAL`、`COPY_OF_COPY` 或 `TATTERED`。
- `map.location` 是地点标签，地图坐标写在 `map.view.center` 下。`view.scale` 可填 `CLOSEST`、`CLOSE`、`NORMAL`、`FAR` 或 `FARTHEST`。使用地图或磁石示例时，把 `world` 换成服务端已加载的世界名。
- `tracked` 控制是否跟踪磁石，`lodestone` 下填世界和 `x`、`y`、`z` 坐标。
- `power` 控制烟花飞行时间。在 `firework` 下可用不同名称增加多组效果。形状可填 `BALL`、`BALL_LARGE`、`STAR`、`BURST` 或 `CREEPER`。

其他原版功能和版本相关写法见 [Minecraft 数据组件](section-minecraft-components.md)。
