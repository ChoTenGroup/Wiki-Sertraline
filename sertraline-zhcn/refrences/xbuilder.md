---
icon: square-x-twitter
---

# XBuilder

[查看完整的物品配置示例](../advanced/item-structures/section-xbuilder.md)

## XItemStack YAML 语法参考

> **来源**: [XItemStack Wiki](https://github.com/CryptoMorin/XSeries/wiki/XItemStack) — Crypto Morin, 最后更新于 2025 年 8 月 29 日
>
> 将完整示例缩进到物品的 `xbuilder` 段下。`Text`、`Boolean` 等类型名称仅用于说明。配置结构以 Sertraline 3.13.2.2 随附的 XItemStack 为准。

***

### 类型说明

| 类型                | 说明                                                                                                                                                          |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Text**            | 字符串，可以是自由文本，也可能仅接受参考文档列出的值。                                                                                              |
| **Boolean**         | `true` 或 `false`。                                                                                                                                                   |
| **Number**          | 非负整数，如 `0`、`1`、`2`；常用于游戏刻（20 tick = 1 秒）。`; min - max` 表示包含端点的范围。 |
| **Decimal**         | 非负小数，如 `0`、`1`、`1.455`；可以使用 `; min - max` 表示范围。                                                                        |
| \*\*\[\*\***Type]** | 指定类型的列表（YAML 列表格式）。                                                                                                                         |
| **类型或类型**    | 该选项接受多种类型。                                                                                                                                   |

**条件选项**：只有物品材质满足指定条件时才生效。

**分类**（如 `class Attribute`）定义可复用的字段组；分类名作为类型出现时，在该位置填写对应字段。

**别名**（如 `alias Color: for Text`）对基础类型添加限制，只接受指定格式或值。

***

### 1. 物品基础属性

```yaml
name: Text
```

物品显示名称，支持颜色代码。

***

```yaml
lore: Text or [Text]
```

物品描述文本。

* 单个字符串表示一行描述。
* YAML 字符串列表表示多行描述。
* 字符串中的换行符（`\n`）也会产生换行。

示例：

```yaml
name: "<blue>Cool Sword" # 也可使用传统颜色代码
lore: |
  You can tell from the name
  that this sword is cool
  because that's exactly what it says!
```

***

```yaml
material: Text
```

[Bukkit Material](https://cryptomorin.github.io/Javadocs-Simplifier/api/v1?url=https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Material.html) 标识符，例如 `DIAMOND_SWORD`, `STONE`。

***

```yaml
amount: Number
```

XItemStack 使用正整数表示数量。Sertraline 的发放命令使用命令里的数量，配方使用 `result.count`。

***

```yaml
damage: Decimal
```

已经损耗的耐久值。数值越大，剩余耐久越少。

***

### 2. 玩家头颅

```yaml
skull: Text
```

**所需材质**: `PLAYER_HEAD`

头颅纹理。支持以下格式，按优先级从高到低排列：

1. **Base64** （推荐）——无需处理编码或拼接
2. **URL** — `textures.minecraft.com` 链接
3. **UUID** — 玩家 UUID
4. **名称** — 玩家名称

> 💡 可以从以下网站获取头颅纹理: [minecraft-heads.com/custom-heads](https://minecraft-heads.com/custom-heads), 该网站也支持 Base64 格式。

***

### 3. 物品行为和外观

```yaml
unbreakable: Boolean
```

是否设置为无法损坏（不消耗耐久）。

***

```yaml
custom-model-data: Text or Number or Color or Boolean or CustomModelDataComponent
```

1.14 及以上版本的自定义模型数据，可通过资源包改变物品外观。

***

```yaml
item-model: Text
```

1.21.2 及以上版本的物品模型组件（`minecraft:item_model`），例如 `minecraft:iron_ingot`；其优先级高于 `custom-model-data`。

***

### 4. 附魔

```yaml
enchants:
  enchantment name: level
```

**示例：**

```yaml
enchants:
  ARROW_FIRE: 2
  DURABILITY: 3
```

每个附魔使用 [Bukkit Enchantment](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/enchantments/Enchantment.html) 名称作为键，等级作为值。

***

**存储的附魔**（用于附魔书），材质为 `ENCHANTED_BOOK`:

```yaml
stored-enchants:
  enchantment name: level
```

***

### 5. 物品标记与发光

```yaml
flags: Flag or [Flag]
```

物品标记可隐藏物品提示中的某些属性，参见 [Bukkit ItemFlag](https://cryptomorin.github.io/Javadocs-Simplifier/api/v1?url=https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/inventory/ItemFlag.html).

**示例：**

```yaml
flags: [HIDE_ATTRIBUTES, HIDE_POTION_EFFECTS]
```

***

```yaml
glow: Boolean
```

当未配置 `enchants` 时，可用 `glow: true` 添加发光效果。该写法会添加耐久 I 附魔，并尝试用 `HIDE_ENCHANTS` 隐藏。

***

### 6. 属性

```yaml
attributes:
  AttributeName:     # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html
    name: Text             # https://minecraft.gamepedia.com/Attribute
    amount: Decimal        # 修饰值
    operation: Text        # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html
    slot: Slot             # 属性生效的槽位
```

物品属性控制描述下方显示的数值修饰符，可使用 `HIDE_ATTRIBUTES` 标记隐藏。

`attributes` 下用属性名称作键，无需列表符号。**槽位可选值：** `HEAD`, `CHEST`, `LEGS`, `FEET`, `HAND`, `OFF_HAND`

***

### 7. 按材质生效的选项

以下选项仅在物品材质满足相应条件时可用。

#### 7.1 旗帜

材质为任意旗帜:

```yaml
patterns:
  STRIPE_DOWNLEFT: RED
```

#### 7.2 皮革盔甲与药水

材质为任意皮革盔甲或药水:

```yaml
color: Color
```

#### 7.3 美西螈桶

当材质为 `AXOLOTL_BUCKET`:

```yaml
color: LUCY or WILD or GOLD or CYAN or BLUE
```

#### 7.4 热带鱼桶

当材质为 `TROPICAL_FISH_BUCKET`:

```yaml
color: DyeColor
pattern-color: DyeColor
pattern: TropicalFishPattern
```

#### 7.5 潜影盒

材质为任意潜影盒:

```yaml
contents:
  "0":
    material: PAPER
    amount: 2
```

#### 7.6 刷怪笼

当材质为 `SPAWNER`:

```yaml
spawner: Text    # EntityType: https://hub.spigotmc.org/javadocs/spigot/org/bukkit/entity/EntityType.html
```

#### 7.7 弩

当材质为 `CROSSBOW`:

```yaml
projectiles:
  first:
    material: ARROW    # 已装填的弹药
```

#### 7.8 迷之炖菜

当材质为 `SUSPICIOUS_STEW`:

```yaml
effects: [Potion]
```

#### 7.9 成书

当材质为 `WRITTEN_BOOK`:

```yaml
book:
  author: "Sertraline"
  title: "Field Guide"
  pages: ["Page one", "Page two"]
  generation: ORIGINAL
```

#### 7.10 地图

当材质为 `FILLED_MAP`:

```yaml
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
```

#### 7.11 刷怪蛋

> ⚠️ 仅用于旧版 Minecraft。

当材质为 `SPAWN_EGG`:

```yaml
creature: EntityType
```

#### 7.12 指南针 (1.16+)

当材质为 `COMPASS`（版本高于 1.15）:

```yaml
tracked: Boolean         # whether it tracks a lodestone
lodestone: Location      # 磁石坐标
```

#### 7.13 烟花

当材质为 `FIREWORK_ROCKET`:

```yaml
power: Number           # 飞行高度或持续时间
firework:
  burst:
    flicker: Boolean       # 烟花是否闪烁
    trail: Boolean         # 烟花是否带有轨迹
    colors:
      base: [Color]        # 主要爆炸颜色
      fade: [Color]   # 渐变颜色
    type: Text           # FireworkEffect.Type: https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/FireworkEffect.Type.html
```

***

### 8. 辅助结构

#### 8.1 CustomModelDataComponent

```yaml
strings: [Text]
floats: [Number]
colors: [Color]
flags: [Boolean]
```

#### 8.2 附魔（Enchant）

```yaml
enchantment name: Number    # 附魔名称 → 等级
```

#### 8.3 属性（Attribute）

```yaml
AttributeName:             # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html
  name: Text
  amount: Decimal            # 修饰值
  operation: Text          # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html
  slot: Slot               # HEAD, CHEST, LEGS, FEET, HAND, OFF_HAND
```

#### 8.4 位置（Location）

```yaml
world: Text
x: Decimal
y: Decimal
z: Decimal
```

#### 8.5 颜色（Color）

```yaml
# Format: "red, green, blue" or "hex color"
# Examples:
#   "255, 0, 0"
#   "#FF0000"
```

#### 8.6 染料颜色（DyeColor）

```
WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK,
GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK
```

#### 8.7 热带鱼花纹（TropicalFishPattern）

```
KOB, SUNSTREAK, SNOOPER, DASHER, BRINELY, SPOTTY,
FLOPPER, STRIPEY, GLITTER, BLOCKFISH, BETTY, CLAYFISH
```

#### 8.8 槽位（Slot）

```
HEAD, CHEST, LEGS, FEET, HAND, OFF_HAND
```

***

### 9. 完整示例

#### 基础剑

```yaml
name: "&2Cool Sword"
lore: |
  You can tell from the name
  that this sword is cool
  because that's exactly what it says!
material: DIAMOND_SWORD
enchants:
  UNBREAKING: 3
  SHARPNESS: 5
```

#### 玩家头颅

```yaml
material: PLAYER_HEAD
skull: "Notch"
name: "&eCustom Head"
```

#### 烟花火箭

```yaml
material: FIREWORK_ROCKET
name: "&dBoom Stick"
power: 3
firework:
  burst:
    type: BALL_LARGE
    flicker: true
    trail: true
    colors:
      base: ["#FF0000", "#00FF00"]
      fade: ["#0000FF"]
```

#### 染色皮革盔甲

```yaml
material: LEATHER_CHESTPLATE
name: "&6Royal Vest"
color: "255, 215, 0"
unbreakable: true
attributes:
  ARMOR:
    name: "generic.armor"
    amount: 8
    operation: ADD_NUMBER
    slot: CHEST
```

***

> **参考链接：**
>
> * [Bukkit Material](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Material.html)
> * [Bukkit ItemFlag](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/inventory/ItemFlag.html)
> * [Bukkit Attribute](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html)
> * [Bukkit AttributeModifier.Operation](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html)
> * [Bukkit EntityType](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/entity/EntityType.html)
> * [Bukkit FireworkEffect.Type](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/FireworkEffect.Type.html)
> * [Minecraft Wiki: Attribute](https://minecraft.gamepedia.com/Attribute)
> * [Minecraft Wiki: Tick](https://minecraft.fandom.com/wiki/Tick)
> * [Minecraft Heads (Head Textures)](https://minecraft-heads.com/custom-heads)



XBuilder 支持的配置格式可参见[此处](https://github.com/CryptoMorin/XSeries/wiki/XItemStack).

