---
icon: wave-pulse
---

# Item Actions

Make an item run a script when a player right-clicks, attacks, or consumes it. Start by creating this compass in `plugins/Sertraline/workspace/hello_compass.yml`.

```yaml
hello_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><gold>Hello Compass"
  sertraline:
    actions:
      onRightClick:
        - async: false
          kether: tell "You used the Hello Compass!"
```

Save the file, run `/sertraline reload`, and use `/sertralineitem give hello_compass`. Hold the compass in your main hand and right-click in the air to see the message.

## Choose a trigger

| Trigger | When it runs |
| --- | --- |
| `onClick`, `onRightClick`, `onLeftClick` | Click interactions. Add `@off` for off-hand interactions |
| `onAttack` | Melee hits and projectile hits that can be traced back to a weapon. Melee uses the main-hand weapon |
| `onShoot`, `onShootTrident` | Firing a bow-like weapon or using a trident's Riptide |
| `onMine`, `onConsume`, `onBreak` | Breaking a block, eating or drinking, or breaking an item by using up its durability |
| `onDrop`, `onPickUp`, `onInventoryClick` | Dropping, picking up, or clicking an item in an inventory slot. Pickup needs a modern server with the matching event |
| `onSwap`, `onSneak` | Swapping hands or starting and stopping sneaking |
| `onLogin` | Checking both hands and all four armor slots on login |

`onMine`, `onSwap`, `onSneak`, and `onLogin` support `@main` and `@off`. For `onSwap`, the suffix is the **destination slot**. For example, `onSwap@off` uses the item moving into the off hand. Login also supports `onLogin@helmet`, `onLogin@chestplate`, `onLogin@leggings`, and `onLogin@boots`. If you configure a base trigger and its slot-specific trigger, both can run. See [Custom Events and Global Listeners](../../event-listeners.md) to use other Bukkit events.

## Add a condition or script

List one or more actions under a trigger, starting each one with `-`. To use a Kether condition, add `condition` to the same list entry as the script. Here is another way to control when the compass sends a message. Replace its existing `onRightClick` entry, then compare a normal right-click with one while sneaking.

```yaml
onRightClick:
  - async: false
    graaljs: |
      if (player.isSneaking()) {
        player.sendMessage("You used the compass while sneaking!");
      }
```

This example uses GraalJS to check whether the player is sneaking. For other script languages, use `kether`, `js` (or `javascript`), `jexl`, or `fluxon`. JavaScript can access `player`, the current `event`, the item that triggered the action as `bItem`, and [item data](item-data.md).

Actions run asynchronously by default. Set `async: false` when accessing an inventory or the world, editing damage, or cancelling an event. For a cancellable event, you can call `event.setCancelled(true)`. Set the default interval in milliseconds with `action.throttle-time` in `config.yml`. To override one trigger, add it under `action.throttle`, for example `onRightClick: 1000`. Set it to `0` to disable throttling for that trigger.

Try `sword_of_ultimate_damage` in the bundled `workspace/default/default_weapon.yml` or `speed_potion_cookie` in `default_consumable.yml` for an item that gives a speed effect when eaten. See [Script Utilities](../../../refrences/script-util/README.md) for tools you can use in scripts.
