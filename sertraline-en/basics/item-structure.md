---
icon: cubes
---

# Item Structure

Put item definitions in files under `plugins/Sertraline/workspace/`. A file can contain multiple top-level item IDs; each ID maps to the sections that configure that item.

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

After saving, run `/sertraline reload` and give an item with `/sertralineitem give red_token`. The filename does not have to match an item ID, but each ID should be unique across loaded item files and written with consistent capitalization.

| Section | Use |
| --- | --- |
| [`xbuilder`](../advanced/item-structures/section-xbuilder.md) | Material, name, lore, and common item properties |
| [`sertraline`](../advanced/item-structures/section-sertraline/README.md) | Sertraline data, classification, and actions |
| [`minecraft`](../advanced/item-structures/section-minecraft-components.md) | Vanilla data components supported by the server version |
| [`custom_data`](../advanced/item-structures/section-custom-data.md) | Custom item data |
| [`mmo`](../advanced/mythiclib-mmoitems-compat.md) | MythicLib-backed attributes when the integration is available |
| [`visual`](../advanced/item-structures/visual-items.md) | Client-side item appearance overrides |

{% hint style="success" %}
All sections are optional; include only the sections your item needs. For a simple custom item, start with `xbuilder`. The `minecraft` section depends on server-version support, and `visual` overrides require PacketEvents 2.10 or newer; these features are not available on the Legacy 1.12.2 build.
{% endhint %}

## Item IDs

The top-level key is the item ID used by commands, recipes, and item references. Use a simple ID such as `red_token`; item lookup is case-sensitive. IDs beginning with `__` are reserved for private manager items and are skipped when loading workspace definitions.

## Top-level shape

```yaml
item_id:
  xbuilder:       # Optional; common item properties
    material: PAPER
  sertraline:     # Optional; Sertraline data and actions
    # ...
  mmo:            # Optional; requires the matching integration
    # ...
  minecraft:      # Optional; version-dependent data components
    # ...
  visual:         # Optional; PacketEvents-based client overrides
    # ...
```

For a full beginner walkthrough, see [Create Your First Item](my-first-item.md). The [item configuration sections](../advanced/item-structures/README.md) document each section in detail.
