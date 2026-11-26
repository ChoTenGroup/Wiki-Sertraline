---
icon: square-x-twitter
---

# XBuilder

[See complete item examples](../advanced/item-structures/section-xbuilder.md)

## XItemStack YAML Syntax Reference

> **Source**: [XItemStack Wiki](https://github.com/CryptoMorin/XSeries/wiki/XItemStack) — Crypto Morin, last updated on August 29, 2025
>
> **Format**: YAWORD (Yet Another Way Of Reading Documentations) — a hybrid YAML and Ruby syntax format used to write configuration documentation on GitHub in a compact, highlighted way.

***

### Type System

| Type                | Description                                                                                                                                                          |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Text**            | String. Can be free text, or restricted to values in linked references.                                                                                              |
| **Boolean**         | `true` or `false`.                                                                                                                                                   |
| **Number**          | Positive integer (such as `0`, `1`, `2`). Often used to represent game ticks (20 ticks = 1 second). May be followed by `; min - max` to indicate an inclusive range. |
| **Decimal**         | Positive decimal number (such as `0`, `1`, `1.455`, `4.5`). May be followed by `; min - max`.                                                                        |
| \*\*\[\*\***Type]** | A list of the given type (YAML list format).                                                                                                                         |
| **Type or Type**    | The option accepts multiple types.                                                                                                                                   |

**Conditional options**: The following options are only valid when the item material matches the specified conditions.

**Classifications** (such as `class Attribute`) define reusable groups of fields. When a classification name appears as a type, its fields are substituted into that position.

**Aliases** (such as `alias Color: for Text`) are named constraints on a base type—they only accept specific formats or values.

***

### 1. Basic Item Properties

```yaml
name: Text
```

The item's display name. Supports color codes.

***

```yaml
lore: Text or [Text]
```

The item's lore (description text).

* A single string -> one line of lore.
* A YAML string list -> multiple lines of lore.
* Newline characters (`\n`) in the string also produce line breaks.

Example:

```yaml
name: "<blue>Cool Sword" # Legacy color codes can also be used
lore: |
  You can tell from the name
  that this sword is cool
  because that's exactly what it says!
```

***

```yaml
material: Text
```

[Bukkit Material](https://cryptomorin.github.io/Javadocs-Simplifier/api/v1?url=https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Material.html) identifier (such as `DIAMOND_SWORD`, `STONE`).

***

```yaml
amount: Number
```

Use a positive integer. Sertraline give commands use their own amount argument, and recipes use `result.count`.

***

```yaml
damage: Decimal
```

The item's damage value/durability value.

***

### 2. Heads (Player Heads)

```yaml
skull: Text
```

**Required material**: `PLAYER_HEAD`

Head texture. Accepted formats (from highest to lowest priority):

1. **Base64** (recommended) — no encoding or concatenation issues
2. **URL** — `textures.minecraft.com` link
3. **UUID** — player UUID
4. **Name** — player name

> 💡 A great resource for obtaining head textures: [minecraft-heads.com/custom-heads](https://minecraft-heads.com/custom-heads), which also supports Base64 format.

***

### 3. Item Behavior and Appearance

```yaml
unbreakable: Boolean
```

Whether the item has the unbreakable property (prevents durability loss).

***

```yaml
custom-model-data: Text or Number or Color or Boolean or CustomModelDataComponent
```

1.14+ custom model data feature. Used to change the item's appearance through resource packs.

***

```yaml
item-model: Text
```

1.21.2+ item model component (`minecraft:item_model`), used for more granular control over the item model, such as `minecraft:iron_ingot`. Takes priority over the `custom-model-data` setting.

***

### 4. Enchantments

```yaml
enchants:
  enchantment name: level
```

**Example:**

```yaml
enchants:
  ARROW_FIRE: 2
  DURABILITY: 3
```

Each enchantment uses a [Bukkit Enchantment](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/enchantments/Enchantment.html) name as the key and the level as the value.

***

**Stored enchantments** (for enchanted books) — when material is `ENCHANTED_BOOK`:

```yaml
stored-enchants:
  enchantment name: level
```

***

### 5. Item Flags and Glow

```yaml
flags: Flag or [Flag]
```

Item flags are used to hide certain properties in the item tooltip. See [Bukkit ItemFlag](https://cryptomorin.github.io/Javadocs-Simplifier/api/v1?url=https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/inventory/ItemFlag.html).

**Example:**

```yaml
flags: [HIDE_ATTRIBUTES, HIDE_POTION_EFFECTS]
```

***

```yaml
glow: Boolean
```

When `enchants` is absent, `glow: true` adds Unbreaking I for the glint and attempts to hide it with `HIDE_ENCHANTS`.

***

### 6. Attributes

```yaml
attributes:
  AttributeName:     # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html
    name: Text             # https://minecraft.gamepedia.com/Attribute
    amount: Decimal        # modifier value
    operation: Text        # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html
    slot: Slot             # the slot where this attribute takes effect
```

Item attributes control the numeric modifiers displayed in the item description (below the lore). They can be hidden using the `HIDE_ATTRIBUTES` flag.

**Slot values:** `HEAD`, `CHEST`, `LEGS`, `FEET`, `HAND`, `OFF_HAND`

***

### 7. Material-Conditional Options

The following options are only available when the item material matches the specified conditions.

#### 7.1 Banners

When material is any kind of banner:

```yaml
patterns:
  STRIPE_DOWNLEFT: RED
```

#### 7.2 Leather Armor and Potions

When material is any kind of leather armor or potion:

```yaml
color: Color
```

#### 7.3 Axolotl Bucket

When material is `AXOLOTL_BUCKET`:

```yaml
color: LUCY or WILD or GOLD or CYAN or BLUE
```

#### 7.4 Tropical Fish Bucket

When material is `TROPICAL_FISH_BUCKET`:

```yaml
color: DyeColor
pattern-color: DyeColor
pattern: TropicalFishPattern
```

#### 7.5 Shulker Box

When material is any kind of shulker box:

```yaml
contents:
  "0":
    material: PAPER
    amount: 2
```

#### 7.6 Spawner

When material is `SPAWNER`:

```yaml
spawner: Text    # EntityType: https://hub.spigotmc.org/javadocs/spigot/org/bukkit/entity/EntityType.html
```

#### 7.7 Crossbow

When material is `CROSSBOW`:

```yaml
projectiles:
  first:
    material: ARROW    # loaded ammunition
```

#### 7.8 Suspicious Stew

When material is `SUSPICIOUS_STEW`:

```yaml
effects: [Potion]
```

#### 7.9 Written Book

When material is `WRITTEN_BOOK`:

```yaml
book:
  author: "Sertraline"
  title: "Field Guide"
  pages: ["Page one", "Page two"]
  generation: ORIGINAL
```

#### 7.10 Filled Map

When material is `FILLED_MAP`:

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

#### 7.11 Spawn Egg

> ⚠️ Only used in legacy Minecraft versions.

When material is `SPAWN_EGG`:

```yaml
creature: EntityType
```

#### 7.12 Compass (1.16+)

When material is `COMPASS` (and version > 1.15):

```yaml
tracked: Boolean         # whether it tracks a lodestone
lodestone: Location      # lodestone coordinates
```

#### 7.13 Firework

When material is `FIREWORK_ROCKET`:

```yaml
power: Number           # flight height/duration
firework:
  burst:
    flicker: Boolean       # whether the firework flickers
    trail: Boolean         # whether the firework has a trail effect
    colors:
      base: [Color]        # main explosion colors
      fade: [Color]   # fade colors
    type: Text           # FireworkEffect.Type: https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/FireworkEffect.Type.html
```

***

### 8. Helper Structures

#### 8.1 CustomModelDataComponent

```yaml
strings: [Text]
floats: [Number]
colors: [Color]
flags: [Boolean]
```

#### 8.2 Enchant

```yaml
enchantment name: Number    # enchantment name -> level
```

#### 8.3 Attribute

```yaml
AttributeName:             # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html
  name: Text
  amount: Decimal            # modifier value
  operation: Text          # https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html
  slot: Slot               # HEAD, CHEST, LEGS, FEET, HAND, OFF_HAND
```

#### 8.4 Location

```yaml
world: Text
x: Decimal
y: Decimal
z: Decimal
```

#### 8.5 Color

```yaml
# Format: "red, green, blue" or "hex color"
# Examples:
#   "255, 0, 0"
#   "#FF0000"
```

#### 8.6 DyeColor

```
WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK,
GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK
```

#### 8.7 TropicalFishPattern

```
KOB, SUNSTREAK, SNOOPER, DASHER, BRINELY, SPOTTY,
FLOPPER, STRIPEY, GLITTER, BLOCKFISH, BETTY, CLAYFISH
```

#### 8.8 Slot

```
HEAD, CHEST, LEGS, FEET, HAND, OFF_HAND
```

***

### 9. Complete Examples

#### Basic Sword

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

#### Player Head

```yaml
material: PLAYER_HEAD
skull: "Notch"
name: "&eCustom Head"
```

#### Firework Rocket

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

#### Colored Leather Armor

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

> **Reference Links:**
>
> * [Bukkit Material](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/Material.html)
> * [Bukkit ItemFlag](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/inventory/ItemFlag.html)
> * [Bukkit Attribute](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html)
> * [Bukkit AttributeModifier.Operation](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/AttributeModifier.Operation.html)
> * [Bukkit EntityType](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/entity/EntityType.html)
> * [Bukkit FireworkEffect.Type](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/FireworkEffect.Type.html)
> * [Minecraft Wiki: Attribute](https://minecraft.gamepedia.com/Attribute)
> * [Minecraft Wiki: Tick](https://minecraft.fandom.com/wiki/Tick)
> * [Minecraft Heads (Head Textures)](https://minecraft-heads.com/custom-heads) \[file content end]



XBuilder supports formula info in [HERE](https://github.com/CryptoMorin/XSeries/wiki/XItemStack).

