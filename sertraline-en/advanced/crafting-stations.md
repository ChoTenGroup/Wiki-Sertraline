---
icon: plug-circle-plus
---

# Crafting Stations

A crafting station is a separate menu. When a player clicks a recipe, the plugin takes the ingredients from their inventory and automatically gives them the output when the timer ends. Crafting stations are experimental.

## Make a blue token from paper

Create `blue_token` using the [item structure example](../basics/item-structure.md). Then save this configuration as `plugins/Sertraline/crafting-stations/practice.yml`.

```yaml
practice_station:
  display:
    title: "<blue>Practice Station"
    layout:
      - "         "
      - "    I    "
      - "         "
    key: "I"
  recipes:
    press_paper:
      displayName: "Blue Token"
      time: 3
      inputs:
        - item: "minecraft:paper"
          amount: 2
      outputs:
        - item: "sertraline:blue_token"
          amount: 1
```

1. Run `/sertraline reload` and put at least two pieces of plain paper in your inventory.
2. Run `/sertraline station sertraline:practice_station`. The station ID is the outermost key, `practice_station`.
3. Click the blue token in the center of the menu. Two pieces of paper are taken, and one token arrives about three seconds later.

## Change the menu

Use nine characters per row in `display.layout`. The number of rows sets the chest menu size. Use quotes to preserve spaces. Each character that matches `display.key` reserves a slot for a recipe. The icon defaults to the first output. To show a book instead, set `display.item` to `"minecraft:book"` under the recipe.

To add a background, replace the spaces you want to fill with `#` and add the following under the station's `display`, alongside `layout`.

```yaml
elements:
  background:
    char: "#"
    material: GRAY_STAINED_GLASS_PANE
    name: "<gray>Decoration"
```

For menus with more than one page of recipes, add elements named `next_page` and `previous_page` and put their `char` values in the layout. Set `roll: true` to wrap between the first and last pages. Element names and descriptions can use `{page}`, `{total_pages}`, `{next_page}`, and `{previous_page}`.

## Change a recipe

- Give each entry under `recipes` its own ID. `time` is in seconds and can also use an expression such as `"{{ random 1 to 10 }}"`.
- Both `inputs` and `outputs` are lists. In each entry, use `item` with an ID such as `minecraft:paper`, `sertraline:blue_token`, or `itemsadder:ruby`. Bare IDs are read as vanilla materials. `amount` defaults to 1. Custom ingredients must match the item data, so start with an item that builds the same way each time.
- A player can have one active craft. Opening a station again during the timer normally cancels the craft and refunds the ingredients. Set `options.cancellable: false` on the recipe to prevent cancellation through reopening the menu.

To require a permission, add this under `press_paper`, alongside `inputs`. Test by clicking the recipe on an account without the permission.

```yaml
conditions:
  - name: "Blue token crafting permission"
    type: permission
    condition: "example.craft.blue_token"
messages:
  ConditionNotMetPermission: "<red>Missing {condition_name}.</red>"
```

Use `type: kether` or `boolean` to check a Kether script in `condition`. Use `type: Scaled` to compare `amount` with `required`; the condition passes when the amount reaches the requirement. Override messages by type, for example with `ConditionNotMetScaled`. Message placeholders include `{condition_name}`, `{condition_amount}`, `{condition_required}`, and `{condition_type}`.

## Show a message when crafting finishes

Add this under `press_paper`, alongside `time`. Reload and make another token to see the message when the output arrives.

```yaml
agents:
  onClaim: |-
    tell "Your blue token is ready!"
```

`onStart` runs when crafting begins. `onClaim` runs after the output is given. A condition can have its own `onSuccess` and `onFail` scripts under `agents`. Station and recipe scripts can both be used, with the recipe script running first. Scripts without a prefix use Kether. Start a JavaScript script with `gjs:` for GraalJS or `js:` for JSR 223, for example `gjs: player.sendMessage("Crafting complete!")`.

## Troubleshooting

- If a station is missing, check the top-level ID, run `/sertraline debug getStations`, and check the reload log.
- If the recipe is missing from the menu, check that the layout contains the character from `display.key` and that `elements` leaves those slots free.
- If crafting will not start, check permissions, ingredient quantities, and item data. Give yourself fresh copies by ID when testing custom ingredients.
- If the timer ends while the player is offline, the plugin attempts to give them the output when they rejoin. If they rejoin before the timer ends, it attempts to refund their materials. A configuration reload cancels active crafts for online players and refunds their ingredients.

See the bundled `crafting-stations/craftingStation.yml` for more layouts and item references. The current menu and crafting flow do not use `displayTime`, `plural`, or `option.enable-tracker`.
