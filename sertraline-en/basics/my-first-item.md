---
icon: gem
---

# Create Your First Item

[Install Sertraline](installation.md), then create `plugins/Sertraline/workspace/my_first_item.yml`:

```yaml
beginner_sword:
  xbuilder:
    material: IRON_SWORD
    name: "<!i><green>Beginner's Sword"
    lore: |
      <!i><gray>A sword for a new adventurer.
      <!i><yellow>Safe travels!
```

`beginner_sword` is the item ID; `material` selects the base item, while `name` and `lore` set its display text. Use spaces for YAML indentation. `<!i>` turns off the default italics for these MiniMessage-formatted lines.

{% hint style="success" %}
The top-level item ID—not the filename—is used in commands and recipes. Keep IDs unique and avoid the reserved `__` prefix. Workspace files may also be organized in nested folders; the default loader scans up to 10 levels deep.
{% endhint %}

In game, reload and give yourself the item:

```text
/sertraline reload
/sertralineitem give beginner_sword
```

To give two swords to an online player named Steve, use `/sertralineitem give beginner_sword Steve 2`. From the server console, include a player name because the console cannot receive an item.

{% hint style="info" %}
If the item is missing, check the reload log for the filename and item ID. The default file filter accepts `.yaml`, `.yml`, `.toml`, `.tml`, `.json`, and `.conf` files; see [Installation and Configuration](installation.md) for `file-load.item`.
{% endhint %}

For a list of available sections and additional examples, continue to [Item Structure](item-structure.md) and the [XBuilder reference](../references/xbuilder.md).
