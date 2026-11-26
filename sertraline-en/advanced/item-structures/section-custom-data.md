---
icon: desktop-arrow-down
---

# Custom Data

Use `custom_data` under the item ID to store values that another plugin or script can read. On Minecraft 1.20.4 and earlier, the values go into custom NBT. On 1.20.5 and later, they go into the custom data component.

```yaml
guide_data_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Reward Token"
  custom_data:
    event: summer
    rank: 1
    claimed: false
    details:
      color: gold
      multiplier: 1.5
    tags: [event, reward]
    rewards:
      - item: emerald
        amount: 2
      - item: gold_ingot
        amount: 1
```

Save, reload, and run `/sertralineitem give guide_data_token`. The example includes text, a number, a boolean, a nested map, a list of values and a list of maps. Keep the element type consistent within a list. NBT stores booleans as bytes, so a viewer may show `1b` for `true` and `0b` for `false`.

Choose keys specific to your server to avoid conflicts with Sertraline keys such as `sertraline_id` and `sertraline_data`. Use [Sertraline Item Data](section-sertraline/item-data.md) when you want actions to save variables or display them in lore.

On a modern server, hold the item and run `/sertraline debug dumpComponent`. Inspect `minecraft:custom_data` to see the saved values. An NBT viewer can inspect older items.
