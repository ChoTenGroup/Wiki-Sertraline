# Custom Attributes

Use attributes to give equipment numeric bonuses. This example adds Focus, then creates a wand that grants eight points while held in the main hand. Make sure `attribute.enable` is `true` in `config.yml`.

## Create the attribute and wand

Save the definition in `plugins/Sertraline/attributes/practice.yml`:

```yaml
attributes:
  tutorial:focus:
    name: "Focus"
    base: 10
    constraint:
      min: 0
      max: 100
```

Save the item in `plugins/Sertraline/workspace/focus_wand.yml`:

```yaml
focus_wand:
  attribute-modifiers:
    - type: tutorial:focus
      id: tutorial:wand/focus
      amount: 8
      operation: minecraft:add_value
      slot: mainhand
  xbuilder:
    material: BLAZE_ROD
    name: "<!i><aqua>Focus Wand"
```

Run `/sertraline reload` and collect the item with `/sertralineitem give focus_wand`. Focus starts at 10, rises to 18 with the wand in your main hand, and returns to 10 when you remove it. Lore, scripts, or gameplay rules can read the custom value.

## Adjust a bonus

| Field | Use |
| --- | --- |
| `type` | ID of the attribute to modify |
| `id` | ID of the bonus. Give bonuses distinct IDs when they should apply together to the same attribute |
| `amount` | Bonus amount. Multiply operations use fractions, so `0.2` means 20% |
| `operation` | `minecraft:add_value` adds a flat amount. `minecraft:add_multiplied_base` adds a fraction of the value at the start of that calculation stage. `minecraft:add_multiplied_total` multiplies by `1 + amount` |
| `slot` | Equipment slot such as `mainhand`, `offhand`, `head`, `chest`, `legs`, or `feet` |

See `attributes/default.yml` and `workspace/default/default_attribute.yml` for more examples.

## Derived values, custom operation stages, and base providers

The attribute engine can calculate one attribute from others, define custom modifier stages, and use an entity's vanilla attribute as a base:

```yaml
attributes:
  practice:armor_ward:
    name: Armor Ward
    base:
      type: vanilla
      attribute: minecraft:armor
      fallback: 0
      transform: "value * 0.5"
  practice:zombie_armor:
    name: Zombie Armor
    base:
      type: by_entity_type
      attribute: minecraft:armor
      fallback: 0
    entities: [minecraft:zombie]
  practice:power:
    name: Power
    base: 10
    operations:
      - minecraft:add_value
      - practice:add_capped
  practice:combat_power:
    name: Combat Power
    derived: "practice:power * 2"
  practice:vitality:
    name: Vitality
    base: 0
    entities: [minecraft:player]
    sync:
      - target: minecraft:max_health
        operation: add_value
        value: value

attribute_operations:
  practice:add_capped:
    expression: "MIN(current + amount, base * 2)"
```

`vanilla` reads the living entity's current vanilla base (before modifiers); `by_entity_type` reads Paper's default value for that entity type. Both use `fallback` when the attribute or platform is unsupported, and `transform` receives the provider result as `value`. `entities` can restrict a definition to entity IDs or Bukkit entity tags. `sync` applies the calculated value to a vanilla attribute as a modifier; it flows from the Sertraline attribute to Minecraft, unlike a vanilla `base` provider which reads from Minecraft. In an `operations` list, stages run in the listed order; custom stage expressions receive `base` (the value at the start of that stage), `current`, and `amount`. `MIN` and `MAX` are available in expressions.

{% hint style="success" %}
A `derived` attribute is calculated from its referenced attribute IDs; it does not use its own base, item modifiers, operation stages, or sync rules. Use a normal attribute when equipment should modify the value.
{% endhint %}

## Try the bundled combat attributes

On a modern server with the bundled combat listeners installed, run `/sertralineitem give combat_feather_boots` and wear the boots. Their 100 fall defense halves the base fall damage under the bundled formula, `damage × 100 / (100 + defense)`. Vanilla enchantments and other effects can still change final damage.

The combat listeners require attributes and `script.graaljs.allow-java-type: true` in `config.yml`. Find the sample gear in `workspace/default/default_combat_weapons.yml` and `default_combat_armor.yml`, and the rules in `global-listeners/default-combat.yml`. When updating an existing server, check that the files from the current version are installed.

### Check the value in game

On a test server with GraalJS, add this section inside `focus_wand` alongside `xbuilder`. Reload and get the wand, then switch to its hotbar slot and wait about a second before right-clicking. Chat should show a Focus value of 18.

```yaml
sertraline:
  actions:
    onRightClick:
      - async: false
        graaljs: |
          player.sendMessage("Focus " + AttributeAPI.getValue(player, "tutorial:focus"));
```
