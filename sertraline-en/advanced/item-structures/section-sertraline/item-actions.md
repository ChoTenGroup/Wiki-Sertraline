---
icon: wave-pulse
---

# Item Actions

Under `actions`, give each trigger a list. Save this example in `workspace/guide_signal.yml`, reload, and run `/sertralineitem give guide_signal`. Right-click with the compass in your main hand or off hand. In the main hand, the condition lets the first message run, and sneaking enables a second message.

```yaml
guide_signal:
  xbuilder:
    material: COMPASS
  sertraline:
    actions:
      onRightClick:
        - async: false
          condition:
            - "check 1 == 1"
          kether: 'tell "Signal received."'
        - async: false
          graaljs: |
            if (player.isSneaking()) {
              player.sendMessage("Sneaking signal.");
            }
      "onRightClick@off":
        - async: false
          kether:
            - 'tell "Off-hand signal."'
```

## Built-in triggers

| Trigger | When it runs and which item it reads |
| --- | --- |
| `onClick`, `onRightClick`, `onLeftClick` | Interact with a block or the air. Use `@off` for off hand. Main hand uses the bare trigger name |
| `onAttack` | Uses the main-hand weapon for melee or a recorded weapon for projectile impact |
| `onShoot` | Uses the weapon when firing a bow or similar weapon |
| `onShootTrident` | Uses the trident when Riptide triggers, on builds with that event |
| `onMine` | Checks both hands on block break. Supports `@main` and `@off` |
| `onConsume` | Uses the item being eaten, drunk or otherwise consumed |
| `onBreak` | Uses the item that breaks when its durability runs out |
| `onDrop` | Uses the dropped item |
| `onPickUp` | Uses the item being picked up, on builds with the pickup event |
| `onInventoryClick` | Uses the item in the clicked inventory slot |
| `onSwap` | Uses the two swapped hand items. `@main` and `@off` describe their destinations |
| `onSneak` | Checks both hands when entering or leaving sneak. Supports `@main` and `@off` |
| `onLogin` | Checks equipment on login. Supports `@main`, `@off`, `@helmet`, `@chestplate`, `@leggings` and `@boots` |

A base trigger and its suffixed variant can both run for the same event. A base trigger that checks multiple slots can run for multiple items. For other Bukkit events, see [Custom Events and Global Listeners](../../event-listeners.md).

## Conditions, scripts and execution

| Action entry field | How to use it |
| --- | --- |
| `condition`, `require`, `requirement`, `requirements` | One Kether condition or a list. All conditions must pass. Omit it to run unconditionally |
| `async` | Defaults to `true`. Use `false` to access the inventory or world, change damage, or cancel an event |
| `kether` / `ke` | A Kether string or list of script lines |
| `javascript` / `js` | JavaScript text. Use `\|` for multiple lines |
| `jexl` / `je` | JEXL text |
| `fluxon` / `fl` | Fluxon text |
| `graaljs` / `gjs` | GraalJS text |
| `mythic` / `mm` | With MythicMobs, use a mechanic string or list. Append `~trigger` to choose a MythicMobs trigger |

You can set several script fields in one entry. Once the condition passes they run in the order Kether, JavaScript, JEXL, Fluxon, GraalJS, and MythicMobs. To try several forms, replace the existing `onRightClick` under `sertraline.actions` with this snippet.

```yaml
onRightClick:
  - async: false
    kether: 'tell "Kether"'
    js: 'player.sendMessage("JavaScript");'
    jexl: 'player.sendMessage("JEXL");'
    fluxon: |
      import 'sertraline:bukkit'
      &player::sendMessage("Fluxon")
    graaljs: 'player.sendMessage("GraalJS");'
```

Scripts can read `player`, `event`, `bItem` (the actual stack), `sItem` (the definition), and keys from item data. For a cancellable event, use `cancellableEvent`. For example, set `async: false` and `js: 'cancellableEvent.setCancelled(true);'` in `onDrop` to prevent dropping.

Set the default interval in milliseconds with `action.throttle-time` in `config.yml`. Override per base trigger under `action.throttle`, such as `onRightClick: 1000`. Set it to `0` to turn off throttling for that trigger. Find more examples in `workspace/default/default_weapon.yml` and `default_consumable.yml`, or see [Item Data](item-data.md) to save a variable from an action.
