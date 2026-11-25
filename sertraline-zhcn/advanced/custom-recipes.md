---
icon: wrench
---

# 自定义配方

下面先看默认配置中的配方。确认 `workspace/default/default_misc.yml` 中存在 `star_of_recall`，然后在 `plugins/Sertraline/recipes/sample.yml` 中找到 `star_from_vanilla`：

```yaml
star_from_vanilla:
  provider: SIMPLE
  format: SHAPED
  type: CRAFTING
  pattern:
    - "PPP"
    - "PGP"
    - "PPP"
  ingredients:
    P: "minecraft:paper"
    G: "minecraft:gold_ingot"
  result:
    id: sertraline:star_of_recall
    count: 1
```

重载后，在工作台将一块金锭放在中央，周围放八张纸。`pattern` 表示摆放形状，`ingredients` 将字符对应到材料，`result` 是产物。`minecraft:` 表示原版物品，`sertraline:` 表示 Sertraline 物品；显式前缀可以避免同名歧义。这是插件自带的配方；如果想修改它，请直接编辑该条目，避免再创建一个相同配方。

{% hint style="info" %}
以下为尚待翻译和逐项核对的原参考内容。配置请优先参照安装版本自带的 `recipes/sample.yml`；其中 `COMPLEX` 示例被注释并标为尚未支持。
{% endhint %}


Custom Recipes Let you can customize recipes for your Sertraline Item!

## Recipe System

### Overview

The recipe system registers custom Minecraft recipes using vanilla mechanics — crafting table, furnace, blast furnace, smoker, campfire, stonecutter, and smithing table. It does **not** use the custom GUI from the crafting station system.

Key features:

* **8 recipe types**: Crafting (shaped/shapeless), Furnace, Blasting, Smoking, Campfire, Stonecutting, Smithing Transform, Smithing Trim
* **Dual provider modes**: `SIMPLE` (Bukkit API) or `COMPLEX` (direct NMS injection)
* **Multi-source ingredients**: Sertraline custom items, vanilla items, Bukkit tags, external plugin items
* **Post-craft functions**: Kether scripts, JavaScript, or console commands
* **Player-aware results**: Sertraline custom item results are rebuilt with player context on preview

***

### File Location

Configuration files are placed under `recipes/` in the plugin data folder:

```
plugins/Sertraline/recipes/
  ├── example.yml
  └── ... (any filename, supports yaml/yml/toml/json)
```

On first startup (if the folder does not exist), `recipes/sample.yml` is auto-extracted as a reference.

***

### Configuration Structure

```yaml
<recipe_id>:
  provider: SIMPLE              # RecipeProvider: SIMPLE (Bukkit API) or COMPLEX (NMS injection)
  type: CRAFTING                # RecipeType (see below)
  format: SHAPED                # RecipeFormat: SHAPED or SHAPELESS (CRAFTING only)

  # --- Crafting (SHAPED) ---
  pattern:                      # 1-3 rows, each 1-3 chars
    - "SSS"
    - "SWS"
    - "SSS"
  ingredients:                  # Maps char → ingredient
    S: "sertraline:star_of_recall"
    W: "sertraline:sword_of_skyline"

  # --- Crafting (SHAPELESS) ---
  ingredients:                  # List of ingredients
    - "sertraline:doe_card"
    - "minecraft:diamond"

  # --- Cooking (FURNACE / BLASTING / SMOKING / CAMPFIRE) ---
  ingredient: "sertraline:doe_card"  # Single ingredient
  experience: 0.5               # XP granted (float)
  cookingTime: 200              # Ticks (200 = 10s)

  # --- STONECUTTING ---
  ingredient: "sertraline:ultimate_pickaxe"

  # --- SMITHING_TRANSFORM / SMITHING_TRIM ---
  template: "sertraline:star_of_recall"
  base: "sertraline:sword_of_skyline"
  addition: "sertraline:blade_of_mythic"

  # --- Result ---
  result:
    id: sertraline:sword_of_eternal_frozen   # Output item ID
    count: 1                                  # Output amount
    function:                                 # Post-craft functions (optional)
      - type: kether
        kether: tell "&aCraft successful!"

  # --- Input Options (optional) ---
  input-options:
    data-filter:
      - "sertraline_data"
      - "sertraline_revision"
    components:
      mode: REMOVE              # REMOVE or KEEP
      elements:
        - "minecraft:enchantments"

  # --- Optional ---
  group: "my_group"             # Recipe book group
  showNotification: true        # Show advancement-style notification (CRAFTING only)
```

***

### Recipe Types

