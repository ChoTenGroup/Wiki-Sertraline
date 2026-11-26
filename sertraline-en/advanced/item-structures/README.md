# Item Configuration Sections

Create item files under `plugins/Sertraline/workspace/`. Each top-level key is an item ID. Indent all sections under that ID at the same level as `xbuilder`. You can organize files in subdirectories and define several items in one file. Use unique, case-sensitive IDs. IDs beginning with `__` are reserved for [private items](../item-manager-system.md).

## Start with a complete example

Save this example as `workspace/guide_items.yml` and run `/sertraline reload`. In game, run `/sertralineitem give guide_event_pass`. Hover to see the event name, right-click with the pass in your main hand, then try using it as paper in a crafting recipe.

```yaml
guide_event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Event Pass"
    lore:
      - "<!i><gray>Event ${val:event_name}$"
  sertraline:
    vals:
      event_name: "Summer Festival"
    actions:
      onRightClick:
        - async: false
          kether: 'tell "This is a Summer Festival pass."'
  common:
    disable-crafting: true
  custom_data:
    event_category: summer

guide_blank_card:
  xbuilder:
    material: PAPER
```

For a plain item, a single `material` is enough, as in `guide_blank_card`. To share names, lore or stats across items, use `$template`, `arguments` and `$list-mode` as described in [Templates](../templates.md).

## Pick the section you need

| Section under the item ID | Use |
| --- | --- |
| [`xbuilder`](section-xbuilder.md) | Set a material, display name, lore, durability, enchantments, vanilla equipment modifiers and material-specific properties |
| [`sertraline`](section-sertraline/README.md) | Set tiers, types, levels, actions, data, lore formats and gems |
| [`minecraft`](section-minecraft-components.md) | Set or remove vanilla components on the server's item |
| [`visual`](visual-items.md) | Control what the client sees, such as displaying a paper item as a diamond |
| [`custom_data`](section-custom-data.md) | Store data for scripts and integrations |
| [`common`](#restrict-item-uses) | Restrict how vanilla operations can use the item |
| [`attribute-modifiers`, `settings.attribute_modifiers`, `attribute`](section-attributes.md) | Apply Sertraline attributes to the equipped entity or weapon |
| [`mmo`](../mythiclib-mmoitems-compat.md) | Apply MMO stats through MythicLib |
| [`chotenatb`, `chotenatb_complex`](section-attributes.md#chotenattributes) | Store stats in the ChoTenAttributes format |

## Choose a base item

Set `material: PAPER` under `xbuilder` for a normal item. Namespaced vanilla IDs such as `"minecraft:paper"` also work. The plugin checks `xbuilder.material`, then `minecraft.material`, then `itemdef.material`. Other fields under `itemdef` depend on the plugin that provides the section.

To use an existing Sertraline item's vanilla material, set `material: "sertraline:guide_blank_card"`. Use a template to share the whole definition. If an external item provider is installed and enabled, you can use `"provider:item-id"` under `xbuilder.material`. For example, use `"mythicmobs:MythicBlade"` if you have a MythicMobs item named `MythicBlade`. The external item becomes the base stack.

## File formats

JSON and TOML can define the same items as YAML. Save files as UTF-8. By default, `config.yml` → `file-load.item` accepts `.yml`, `.yaml`, `.conf` (YAML), `.json`, `.toml` and `.tml` and skips names beginning with `#` or `!`.

```json
{
  "guide_json_card": {
    "xbuilder": {
      "material": "PAPER",
      "name": "<!i><gold>JSON Card"
    }
  }
}
```

```toml
[guide_toml_card.xbuilder]
material = "PAPER"
name = "<!i><gold>TOML Card"
```

## Restrict item uses

| `common` field | Effect |
| --- | --- |
| `disable-crafting` | `true` blocks use in a crafting table or the inventory crafting grid. `vanilla` applies only to recipes in the `minecraft` namespace |
| `disable-smelting` | Blocks use as a smelting input. `true` and `vanilla` have the same effect |
| `disable-smithing` | Blocks taking a smithing result on builds with smithing events. `true` and `vanilla` have the same effect |
| `disable-village-trade` | Blocks selecting a villager trade that explicitly requires the item on builds with trade events. `true` and `vanilla` have the same effect |

| `disable-enchanting` | Blocks enchanting the item at an enchanting table |
| `disable-anvil` | If either input is restricted, hides the anvil result and blocks taking it |
| `disable-brewing` | Cancels the whole brew when the ingredient or any of the three bottle slots contains a restricted item |
| `disable-stonecutting` | Blocks taking the stonecutter result on servers with stonecutters. The interface can still show a preview |

All settings default to off. Set `false` explicitly to disable a restriction, or use quoted `"yes"` and `"no"` for on and off. [Crafting stations](../crafting-stations.md) have their own ingredient rules. These restrictions find the public definition by its Sertraline ID, which is removed by `sertraline.no-sertraline-id: true`. For enchanting tables, anvils, brewing stands and stonecutters, `true` and `vanilla` have the same effect. Test the sword at an enchanting table and anvil, the nether wart with water bottles, and the stone in a stonecutter. These four settings require a build that includes their restrictions.

```yaml
guide_locked_sword:
  xbuilder:
    material: IRON_SWORD
  common:
    disable-enchanting: true
    disable-anvil: true

guide_locked_wart:
  xbuilder:
    material: NETHER_WART
  common:
    disable-brewing: true

guide_locked_stone:
  xbuilder:
    material: STONE
  common:
    disable-stonecutting: true
```

Configure `enchanting-blocked`, `anvil-blocked` and `stonecutting-blocked` under `messages.common` in `config.yml`. Set a message to an empty string to suppress it. Brewing runs in the background and sends no player message.

## Check your item

1. Reload, run `/sertraline manager use public`, and run `/sertraline manager info <item-id>` to check that the definition loaded.
2. Give yourself a fresh copy. Previously issued items can retain durability, enchantments and variables depending on the item update settings.
3. If giving fails, check the console, ID, file extension, indentation and material. For components, models and integrations, also check your server version, resource pack and dependencies.
