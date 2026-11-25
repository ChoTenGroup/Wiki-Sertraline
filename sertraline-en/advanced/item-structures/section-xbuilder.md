---
icon: hammer-war
---

# XBuilder

Use `xbuilder` to set an item's material, name, lore, and common properties. This example makes a diamond sword with Unbreaking III and Sharpness V. Save it to `plugins/Sertraline/workspace/cool_sword.yml`.

```yaml
cool_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "&2Cool Sword"
    lore: |
      <!i><gray>A sword with two enchantments.
    enchants:
      UNBREAKING: 3
      SHARPNESS: 5
```

Run `/sertraline reload` and `/sertralineitem give cool_sword`. Hover over the sword to see its name, description, and enchantments. Under `enchants`, use the enchantment name as the key and its level as the value.

## Change durability and attack damage

To make the sword unbreakable and give it a main-hand attack damage modifier, add these fields under `xbuilder`, at the same indentation as `material`.

```yaml
unbreakable: true
attributes:
  ATTACK_DAMAGE:
    amount: 10
    operation: ADD_NUMBER
    slot: HAND
```

On servers with data components, `name` sets the item's `custom_name`. Names and lore accept MiniMessage and legacy color codes. Write lore as a YAML list or a multiline string.

See the [XBuilder reference](../../refrences/xbuilder.md) for more fields. Use [Minecraft data components](section-minecraft-components.md) for options such as stack size and equip behavior.
