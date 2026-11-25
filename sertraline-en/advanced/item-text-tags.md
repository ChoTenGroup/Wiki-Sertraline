---
icon: sensor-cloud
---

# Item Text Tags

Use `${tag:parameter}$` to insert data into an item name, lore, or another field that handles tags. This example displays the number of rounds, the rewards, and a contact. Save it to `plugins/Sertraline/workspace/event_pass.yml`, reload, and run `/sertralineitem give event_pass`.

```yaml
event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Event Pass"
    lore:
      - "<!i><gray>Rounds ${val:roundsAsIn}$"
      - "<!i><gray>Rewards ${val:rewardsAsLiSplitBy / }$"
      - "<!i><gray>Contact ${val:contact?:—}$"
  sertraline:
    vals:
      rounds: 2.7
      rewards: ["Gold", "Paper"]
```

The result displays `3` rounds and `Gold / Paper` as rewards. Since `contact` is missing, the contact line shows `—`. Add `contact: "Steve"` under `vals`, then reload and get a fresh pass to see the new value.

## Choose and format a value

- `val` reads `sertraline.vals`. Use `var` for data stored on the item and `dynamic` for values computed on demand. See [Item Data](item-structures/section-sertraline/item-data.md).
- Add `?:fallback` after the key to supply fallback text, or use `!!` to render an empty string in place of `null`. If the corresponding data section is entirely empty, the tag can remain as written. The example defines other entries in `vals`.
- Append `AsIn` to round to a whole number, `AsSt` for text, `AsDo` or `AsFl` for floating-point numbers, `AsBy` for a byte, `AsLo` for a long integer, `AsBo` for a Boolean, `AsBd` for a high-precision decimal, `AsLi` to join a list, or `AsMa` to join key-value pairs.

## Change list and map separators

Lists are comma-separated by default. `AsLiSplitBy / ` separates entries with a slash and surrounding spaces. Use `AsLiSplitByBLANK` to join with an empty separator. To join with a single space, write that space after `SplitBy`.

For a map, use a backtick between the two separators after `AsMaSplitBy`. The first separates a key from its value and the second separates entries. For example, ``${val:statsAsMaSplitBy=` / }$`` displays `{ hp: 10, speed: 2 }` as `hp=10 / speed=2`.
