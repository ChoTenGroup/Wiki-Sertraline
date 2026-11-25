---
icon: tent-double-peak
---

# Lore Formats

Use one lore format to display type, tier, and level consistently across items. Save this configuration as `plugins/Sertraline/lore-formats/practice.yml`.

```yaml
practice_lore:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - key: "xbuilder:lore"
      content: "{value}"
    - key: "sertraline:type"
      content: "<!i><gray>Type {type:name}"
    - key: "sertraline:tier"
      content: "<!i><gray>Tier {tier:format}"
    - key: "sertraline:level"
      content: "<!i><gray>Level {level:name}"
```

In the [item tier example](item-tier.md), change `classified_sword` from `lore-format: loreformat` to `lore-format: practice_lore`. Reload and get a fresh sword. Its original lore and the three classification lines appear in the order defined in `elements`.

## Change the displayed content

- Set `content` by itself to display fixed text. When you set a `key`, use `{value}` for its value. Lists produce one line per entry.
- `key: "sertraline:vals.note"` reads the item's `vals.note`. The `*` in `key: "*sertraline.vals.note"` tells the format to read directly from the item configuration along the dotted path. Use the normalized uppercase key for MMO stats, for example `mmo:ATTACK_DAMAGE`.
- Use `{val:note}` in a line to display the fixed value. To reuse one format across items, set the same `lore-format` ID under each item's `sertraline` section.

| Setting | Default | Effect |
| --- | --- | --- |
| `overwrite` | `true` | Replace the item's original lore. Set it to `false` to append |
| `visual` | `false` | Enable packet-based display. Requires PacketEvents and packet lore to be enabled |
| `auto-skip-blank` | `true` | Collapse consecutive blank lines |

## Show a line only when data exists

Add this entry to the end of `practice_lore`'s `elements` list. Add `vals: { note: "Event reward" }` under the sword's `sertraline` section. After reloading, the note line appears on items that supply a `note`.

```yaml
- content: "<!i><yellow>Note {val:note}"
  lineMode: ALL
  lineRequire:
    - "sertraline:vals.note"
```

When `lineRequire` lists multiple keys, `ANY` means at least one exists, `ALL` means all exist, `NOT` means none exist, and `NOT_ALL` means at least one is missing.

## Format numbers

| Syntax | Display |
| --- | --- |
| `{value}` | The original text of the value |
| `{auto}` | Removes a trailing `.0`, so `15.0` displays as `15` |
| `{plus}{auto}` | Adds a plus sign to positive numbers. Negative numbers keep their own minus sign |
| `{minus}` | Emits a standalone minus sign for negative numbers |
| `{round:2}` | Displays two decimal places in this version, so `15.567` displays as `15.57`. The number after the colon controls the field width |

Set `placeholders.plus` and `placeholders.minus` in `config.yml` to change the separate symbols. With player context, `content` can use PlaceholderAPI placeholders and inline Kether inside `{{ ... }}`. The bundled `lore-formats/compare_example.yml` and `lore-formats/switchable_example.yml` show equipment comparisons and lore switching.