| Type                 | Station        | Description                                                                  |
| -------------------- | -------------- | ---------------------------------------------------------------------------- |
| `CRAFTING`           | Crafting Table | Shaped (grid pattern) or shapeless. Requires `format: SHAPED` or `SHAPELESS` |
| `FURNACE`            | Furnace        | Smelting with XP. `cookingTime` in ticks (20 = 1s)                           |
| `BLASTING`           | Blast Furnace  | Faster smelting. `cookingTime` in ticks                                      |
| `SMOKING`            | Smoker         | Food smelting. `cookingTime` in ticks                                        |
| `CAMPFIRE`           | Campfire       | Slow cooking. `cookingTime` in ticks                                         |
| `STONECUTTING`       | Stonecutter    | 1 ingredient → 1 result, no XP                                               |
| `SMITHING_TRANSFORM` | Smithing Table | Template + Base + Addition → Result                                          |
| `SMITHING_TRIM`      | Smithing Table | Template + Base + Addition → Trimmed item                                    |

***

### Providers

#### SIMPLE (Bukkit API)

Default mode. Registers recipes via `Bukkit.addRecipe()`. Works with all standard Bukkit recipe mechanics and is compatible with other plugins that interact with Bukkit's recipe manager.

#### COMPLEX (NMS Injection)

Injects recipes directly into Minecraft's native `RecipeManager.recipes` map via reflection. Use when:

* Sertraline custom items as ingredients need precise NBT/component matching
* Bukkit API matching is insufficient for custom item data
* Need to bypass other plugins' Bukkit recipe interception

**Note:** COMPLEX provider examples are commented out in sample.yml. Use at your own risk.

***

### Ingredients

#### Formats

**1. String format (simple)**

```yaml
ingredient: "sertraline:doe_card"
```

**2. Object format (with amount)**

```yaml
ingredient:
  itemId: "sertraline:doe_card"
  amount: 2
```

**3. Tag format**

```yaml
ingredient: "#minecraft:planks"     # Bukkit tag
ingredient: "tag:planks"            # Shorthand (assumes minecraft namespace)
```

Tag object format:

```yaml
ingredient:
  tagId: "#minecraft:planks"
  amount: 2
```

**4. Multi-choice (OR logic)**

```yaml
ingredient:
  options:
    - "sertraline:sword_of_skyline"
    - "sertraline:sword_of_counter"
    - "sertraline:sword_of_soulbound"
```

#### Item Sources

| Source            | Identifier             | Example                       |
| ----------------- | ---------------------- | ----------------------------- |
| Vanilla Minecraft | `minecraft:<material>` | `minecraft:diamond`           |
| Sertraline custom | `sertraline:<id>`      | `sertraline:doe_card`         |
| ItemsAdder        | `itemsadder:<id>`      | `itemsadder:ruby`             |
| Oraxen            | `oraxen:<id>`          | `oraxen:sapphire`             |
| Nexo              | `nexo:<id>`            | `nexo:mythril_ingot`          |
| MythicMobs        | `mythicmobs:<id>`      | `mythicmobs:some_item`        |
| MMOItems          | `mmoitems:<type>/<id>` | `mmoitems:SWORD/flame_katana` |

Omitting the source defaults to `minecraft:`.

#### Resolution Logic

| Ingredient Type                         | Resolves To                              | Matching                                   |
| --------------------------------------- | ---------------------------------------- | ------------------------------------------ |
| Vanilla item (`minecraft:stone`)        | `MaterialChoice`                         | Matches by material                        |
| Sertraline item (`sertraline:doe_card`) | `ExactChoice`                            | Matches by exact ItemStack (including NBT) |
| External plugin item                    | `ExactChoice`                            | Matches by exact ItemStack                 |
| Tag (`#minecraft:planks`)               | `MaterialChoice(Tag)`                    | Matches any material in tag                |
| Choice (`options: [...]`)               | Merged `ExactChoice` or `MaterialChoice` | Matches any option (OR logic)              |

***

### Result

#### Fields

| Field      | Type   | Required | Description                                                                      |
| ---------- | ------ | -------- | -------------------------------------------------------------------------------- |
| `id`       | String | Yes      | Output item ID (e.g., `sertraline:sword_of_eternal_frozen`, `minecraft:diamond`) |
| `count`    | Int    | No       | Output amount (default: 1)                                                       |
| `function` | List   | No       | Post-craft function list (see below)                                             |

#### Post-Craft Functions

Executed when the player takes the result from the crafting table. Three types:

```yaml
result:
  id: sertraline:sword_of_eternal_frozen
  count: 1
  function:
    - type: kether
      kether: tell "&aCraft successful! Obtained Eternal Frozen Sword."
    - type: javascript
      script: player.sendMessage("§aCraft successful!")
    - type: command
      command: "say %player% crafted a legendary sword!"
```

**Function Types**

| Type                | Field                | Description                                           |
| ------------------- | -------------------- | ----------------------------------------------------- |
| `kether`            | `kether: "<script>"` | Executes a Kether script with the player as sender    |
| `javascript` / `js` | `script: "<js>"`     | Executes a JavaScript snippet                         |
| `command`           | `command: "<cmd>"`   | Executes a console command (`%player%` → player name) |

Short form (just the script string, defaults to Kether):

