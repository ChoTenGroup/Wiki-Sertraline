---
icon: hammer-war
---

# XBuilder

Add `xbuilder` below an item ID in a `workspace/` file. Copy a complete example, run `/sertraline reload`, then run `/sertralineitem give <item-id>` in game.

## Base item, text and durability

```yaml
guide_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>Guide Sword"
    lore:
      - "<!i><gray>Line one"
      - "&7Line two"
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

| Field | How to use it |
| --- | --- |
| `material` | Choose a material supported by your server, such as `IRON_SWORD` or `PAPER`. See [Item Sections](README.md) for other item sources |
| `name` | Custom display name. `<!i>` removes italic styling. MiniMessage and legacy `&` color codes are supported |
| `item-name` | On versions with this item-meta field, sets the base item name. An explicit `name` takes display priority |
| `lore` | A string for one line, a YAML list for separate lines, or `lore: \|` followed by indented lines. See [item text tags](../item-text-tags.md) for dynamic text |
| `damage` | Durability already used, as an integer. Use `0` for full durability |
| `unbreakable` | Set to `true` to prevent further durability loss |
| `amount` | Stack count for nested XBuilder items, such as shulker contents. Giving a Sertraline item uses the command's quantity |
| `enchants` | Map enchantment names to levels, for example `SHARPNESS: 5` |
| `stored-enchants` | Same shape for enchantments stored in an `ENCHANTED_BOOK` |
| `flags` | List of tooltip flags supported by your version, such as `[HIDE_ATTRIBUTES, HIDE_ENCHANTS]`. A single flag string also works. Use `ALL` to hide all supported tooltip details |
| `glow` | Set to `true` for a glint. XBuilder can add a hidden Unbreaking enchantment to produce it. For a component-only glint on modern servers, use `minecraft.enchantment_glint_override` |

`attributes` adds vanilla equipment modifiers. Each entry takes `amount`, `operation` and optionally `name` and `slot`. Use `ADD_NUMBER` for a fixed amount, `ADD_SCALAR` for a fraction of the base, and `MULTIPLY_SCALAR_1` for a total multiplier. For example, `amount: 0.2` with `ADD_SCALAR` adds 20% of the base. Use `HAND`, `OFF_HAND`, `HEAD`, `CHEST`, `LEGS` or `FEET` for the equipment slot. For attributes defined by Sertraline, use [item attribute sections](section-attributes.md).

## Models, heads and enchanted books

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

- `custom-model-data` takes a number on servers that support legacy numeric model data. On servers with the expanded model component it can take `floats`, `strings`, `flags` and `colors` lists. The expanded syntax also accepts a single number, plain string or color, or a list of one type such as `custom-model-data: [1.5, 2.5]`. Put booleans under `flags`, for example `flags: [true]`. In the object form, `float: 1.5`, `string: guide`, `color: "#FFAA00"` and `flag: true` each set a single entry. Match the data to your resource pack.
- `item-model` selects a namespaced model on 1.21.2 and later. It selects the model definition, which can in turn use model data as a predicate.
- `skull` accepts a player name, UUID, Minecraft texture URL or Base64 texture. For a fixed head texture, supply the texture value directly.

## Fields for particular materials

Use the matching material for each set of fields.

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

- For armor trim, use namespaced material and pattern IDs. For banners, map pattern names to dye names.
- `color` accepts a quoted RGB string or hex for leather and potions. For axolotls use `LUCY`, `WILD`, `GOLD`, `CYAN` or `BLUE`. Fish use dye names for `color` and `pattern-color` and a tropical fish pattern for `pattern`.
- Shulker `contents` uses quoted slot numbers from `"0"` to `"26"`. Crossbow `projectiles` uses named entries with nested XBuilder fields.
- Set `spawner` to the entity type. On older servers with the shared `SPAWN_EGG` material, specify `creature: ZOMBIE`. On modern servers, use `ZOMBIE_SPAWN_EGG` as the material.
- For potions, `base-type` chooses the vanilla base, such as `WATER` or `SWIFTNESS`. `effects` adds custom effects to potions or suspicious stew. The comma-separated fields are effect ID, duration in seconds and one-based level. `"SPEED,30,2"` gives Speed II for 30 seconds.

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

- Put a written book's `author`, `title`, `pages` and `generation` under `book`. Generation can be `ORIGINAL`, `COPY_OF_ORIGINAL`, `COPY_OF_COPY` or `TATTERED`.
- `map.location` is a label. Set coordinates in `map.view.center`. For `view.scale` use `CLOSEST`, `CLOSE`, `NORMAL`, `FAR` or `FARTHEST`. Replace `world` with a loaded world name when setting a map or lodestone.
- `tracked` controls lodestone tracking. `lodestone` needs `world`, `x`, `y` and `z`.
- `power` sets firework flight time. Add named entries under `firework` to combine effects. Types are `BALL`, `BALL_LARGE`, `STAR`, `BURST` and `CREEPER`.

For other vanilla features and version-specific structures, see [Minecraft data components](section-minecraft-components.md).
