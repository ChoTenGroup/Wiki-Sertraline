# Item Tiers, Types, and Levels

Use `tier` for rarity, `type` for category, and `level` for rank. Players can filter items by these labels in the item browser, and you can display them in [lore formats](lore-format.md). Configure stats and usage requirements through the relevant attributes or scripts.

| Field | Bundled file | Example IDs |
| --- | --- | --- |
| `sertraline.tier` | `tiers/default.yml` | `E`, `D`, `C`, `B`, `A`, `S` |
| `sertraline.type` | `types/default.yml` | `WEAPON`, `SWORD`, `DAGGER`, `TOOL` |
| `sertraline.level` | `levels/default.yml` | `normal`, `advanced`, `super` |

## Create and display a custom tier

Save this definition in `plugins/Sertraline/tiers/practice.yml`:

```yaml
legendary:
  name: "<gold>Legendary</gold>"
  description: "Rare adventuring gear"
  color: "<gold>"
```

Then save this item in `plugins/Sertraline/workspace/classified_sword.yml`:

```yaml
classified_sword:
  sertraline:
    tier: legendary
    type: SWORD
    level: advanced
    lore-format: loreformat
  xbuilder:
    material: IRON_SWORD
    name: "<!i><gold>Adventurer's Sword"
    lore:
      - "<!i><gray>Gear for a seasoned adventurer."
```

Reload, then run `/sertralineitem give classified_sword`. The bundled `loreformat` in `lore-formats/loreGenerator.yml` displays the sword's type, tier, and level. Use `{type:name}`, `{tier:format}`, and `{level:name}` to display those labels in your own format.

Follow the matching default file when defining a custom type or level. A type can specify a `parent`. For example, the bundled `SWORD` type has `WEAPON` as its parent. The item-browser type filter follows this hierarchy, so filtering for `WEAPON` also includes child types such as `SWORD` and `DAGGER`. Keep IDs unique within each category.