```yaml
function:
  - tell "&aSuccess!"
```

***

### Input Options

Controls how input item data is handled during crafting:

```yaml
input-options:
  data-filter:
    - "sertraline_data"
    - "sertraline_revision"
  components:
    mode: REMOVE              # REMOVE or KEEP
    elements:
      - "minecraft:enchantments"
```

#### Fields

| Field                 | Type               | Description                                                             |
| --------------------- | ------------------ | ----------------------------------------------------------------------- |
| `data-filter`         | List\<String>      | NBT data keys to filter (REMOVE or KEEP depending on `components.mode`) |
| `components.mode`     | `REMOVE` or `KEEP` | Whether to remove or keep listed components                             |
| `components.elements` | List\<String>      | Data component namespaces (e.g., `minecraft:enchantments`)              |

***

### Player-Aware Result Preview

When a player opens a crafting table with a Sertraline recipe, the `onPrepareCraft` event listener rebuilds the result item with the player's context — replacing placeholders, applying player-specific variables, and updating dynamic lore.

This means:

* Sertraline custom items as results show correct player-specific data in the preview slot
* Items with `sertraline:vars` are rebuilt per-player
* The preview updates in real-time as the player changes

***

### Commands

| Command              | Description                           |
| -------------------- | ------------------------------------- |
| `/sertraline reload` | Reloads all configs including recipes |

Recipes are re-loaded from disk and re-registered on each reload.

***

### Complete Examples

#### Shaped Crafting — 9 cards → 1 star

```yaml
star_from_cards:
  provider: SIMPLE
  format: SHAPED
  type: CRAFTING

  pattern:
    - "DDD"
    - "DDD"
    - "DDD"
  ingredients:
    D: "sertraline:doe_card"
  result:
    id: sertraline:star_of_recall
    count: 1
```

#### Shapeless — Mix two items

```yaml
mixed_sugar:
  provider: SIMPLE
  format: SHAPELESS
  type: CRAFTING

  ingredients:
    - "sertraline:bagged_zero_calorie_sugar"
    - "sertraline:bagged_crystal_sugar"
  result:
    id: sertraline:bagged_crystal_sugar
    count: 2
```

#### Furnace — Smelt item with XP

```yaml
smelt_pills_from_cookie:
  provider: SIMPLE
  type: FURNACE

  ingredient: "sertraline:speed_potion_cookie"
  result:
    id: sertraline:sertraline_pills_14
    count: 1
  experience: 0.5
  cookingTime: 200
```

#### Blast Furnace — Faster smelting

```yaml
blast_pills_from_card:
  provider: SIMPLE
  type: BLASTING

  ingredient: "sertraline:doe_card"
  result:
    id: sertraline:sertraline_pills_42
    count: 1
  experience: 0.5
  cookingTime: 100
```

#### Smithing — Upgrade item

```yaml
mythic_sword_upgrade:
  provider: SIMPLE
  type: SMITHING_TRANSFORM

  template: "sertraline:star_of_recall"
  base: "sertraline:sword_of_skyline"
  addition: "sertraline:blade_of_mythic"
  result:
    id: sertraline:test_choten_sword
    count: 1
```

#### Stonecutter — 1 input → multiple output

```yaml
cut_pickaxe_to_wands:
  provider: SIMPLE
  type: STONECUTTING

  ingredient: "sertraline:ultimate_pickaxe"
  result:
    id: sertraline:block_wand
    count: 2
```

#### Multi-choice ingredient (OR) + command function

```yaml
any_sword_to_star:
  provider: SIMPLE
  format: SHAPELESS
  type: CRAFTING

  ingredients:
    - options:
        - "sertraline:sword_of_skyline"
        - "sertraline:sword_of_counter"
        - "sertraline:sword_of_soulbound"
  result:
    id: sertraline:star_of_recall
    count: 1
    function:
      - type: command
        command: "say %player% recycled a sword!"
```

***

### Comparison: Recipe System vs. Crafting Station System

| Aspect                | Recipe System                                     | Crafting Station System               |
| --------------------- | ------------------------------------------------- | ------------------------------------- |
| **Config folder**     | `recipes/`                                        | `crafting-stations/`                  |
| **UI**                | Vanilla Minecraft (crafting table, furnace, etc.) | Custom GUI (6-row Chest)              |
| **Crafting time**     | Instant                                           | Configurable timed (seconds)          |
| **Conditions**        | None (anyone can use if ingredients match)        | Kether/Scaled/Permission checks       |
| **Lifecycle scripts** | Post-craft only (via `result.function`)           | onStart/onCancel/onClaim agents       |
| **Persistence**       | None                                              | DataContainer (survives quit/restart) |
| **Pagination**        | N/A (vanilla recipe book)                         | Built-in page browsing                |
| **Item sources**      | Sertraline, vanilla, external plugins             | Same (via OmniItem)                   |
| **Use case**          | Standard Minecraft crafting                       | Custom timed progression systems      |

