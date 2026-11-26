---
icon: binary-circle-check
---

# Item Data

## Pick a data type

| Configuration field | Text tag | Use |
| --- | --- | --- |
| `vals` / `val` | `${val:key}$` | A fixed value shared by all copies, such as text, a number, a list or a map |
| `vars` / `var` | `${var:key}$` | An initial value saved on each generated item. An action can change it later |
| `dynamics` / `dynamic` | `${dynamic:key}$` | Evaluate Kether when rendering or reading the value. Use a string, YAML block or list of script lines |
| `keep-context` / `keepContext` | `${var:key}$` | Save selected build context keys as item variables. Use `${context:key}$` to read context during that build |

```yaml
guide_roll_card:
  xbuilder:
    material: PAPER
    lore:
      - "<!i><gray>${val:guide_title}$"
      - "<!i><gray>Uses: ${var:usesAsIn}$"
      - "<!i><gray>Preview: ${dynamic:rollAsIn}$"
  sertraline:
    vals:
      guide_title: "Practice card"
      rewards: [Gold, Paper]
      stats: {points: 10, bonus: 2}
    vars:
      uses: 3
    dynamics:
      roll: "random 1 to 6"

guide_drop_medal:
  xbuilder:
    material: GOLD_NUGGET
    lore:
      - "<!i><gray>Killer: ${var:killer?:—}$"
  sertraline:
    keep-context: [killer]
```

Reload and run `/sertralineitem give guide_roll_card`. `uses` starts at 3. `roll` evaluates to a number from 1 to 6. The displayed text stays the same until the lore is rendered again. To keep a single roll, save the result as a variable.

When `guide_drop_medal` is generated as a [MythicMobs drop](../../../references/universal-mythicmobs.md) with a player as killer, it can retain `killer` in the item. A plain give command has no such context and uses the fallback `—`.

## Save a variable and refresh the lore

Variables are stored under `sertraline_data`. Assigning a temporary script variable alone leaves the saved item unchanged. Add the following `actions` under the card's existing `sertraline` section, aligned with `vars`.

```yaml
actions:
  onRightClick:
    - async: false
      js: |
        if (Number(uses) > 0) {
          var tag = ItemStackUtil.getItemTag(bItem);
          tag.putDeep("sertraline_data.uses", Number(uses) - 1);
          ItemStackUtil.setItemTagDirect(bItem, tag);
          ItemStackUtil.rebuildLore(bItem, player);
        }
```

Reload, give yourself a fresh copy and right-click with it in your hand. The number of uses drops from 3 to 2 and rebuilding the lore evaluates the roll again. See [Item Text Tags](../../item-text-tags.md) for formatting and fallbacks. The bundled `sword_of_counter` in `workspace/default/default_weapon.yml` shows how to accumulate damage instead.
