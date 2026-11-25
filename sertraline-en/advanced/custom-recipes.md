---
icon: wrench
---

# Custom Recipes

Save recipes under `plugins/Sertraline/recipes/` to let players craft items in vanilla interfaces. Modern recipes are unavailable on the Legacy build. For a separate menu with timed crafting and ingredient quantities, see [Crafting Stations](crafting-stations.md).

## Try a bundled shaped recipe

Find `star_from_vanilla` in `recipes/sample.yml` and make sure `star_of_recall` is loaded from `workspace/default/default_misc.yml`. The example below omits the optional `provider` field.

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

Place one gold ingot in the center of a crafting table and one sheet of paper in each surrounding slot. Each string in `pattern` represents a row, `ingredients` maps its letters to items, and a space represents an empty slot. Edit the existing entry if you want to change this recipe.

## Add a shapeless recipe

Follow the [first item tutorial](../basics/my-first-item.md) to create `beginner_sword`, then save this as `recipes/practice.yml`:

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

Run `/sertraline reload`. Place two iron ingots and a stick in three separate crafting slots in any order. Each slot consumes one item, so list repeated ingredients once per slot.

## Add a furnace recipe

Add this top-level entry to `recipes/practice.yml` and reload. Put mud and fuel in a furnace. The recipe takes 200 game ticks, about 10 seconds, and grants experience when the player collects the result.

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

## Choose ingredients and results

- Use `minecraft:paper` for a vanilla material, `sertraline:beginner_sword` for a Sertraline item, and the source plugin's prefix for external items. A bare ID is looked up as a vanilla material first, then as a Sertraline item. Changes to custom item names, enchantments, or other item data can affect matching.
- An ingredient can be a Minecraft item tag such as `"#minecraft:planks"` or an alternatives map such as `{ options: ["minecraft:oak_planks", "minecraft:birch_planks"] }`.
- `result.id` selects the output and `result.count` defaults to 1. Crafting table recipes can run scripts or commands under `result.function` when a player takes the output. See `craft_eternal_with_kether` and `any_sword_to_star` in the bundled file.

| `type` | Interface | Main fields |
| --- | --- | --- |
| `CRAFTING` | Crafting table | `SHAPED` uses a pattern and letter mapping. `SHAPELESS` uses an ingredient list |
| `FURNACE`, `BLASTING`, `SMOKING`, `CAMPFIRE` | Furnace, blast furnace, smoker, campfire | `ingredient`, `experience`, `cookingTime` |
| `STONECUTTING` | Stonecutter | `ingredient` |
| `SMITHING_TRANSFORM`, `SMITHING_TRIM` | Smithing table | `template`, `base`, `addition` |

If a recipe is missing, check the reload log for unresolved items or registration errors and make sure the recipe ID is unique.

## Recipe System

### Overview

The recipe system registers custom Minecraft recipes using vanilla mechanics — crafting table, furnace, blast furnace, smoker, campfire, stonecutter, and smithing table. It does **not** use the custom GUI from the crafting station system.

Key features:

* **8 recipe types**: Crafting (shaped/shapeless), Furnace, Blasting, Smoking, Campfire, Stonecutting, Smithing Transform, Smithing Trim
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

Give each recipe a unique top-level ID. Select the fields for its type from the table below and use the matching complete example. All types use `result` with an item `id` and `count`. Set `group` for recipe-book grouping. `showNotification` controls crafting-table notifications.

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

### Recipe registration

You can omit `provider` when creating a recipe. In the verified Sertraline 3.13.2.2 build, the default is `SIMPLE`. The examples on this page use that default.

| `provider` value | Registration | When to use it |
| --- | --- | --- |
| `SIMPLE` (default) | Registers through the Bukkit API | Regular recipes, including recipes that match metadata on Sertraline or external plugin ingredients |
| `COMPLEX` (experimental) | Registers directly with Minecraft's server recipe manager (NMS) | Recipes that can match ingredients by their base material or tag |

Writing `provider: SIMPLE` has the same effect as omitting the field. The bundled `recipes/sample.yml` leaves the `COMPLEX` examples commented out. In this mode, custom names, enchantments, and components do not affect ingredient matching. If a custom card uses paper as its base material, ordinary paper can satisfy the ingredient. `COMPLEX` rejects ingredients with `exact: true`.

After a player joins or recipes are reloaded, Sertraline uses NMS to unlock registered recipes for the recipe book. This step applies to both registration modes.

***

### Ingredients

#### Formats

**1. String format (simple)**

```yaml
ingredient: "sertraline:doe_card"
```

**2. Object format**

```yaml
ingredient:
  itemId: "sertraline:doe_card"
  amount: 1
```

**3. Tag format**

```yaml
ingredient: "#minecraft:planks"     # Bukkit tag
```

```yaml
ingredient: "tag:planks"            # Shorthand (assumes minecraft namespace)
```

Tag object format:

```yaml
ingredient:
  tagId: "#minecraft:planks"
  amount: 1
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

A bare ID checks vanilla materials first and then falls back to Sertraline items.

#### Ingredient matching

| Ingredient | Default registration (`SIMPLE`) | Direct NMS registration (`COMPLEX`) |
| --- | --- | --- |
| Vanilla item such as `minecraft:stone` | Matches by material. In the object form, `exact: true` matches an unmodified vanilla stack | Matches by material |
| Sertraline item such as `sertraline:doe_card`, or an external plugin item | Matches the generated item, including its name, enchantments, and other metadata. Use items that build consistently | Matches the item's base material |
| Tag such as `#minecraft:planks` | Accepts any material in the tag | Accepts any material in the tag |
| Alternatives under `options` | Accepts any option that satisfies its own matching rules | Accepts any option's base material or tag member |

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

The loader accepts these fields. They currently have no effect on ingredient matching. To match metadata, use a custom item ID as the ingredient with the default registration mode.

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
| `data-filter`         | List\<String>      | Reserved data-key list |
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
  type: STONECUTTING

  ingredient: "sertraline:ultimate_pickaxe"
  result:
    id: sertraline:block_wand
    count: 2
```

#### Multi-choice ingredient (OR) + command function

```yaml
any_sword_to_star:
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
