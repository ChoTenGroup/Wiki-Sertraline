---
icon: infinity
---

# MythicLib and MMOItems Compatibility

On modern builds, use `mmo` to add MMO stats to Sertraline items. Install MythicLib together with MMOItems or MMOCore. Available stats depend on the versions you install.

## Add attack damage to a main-hand weapon

Save this as `plugins/Sertraline/workspace/practice_mmo_sword.yml`.

```yaml
practice_mmo_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>Practice War Sword"
  mmo:
    attack-damage: 10
    allowed: ["mainhand"]
```

Reload and run `/sertralineitem give practice_mmo_sword`. Switch to its hotbar slot and check your MMO stats or attack a test target. The sword contributes 10 MMO attack damage in the main hand. The off hand does not apply this bonus.

If you have PlaceholderAPI installed, hold a plain diamond sword and run `/papi parse me %mythiclib_raw_stat_attack_damage%`. Switch to the practice sword, wait about a second, and run the command again. The value should be 10 higher.

Use a number for a flat bonus, such as `attack-damage: 10`. Percentage bonuses can use values such as `max-health: "5%"`. See the bundled `workspace/default/default_mmoitem.yml` for more examples.

<a id="configuration-keys"></a>

## Equipment restrictions and advanced settings

| Field | Use |
| --- | --- |
| `allowed` | Positions in which the bonus applies. Use lowercase `mainhand`, `offhand`, `helmet`, `chestplate`, `leggings`, and `boots`, or use `hand` to accept either hand. An empty list uses the matching armor slot or defaults to the main hand for other materials |
| `slot` | The equipment slot reported to MythicLib. To override it, use `MAIN_HAND`, `OFF_HAND`, `HEAD`, `CHEST`, `LEGS`, or `FEET`. The default follows the actual equipped slot |
| `source` | The modifier source reported to MythicLib. Defaults to `MAINHAND_ITEM`, `OFFHAND_ITEM`, or `ARMOR` based on where the item is equipped. Available sources depend on the MythicLib version |
| `id` | The identifier for the item's modifier. Defaults to the item ID |

If you encounter an attack-speed rebound, try `fix-attack-speed: true` under `mmo`. To undo that setting, remove the field, reload, and get a fresh copy of the item.
