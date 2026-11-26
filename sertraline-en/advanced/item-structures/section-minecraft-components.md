---
icon: file-zipper
---

# Minecraft Data Components

Use `minecraft` for native components on the server's item. Components require Minecraft 1.20.5 or later and depend on the server version. At the top of this section, `max_stack_size` and `max-stack-size` are equivalent. Within a component's object, use the field names required by Minecraft. `material` is a special key that selects the base material. For older NBT-based builds, use [`custom_data`](section-custom-data.md).

## Numbers, switches, IDs, text, lists and removal

The examples below target servers that support these components in 1.21.4. After reloading, use `/sertralineitem give <item-id>`.

```yaml
guide_stack_token:
  xbuilder:
    material: PAPER
  minecraft:
    max_stack_size: 1
    enchantment_glint_override: true
    rarity: rare
    item_model: "minecraft:iron_ingot"

guide_component_text:
  xbuilder:
    material: PAPER
  minecraft:
    custom_name: '{"text":"Component token","color":"gold","italic":false}'
    lore:
      - '{"text":"First line","color":"gray"}'
      - '{"text":"Second line","color":"aqua"}'

guide_showpiece:
  xbuilder:
    material: DIAMOND_CHESTPLATE
  minecraft:
    unbreakable: {}
    equippable: null
```

- Use a number for `max_stack_size`, `true` or `false` for a switch, an enum such as `rare`, or a resource ID such as `"minecraft:iron_ingot"` where the component expects one.
- `custom_name` takes a native JSON text component. `lore` takes a list of text components. Use [XBuilder](section-xbuilder.md) for MiniMessage text and Sertraline item tags.
- `unbreakable: {}` supplies an empty map. `equippable: null` removes the material's default component. Use YAML `null` without quotes. Remove one component per key.

## Components with nested fields

This food and weapon use the 1.21.4 component format. The food gives six hunger points and can be eaten at full hunger. The weapon has Sharpness III and a main-hand attack modifier. Adapt the values and structure for the server version you use.

```yaml
guide_food:
  xbuilder:
    material: APPLE
  minecraft:
    food:
      nutrition: 6
      saturation: 2.4
      can_always_eat: true
    consumable:
      consume_seconds: 1.0
      animation: eat
      sound: "minecraft:entity.generic.eat"

guide_component_blade:
  xbuilder:
    material: IRON_SWORD
  minecraft:
    enchantments:
      levels:
        "minecraft:sharpness": 3
    attribute_modifiers:
      modifiers:
        - type: "minecraft:attack_damage"
          id: "guide:blade/damage"
          amount: 8
          operation: add_value
          slot: mainhand
```

## Find components by purpose

| Purpose | Components |
| --- | --- |
| Names, lore, models and tooltips | `custom_name`, `item_name`, `lore`, `item_model`, `custom_model_data`, `rarity`, `tooltip_display`, `tooltip_style`, `hide_tooltip`, `hide_additional_tooltip` |
| Stacking, durability, repairs and resistance | `max_stack_size`, `damage`, `max_damage`, `unbreakable`, `repairable`, `repair_cost`, `damage_resistant`, `break_sound` |
| Enchanting, combat and equipment | `enchantments`, `stored_enchantments`, `enchantment_glint_override`, `enchantable`, `attribute_modifiers`, `weapon`, `equippable`, `trim`, `glider`, `blocks_attacks`, `minimum_attack_charge`, `damage_type`, `kinetic_weapon`, `piercing_weapon`, `swing_animation` |
| Food, potions, use and effects | `food`, `consumable`, `use_cooldown`, `use_effects`, `use_remainder`, `potion_contents`, `potion_duration_scale`, `suspicious_stew_effects`, `death_protection`, `ominous_bottle_amplifier` |
| Tools, adventure and projectiles | `tool`, `can_break`, `can_place_on`, `charged_projectiles`, `intangible_projectile`, `debug_stick_state` |
| Containers, blocks and entities | `container`, `bundle_contents`, `container_loot`, `lock`, `block_entity_data`, `block_state`, `entity_data`, `bucket_entity_data`, `bees`, `profile` |
| Maps, books, sounds and decorations | `map_id`, `map_color`, `map_decorations`, `lodestone_tracker`, `writable_book_content`, `written_book_content`, `recipes`, `instrument`, `jukebox_playable`, `note_block_sound`, `banner_patterns`, `base_color`, `dyed_color`, `pot_decorations`, `firework_explosion`, `fireworks`, `provides_banner_patterns`, `provides_trim_material` |

The table covers components from several versions. See the [Minecraft Wiki's data component format](https://minecraft.wiki/w/Data_component_format) for the values, nested fields and availability of each component. `minecraft.custom_data` sets the native custom data component. The top-level `custom_data` section also supports older NBT-based servers.

Hold the item and run `/sertraline debug dumpComponent` to inspect its components. Append `true` for the full component list. Paper builds that provide it also have `/paper dumpitem` and `/paper dumpitem all`.
