---
icon: banjo
---

# Sertraline

Use `sertraline` for categories, levels, stored data, actions, lore formats and gems. Put it under the item ID at the same level as `xbuilder` and combine the fields that the item needs.

```yaml
guide_ranked_blade:
  xbuilder:
    material: IRON_SWORD
  sertraline:
    tier: C
    type: SWORD
    level: normal
    lore-format: comparable_format
    compare-path: MAIN_HAND
    revision-id: 1

guide_plain_coin:
  xbuilder:
    material: GOLD_NUGGET
  sertraline:
    no-sertraline-id: true
```

| Field | How to use it |
| --- | --- |
| `tier` | A tier ID from `tiers/`, such as the bundled `C` |
| `type` | A type ID from `types/`, such as `SWORD` |
| `level` | A level ID from `levels/`, such as `normal` |
| `lore-format` | Use a format ID from `lore-formats/`. Omit this field to leave it unspecified |
| `compare-path` | Equipment slot used by comparison lore, such as `MAIN_HAND`, `OFF_HAND`, `HEAD`, `CHEST`, `LEGS` or `FEET` |
| `revision-id` | An integer revision you control. If omitted, automatic tracking can assign the revision when enabled |
| `no-sertraline-id` | Use `true` to omit the Sertraline ID stamped on the generated item. Features that use this ID to look up the definition, such as item actions, restrictions and automatic updates, lose that link |
| `vals`, `vars`, `dynamics`, `keep-context` | Set fixed values, saved variables, dynamic values and build context to retain. See [Item Data](item-data.md) |
| `actions` | Set action lists under event triggers. See [Item Actions](item-actions.md) |
| `gem`, `gem-slots` | Define gems or item sockets |
| `gem-click` / `gem-click-socket` | Item-wide click socketing switch. Defaults to enabled. Use `false` to disable click socketing. A socket or gem can further restrict clicking with its own `click` setting |

Tier, type and level are labels for classification and display. Add attributes or actions if you want a tier to change damage. See [Tiers, Types, and Levels](../../item-tier.md) and [Lore Format](../../lore-format.md).

The default mappings also accept aliases such as `action`, `loreFormat`, `comparePath` / `compare`, `gemSlots`, `gemClick` and `gemClickSocket`. Use one spelling per field and item.

### Set a revision for updates

When `item-update.enabled` in `config.yml` is enabled, the revision can trigger automatic item rebuilding. Give yourself `guide_ranked_blade`, change the item and increase `revision-id: 1` to `2`, then reload. When an enabled update trigger checks the item, such as switching the held item, the old item is rebuilt. By default, rebuilding preserves stored variables. Give yourself a fresh item after reload to inspect a change immediately.

### Create a gem and a socket

Set `gem-slots` on the equipment that holds the gem and `gem` on the gem. Try the bundled `gem_example_sword` and `gem_example_ruby` first. See [Gems](../../gems.md) for socket conditions, compatible types, modification syntax, interactions and lore placeholders.

```yaml
guide_socketed_pickaxe:
  xbuilder:
    material: IRON_PICKAXE
    lore:
      - "<!i><gray>Power ${var:powerAsIn}$"
      - "<!i>${gem-slot:utility}$"
  sertraline:
    vars:
      power: 5
    gem-click: true
    gem-slots:
      utility:
        name: "<aqua>Utility Socket"
        gem-color: "blue"

guide_utility_gem:
  xbuilder:
    material: BLUE_DYE
  sertraline:
    gem:
      color: blue
      click: true
      modify-data:
        "sertraline:vars.power": "+1"
```

Hold the pickaxe in your main hand and the gem in your off hand, then run `/sertralinegem socket utility`. The displayed power rises from 5 to 6. Run `/sertralinegem unsocket utility` to remove the gem and restore the value.
