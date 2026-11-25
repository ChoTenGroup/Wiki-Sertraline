---
icon: puzzle-piece
---

# Item Structure

### Top-Level Structure <a href="#top-level-structure" id="top-level-structure"></a>

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
| **Usage**            | -        | Used in commands: `/sertralineitem give <player> <item_id>` |
| **Storage**          | -        | Stored in item NBT as `sertraline_id` tag                   |

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
