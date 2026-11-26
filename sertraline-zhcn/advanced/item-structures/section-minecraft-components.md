---
icon: file-zipper
---

# Minecraft 数据组件

`minecraft` 为服务器上的实际物品配置原版组件，需要 1.20.5 或更高版本，具体组件取决于服务端版本。组件名可写为 `max_stack_size` 或 `max-stack-size`。组件内部的键使用 Minecraft 规定的原名。`material` 是 Sertraline 读取基础材质的特殊字段。旧服的自定义 NBT 写在 [`custom_data`](section-custom-data.md)。

## 数字、开关、ID、文本、列表与移除

以下示例适用于支持相应组件的 1.21.4 服务端。一次只添加你需要的组件。重载后，用 `/sertralineitem give <物品ID>` 领取。

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

- 数字直接填写，例如 `max_stack_size: 1`。布尔值写 `true` 或 `false`。枚举如 `rare`，资源 ID 如 `"minecraft:iron_ingot"`。
- `custom_name` 是原版 JSON 文本组件，`lore` 是文本组件列表。要用 MiniMessage 或物品标签构造名称、描述，见 [XBuilder](section-xbuilder.md)。
- `unbreakable: {}` 给组件提供空对象。`equippable: null` 移除材质自带的装备组件。用 YAML 的 `null`，双引号包起来的 `"null"` 是普通文字。每个组件分别移除。

## 配置带字段的组件

下面的食物和武器使用 1.21.4 的数据格式。食物恢复 6 点饥饿值并可在饱食时吃，武器自带锋利 III 和主手攻击加成。其他版本根据相应的原版组件格式调整结构。

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

## 按用途查组件

| 用途 | 常用组件 |
| --- | --- |
| 名称、描述、模型、提示外观 | `custom_name`、`item_name`、`lore`、`item_model`、`custom_model_data`、`rarity`、`tooltip_display`、`tooltip_style`、`hide_tooltip`、`hide_additional_tooltip` |
| 堆叠、耐久、修理、抗性 | `max_stack_size`、`damage`、`max_damage`、`unbreakable`、`repairable`、`repair_cost`、`damage_resistant`、`break_sound` |
| 附魔、战斗、穿戴 | `enchantments`、`stored_enchantments`、`enchantment_glint_override`、`enchantable`、`attribute_modifiers`、`weapon`、`equippable`、`trim`、`glider`、`blocks_attacks`、`minimum_attack_charge`、`damage_type`、`kinetic_weapon`、`piercing_weapon`、`swing_animation` |
| 食物、药水、使用和效果 | `food`、`consumable`、`use_cooldown`、`use_effects`、`use_remainder`、`potion_contents`、`potion_duration_scale`、`suspicious_stew_effects`、`death_protection`、`ominous_bottle_amplifier` |
| 工具、冒险与投射物 | `tool`、`can_break`、`can_place_on`、`charged_projectiles`、`intangible_projectile`、`debug_stick_state` |
| 容器、方块和实体 | `container`、`bundle_contents`、`container_loot`、`lock`、`block_entity_data`、`block_state`、`entity_data`、`bucket_entity_data`、`bees`、`profile` |
| 地图、书、音效和装饰 | `map_id`、`map_color`、`map_decorations`、`lodestone_tracker`、`writable_book_content`、`written_book_content`、`recipes`、`instrument`、`jukebox_playable`、`note_block_sound`、`banner_patterns`、`base_color`、`dyed_color`、`pot_decorations`、`firework_explosion`、`fireworks`、`provides_banner_patterns`、`provides_trim_material` |

上表包含不同版本的组件。写法以当前服务端的组件格式为准，具体取值、对象字段和版本见 [Minecraft Wiki 数据组件格式](https://minecraft.wiki/w/Data_component_format)。例如 `minecraft.custom_data` 可写原版自定义数据组件，`custom_data` 顶层段还支持旧版 NBT。

手持物品执行 `/sertraline debug dumpComponent` 查看实际组件，末尾加 `true` 查看完整列表。在支持它的 Paper 构建上也可用 `/paper dumpitem` 或 `/paper dumpitem all`。
