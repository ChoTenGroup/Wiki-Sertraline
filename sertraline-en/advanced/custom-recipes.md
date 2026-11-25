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

| Syntax | What matches |
| --- | --- |
| `"minecraft:paper"` | An item with paper as its base material |
| `"sertraline:beginner_sword"` | A Beginner Sword with matching data such as name and enchantments |
| `"itemsadder:ruby"` | A custom item supplied by the named item plugin |
| `"#minecraft:planks"` or `"tag:planks"` | A material from the Minecraft planks tag. Quote the form that starts with `#` |
| `{ itemId: "minecraft:paper", exact: true }` | An item with the same data as plain paper |
| `{ tagId: "#minecraft:planks" }` | A tag written as an object |

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

| `type` | Workstation | Configuration |
| --- | --- | --- |
| `CRAFTING` | Crafting table | `SHAPED` uses a `pattern` and character mapping. `SHAPELESS` uses an ingredient list |
| `FURNACE`, `BLASTING`, `SMOKING`, `CAMPFIRE` | Furnace, blast furnace, smoker, campfire | `ingredient`, `experience`, `cookingTime` |
| `STONECUTTING` | Stonecutter | `ingredient` |
| `SMITHING_TRANSFORM` | Smithing table upgrade | `template`, `base`, `addition` |
| `SMITHING_TRIM` | Smithing table armor trim | `template`, `base`, `addition`. Use a server and provider that support this recipe type |

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

- Leaving out `provider` uses `SIMPLE`, with the ingredient rules above. Experimental `COMPLEX` recipes match base materials or tags, so plain paper may stand in for a custom item made of paper. This provider rejects `exact: true`.
- The filters under `input-options` are reserved. For full data matching, use `SIMPLE` and a custom item ID.
- Keep recipe IDs unique across the folder and use lowercase letters, numbers, and underscores. Check registration in the console after reloading. If a recipe works by hand but is missing from the recipe book, sign in again.

The bundled `recipes/sample.yml` has more complete examples. Plugin developers can consult the [Recipe API](../refrences/api/recipe-api.md).
