---
icon: cubes
---

# Item Structure

You can define several items in one file under `plugins/Sertraline/workspace/`. Each top-level key is a unique item ID. Use the same capitalization in commands and recipes.

```yaml
red_token:
  xbuilder:
    material: PAPER
    name: "<!i><red>Red Token"

blue_token:
  xbuilder:
    material: PAPER
    name: "<!i><blue>Blue Token"
```

Save the file, run `/sertraline reload`, and use `/sertralineitem give red_token` in game. Add more sections as your item needs them.

| Section | Use |
| --- | --- |
| [`xbuilder`](../advanced/item-structures/section-xbuilder.md) | Material, name, lore, and common item properties |
| [`sertraline`](../advanced/item-structures/section-sertraline/README.md) | Classification, data, and actions |
| [`minecraft`](../advanced/item-structures/section-minecraft-components.md) | Data components supported by the server version |
| [`custom_data`](../advanced/item-structures/section-custom-data.md) | Custom data |
| [`mmo`](../advanced/mythiclib-mmoitems-compat.md) | MythicLib attributes |

## Configuration reference <a href="#top-level-structure" id="top-level-structure"></a>

Each item definition follows this top-level structure:

```yaml
item_id:
  xbuilder:      # Optional, but recommended
    # Basic item properties
  sertraline:    # Optional
    # Custom logic and actions
  mmo:           # Optional, requires mmo module enabled
    # MMO attributes
  minecraft:     # Optional
    # Vanilla Minecraft components
  visual:        # Optional
    # Client-side visual overrides
```

#### Item ID Requirements <a href="#item-id-requirements" id="item-id-requirements"></a>

| Property             | Type     | Description                                                 |
| -------------------- | -------- | ----------------------------------------------------------- |
| **Pattern**          | String   | Recommended use: `[a-zA-Z0-9_]+`                            |
| **Uniqueness**       | Required | Must be unique across all loaded YAML files                 |
| **Case Sensitivity** | Yes      | `sword_of_skyline` ≠ `Sword_Of_Skyline`                     |
| **Usage**            | -        | Used in commands: `/sertralineitem give <item_id> [player]` |
| **Storage**          | -        | Stored in item NBT as `sertraline_id` tag                   |

## Developer reference

### Processors & Adapters

Processors & Adapters is **Sertraline**'s internal features.

When item was loaded, the processors will work: It will read configuration and write them into a ModernSItem's data.\
After generating an item, the Adapter will work: it will read the data from ModernSItem and process it into the item.

Example a Item was this:

```
WOODEN_SWORD:
  xbuilder:
    name: "<!i><#bb9900>Wooden Sword"
    material: WOODEN_SWORD

```

In this example we created a item named `WOODEN_SWORD`.

On **Sertraline** Loading, XBuilder processor will process data to this:

`ModernSItem(key=WOODEN_SWORD, data={xbuilder={material=WOODEN_SWORD, name="<!i><#bb9900>Wooden Sword`"`}}, config=...)`

And then, when item generation, XBuilder Adapter will work, parse `xbuilder:material` to material, parse `xbuilder:name` for item name.

Source code below:

```kt
// ITEM BUILDER
val itemSource = source ?: itemSource(template.getDeepData("xbuilder:material") ?: template.getDeepData("minecraft:material"), player)
```

```kt
// XBUILDER ADAPTER ($prefix = "xbuilder")
val name = sItem.getDeepData("$prefix:name")?.toString().performPlaceholders(sItem, player)?.toComponent()
// ...
if (name != null) originalMeta.displayName(name)
```
