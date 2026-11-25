---
icon: binary-circle-check
---

# Item Data

Define data under an item's `sertraline` section and use [item text tags](../../item-text-tags.md) to show those values in its name or lore.

| Field | Tag | When to use it |
| --- | --- | --- |
| `vals` | `${val:key}$` | Shared information such as the name of an event |
| `vars` | `${var:key}$` | Data stored on each item when built, such as a remaining use count |
| `dynamics` | `${dynamic:key}$` | Values computed with a Kether script when read |

## Display a fixed value

Save this as `plugins/Sertraline/workspace/practice_badge.yml`. Reload, run `/sertralineitem give practice_badge`, and hover over the item to see the event name.

```yaml
practice_badge:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Event Badge"
    lore:
      - "<!i><gray>Event ${val:season}$"
  sertraline:
    vals:
      season: "Summer Festival"
```

## Store remaining uses

Add the following at the top level of the same file. Reload and run `/sertralineitem give practice_ticket`. Each ticket starts with three uses, which item actions can update independently.

```yaml
practice_ticket:
  xbuilder:
    material: PAPER
    name: "<!i><aqua>Practice Ticket"
    lore:
      - "<!i><gray>Uses left ${var:usesAsIn}$"
  sertraline:
    vars:
      uses: 3
```

The `uses` is stored in the item's `sertraline_data` and retained under the default rebuild settings. Item actions can update this value and refresh the lore. The bundled `sword_of_counter` in `workspace/default/default_weapon.yml` shows how to add up damage on hit.
