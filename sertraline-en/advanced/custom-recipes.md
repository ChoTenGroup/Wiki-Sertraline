---
icon: wrench
---

# Custom Recipes

Create files in `plugins/Sertraline/recipes/` to let players make items in vanilla workstations such as crafting tables and furnaces. Custom recipes are available in modern builds. Use a [crafting station](crafting-stations.md) for a separate menu with inventory-based ingredients and a custom timer.

## Try a bundled recipe

Open `recipes/sample.yml` and find `star_from_vanilla`. Its output is `star_of_recall` from `workspace/default/default_misc.yml`. The example below leaves `provider` at its default.

```yaml
star_from_vanilla:
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
    id: "sertraline:star_of_recall"
    count: 1
```

Place a gold ingot in the center of a crafting table and put one piece of paper in each surrounding slot to make a Star of Recall. Each line in `pattern` is a row in the grid. `ingredients` defines what each letter means. Use a space for an empty slot. Edit the existing entry to change the bundled recipe.

## Add a shapeless recipe

Create `beginner_sword` by following [My First Item](../basics/my-first-item.md). Save the following as `plugins/Sertraline/recipes/practice.yml`.

```yaml
practice_sword_recipe:
  type: CRAFTING
  format: SHAPELESS
  ingredients:
    - "minecraft:iron_ingot"
    - "minecraft:iron_ingot"
    - "minecraft:stick"
  result:
    id: "sertraline:beginner_sword"
    count: 1
```

Run `/sertraline reload`. Put two iron ingots and a stick in three separate crafting slots in any order to make the Beginner Sword. Each vanilla recipe slot uses one item. Leave `amount` out or set it to 1. A higher value fails registration. For repeated ingredients, list the item more than once in a shapeless recipe or repeat its character in a shaped pattern.

## Add a furnace recipe

Add this entry at the top level of `recipes/practice.yml`. Reload, then put mud and fuel in a furnace to make a brick. At normal speed 200 game ticks take about ten seconds. The furnace awards the recipe experience when the output is taken.

```yaml
bake_practice_brick:
  type: FURNACE
  ingredient: "minecraft:mud"
  experience: 0.5
  cookingTime: 200
  result:
    id: "minecraft:brick"
    count: 1
```

## Choose ingredients

| Syntax                                       | What matches                                                                  |
| -------------------------------------------- | ----------------------------------------------------------------------------- |
| `"minecraft:paper"`                          | An item with paper as its base material                                       |
| `"sertraline:beginner_sword"`                | A Beginner Sword with matching data such as name and enchantments             |
| `"itemsadder:ruby"`                          | A custom item supplied by the named item plugin                               |
| `"#minecraft:planks"` or `"tag:planks"`      | A material from the Minecraft planks tag. Quote the form that starts with `#` |
| `{ itemId: "minecraft:paper", exact: true }` | An item with the same data as plain paper                                     |
| `{ tagId: "#minecraft:planks" }`             | A tag written as an object                                                    |

Bare IDs match vanilla materials first and then Sertraline items. For custom ingredients, use items that build the same way each time. Random stats, player-specific names, and item updates can change the data the recipe matches.

For example, use these two `ingredients` in a shapeless recipe. One slot accepts oak or birch planks and the other uses plain paper.

```yaml
ingredients:
  - options:
      - "minecraft:oak_planks"
      - "minecraft:birch_planks"
  - itemId: "minecraft:paper"
    exact: true
```

## Choose a workstation and output

| `type`                                       | Workstation                              | Configuration                                                                           |
| -------------------------------------------- | ---------------------------------------- | --------------------------------------------------------------------------------------- |
| `CRAFTING`                                   | Crafting table                           | `SHAPED` uses a `pattern` and character mapping. `SHAPELESS` uses an ingredient list    |
| `FURNACE`, `BLASTING`, `SMOKING`, `CAMPFIRE` | Furnace, blast furnace, smoker, campfire | `ingredient`, `experience`, `cookingTime`                                               |
| `STONECUTTING`                               | Stonecutter                              | `ingredient`                                                                            |
| `SMITHING_TRANSFORM`                         | Smithing table upgrade                   | `template`, `base`, `addition`                                                          |
| `SMITHING_TRIM`                              | Smithing table armor trim                | `template`, `base`, `addition`. Use a server and provider that support this recipe type |

For example, you can cut one block of obsidian into two blackstone slabs.

```yaml
cut_practice_slabs:
  type: STONECUTTING
  ingredient: "minecraft:obsidian"
  result:
    id: "minecraft:blackstone_slab"
    count: 2
```

Set the output with `result.id` and `result.count`. The count defaults to 1. `SMITHING_TRIM` also requires a valid `result.id`, while the template and armor determine the actual output. The crafting grid builds Sertraline output previews using the current player's context. Set `group` to group crafting, cooking, or stonecutting recipes. On platforms that support it, `showNotification: false` hides the recipe-book notification for crafting recipes.

## Show a message after crafting

Add this `function` under a crafting-table recipe's `result`, alongside `id`. Reload and take the output to see the message.

```yaml
function:
  - type: kether
    kether: tell "Crafting complete!"
```

You can use `type: command` with `command: "say %player% completed a recipe!"`. This runs as the console and substitutes the player's name for `%player%`. The `javascript` or `js` type takes a `script` field. The recipe JavaScript in this version runs without an injected `player` variable.

## Providers and troubleshooting

* Leaving out `provider` uses `SIMPLE`, with the ingredient rules above. Experimental `COMPLEX` recipes match base materials or tags, so plain paper may stand in for a custom item made of paper. This provider rejects `exact: true`.
* The filters under `input-options` are reserved. For full data matching, use `SIMPLE` and a custom item ID.
* Keep recipe IDs unique across the folder and use lowercase letters, numbers, and underscores. Check registration in the console after reloading. If a recipe works by hand but is missing from the recipe book, sign in again.

The bundled `recipes/sample.yml` has more complete examples. Plugin developers can consult the [Recipe API](../references/recipe-api.md).


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): every crafting recipe slot consumes one item, so ingredient `amount` must be `1`; repeat an ingredient across slots for larger recipes. `exact: true` is supported by the `SIMPLE` provider only. `COMPLEX` rejects exact ingredients, and the current `input-options` filters are parsed but not applied.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

---
icon: wrench
---

# Custom Recipes

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


</details>
