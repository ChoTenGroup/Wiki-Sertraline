# Item Attribute Sections

Use `xbuilder.attributes` for vanilla equipment modifiers, `attribute-modifiers` for Sertraline attributes, and `mmo` for MythicLib stats. This example uses `demo:might` and `demo:vitality` from `attributes/default.yml`. Enable `attribute.enable` in `config.yml`.

## Fixed, percentage and dynamic modifiers

```yaml
guide_might_blade:
  xbuilder:
    material: IRON_SWORD
  attribute-modifiers:
    - type: demo:might
      id: guide:blade/might
      amount: 8
      operation: minecraft:add_value
      slot: mainhand
    - type: demo:might
      id: guide:blade/sneak_bonus
      amount: 0.2
      operation: minecraft:add_multiplied_base
      slot: mainhand
      scope: entity
      conditions:
        - "player.isSneaking()"
      update_interval: 10

guide_level_helmet:
  xbuilder:
    material: IRON_HELMET
  settings:
    attribute_modifiers:
      - type: demo:vitality
        id: guide:helmet/vitality
        amount:
          type: js
          script: "return 4 + player.getLevel() * 0.2;"
        operation: minecraft:add_value
        slot: head
```

Both lists use the same format. If you set both `attribute-modifiers` and `settings.attribute_modifiers` on the same item, the former takes precedence. Reload and give yourself each item. Test the sword while holding it in your main hand and sneaking, and test the helmet at different player levels. See [Custom Attributes](../attributes.md) for how to inspect the results.

| Field | How to use it |
| --- | --- |
| `type` | An existing attribute ID. Derived attributes cannot be modified directly |
| `id` | A unique modifier ID, such as `guide:blade/might`. Use different IDs to stack modifiers on the same attribute |
| `amount` | A number, expression string, or object with `type` / `engine` and `expression` / `script`. Use `player` for the equipped player |
| `operation` | `minecraft:add_value` adds a fixed amount, `minecraft:add_multiplied_base` adds a fraction of the value at the start of that calculation stage, and `minecraft:add_multiplied_total` multiplies by `1 + amount`. Registered custom operations also work |
| `slot` | `mainhand` / `main_hand`, `offhand` / `off_hand`, `head` / `helmet`, `chest` / `chestplate`, `legs` / `leggings`, `feet` / `boots`. Use `hand` for either hand, `armor` for armor slots, and `any` or `all` for all slots |
| `scope` | Defaults to `entity` for the wearer. Use `weapon` for combat rules that read a weapon's stats, such as the bundled combat listeners |
| `conditions` / `condition` | One condition or a list. Every condition must return true. You can use an expression string or `{type: expression, expression: "player.isSneaking()"}` |
| `update_interval` | How often to reevaluate dynamic amounts and conditions in ticks. Defaults to 20, with a minimum of 1 |

## Legacy attribute values

Use `attribute` for a map of attribute names to values. `4` adds a fixed value and `"15%"` expresses a percentage. Use `allowed` to choose slots. An empty or omitted list lets Sertraline infer the slot from armor material, defaulting to main hand for other items. `slot` and `source` describe modifier metadata. The list form above lets you choose the operation, ID and slot for each modifier.

```yaml
guide_legacy_attribute:
  xbuilder:
    material: IRON_SWORD
  attribute:
    allowed: [mainhand, offhand]
    attack_damage: 4
    crit_chance: "15%"
```

<a id="chotenattributes"></a>

## ChoTenAttributes data

For plugins that consume this data, use `chotenatb` for one group or `chotenatb_complex` for multiple groups. Both write the `chotenatb` data key. Pick one per item. If you omit `allowed` in the simple form, armor uses its equipment slot and other materials use the main hand. Specify `allowed` on each group in the complex form.

```yaml
guide_choten_sword:
  xbuilder:
    material: DIAMOND_SWORD
  chotenatb:
    attack_damage: 30
    max_health: "10%"
    allowed: [mainhand]

guide_choten_armor:
  xbuilder:
    material: DIAMOND_CHESTPLATE
  chotenatb_complex:
    - max_health: 10
      allowed: [chestplate]
    - movement_speed: "5%"
      allowed: [chestplate]
```

For MythicLib, MMOItems and MMOCore, see [MMO stat examples](../mythiclib-mmoitems-compat.md).

Use `chotenatb` or `chotenatb_complex` for the bundled ChoTen item format. In this build, support for the legacy `choten` section is limited to loading its configuration data.
