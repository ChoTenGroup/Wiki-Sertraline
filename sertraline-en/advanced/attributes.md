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

## Try the bundled combat attributes

On a modern server with the bundled combat listeners installed, run `/sertralineitem give combat_feather_boots` and wear the boots. Their 100 fall defense halves the base fall damage under the bundled formula, `damage × 100 / (100 + defense)`. Vanilla enchantments and other effects can still change final damage.

The combat listeners require attributes and `script.graaljs.allow-java-type: true` in `config.yml`. Find the sample gear in `workspace/default/default_combat_weapons.yml` and `default_combat_armor.yml`, and the rules in `global-listeners/default-combat.yml`. When updating an existing server, check that the files from the current version are installed.

### Check the value in game

On a test server with GraalJS, add this section inside `focus_wand` alongside `xbuilder`. Reload and right-click while holding the wand to see your current Focus in chat.

```yaml
sertraline:
  actions:
    onRightClick:
      - async: false
        graaljs: |
          player.sendMessage("Focus " + AttributeAPI.getValue(player, "tutorial:focus"));
```
