# Custom Events and Global Listeners

Use `custom-events.yml` when a Sertraline item should react to an additional event. Use a file in `global-listeners/` for behavior that should run server-wide without requiring a Sertraline item.

## Choose an event class

Both systems register Bukkit event classes. Built-in Bukkit/Paper events can often be written with a short name such as `PlayerJoinEvent`; you can also use the fully qualified Java class name, including its package.

{% hint style="success" %}
For events from another plugin, use the event's **fully qualified class name** in `class` or `bind`, for example `com.vendor.plugin.event.SomeEvent` (replace this with the real package and class). The short-name lookup only searches known Bukkit/Paper packages, not arbitrary plugin packages. The other plugin must be installed, and Sertraline must be able to load its event class at runtime. The target class must extend Bukkit's `org.bukkit.event.Event`.
{% endhint %}

Find the exact class name in the other plugin's API/Javadocs or source. If the listener is not registered after reload, check the server log for a class-loading or event-class error, and verify the plugin version and package name.

## React to an event with an item

Add a uniquely named entry at the top level of `plugins/Sertraline/custom-events.yml`. This example fires when a player changes worlds:

```yaml
onPracticeWorldChange:
  class: org.bukkit.event.player.PlayerChangedWorldEvent
  item-source:
    - main-hand
```

Use the same trigger name under the item's actions. For example, save this as `workspace/practice_compass.yml`:

```yaml
practice_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><aqua>Practice Compass"
  sertraline:
    actions:
      onPracticeWorldChange:
        - async: false
          graaljs: |
            player.sendMessage("You left " + event.getFrom().getName() + ".");
```

Reload with `/sertraline reload`, give yourself the item with `/sertralineitem give practice_compass`, and change worlds while holding it. The message should name the world you left.

`class` is the event class; `listen` is also accepted as a compatibility alias. `priority` and `accept-cancelled` work like they do for global listeners. If omitted, priority is `NORMAL` and cancelled events are ignored. Trigger names must be unique and must not reuse a built-in action name.

`item-source` chooses where Sertraline looks for the triggering item. It defaults to both hands and also accepts `main-hand`, `off-hand`, `both-hands`, `armor`, `inventory`, or `event`. A third-party event may not expose a player or item in a way Sertraline can infer. In that case, use `custom-source` with event getter/field paths to select them explicitly; when present, `custom-source` overrides `item-source`:

```yaml
onPluginItemUse:
  class: com.vendor.plugin.event.PluginItemEvent
  custom-source:
    player: event.player
    bItem: event.item
```

Replace the example class and paths with the actual event API. `custom-source` must resolve a player and an `ItemStack` (or dropped item) for the item action to run. See the bundled `custom-events.yml` for more examples, including an attacker/item mapping.

## Run a server-wide listener

Save this as `plugins/Sertraline/global-listeners/practice_welcome.yml`. This listener uses the fully qualified name for Bukkit's join event:

```yaml
practice_welcome:
  bind: org.bukkit.event.player.PlayerJoinEvent
  async: false
  eval:
    graaljs: |
      player.sendMessage("Welcome to the server!");
```

Run `/sertraline reload`, then reconnect to see the message. A global listener does not require an item. Each top-level key in a listener file is a listener entry; `bind` is the preferred event-class key (`class` and `listen` are accepted aliases).

| Field | Purpose |
| --- | --- |
| `bind` | Event class: a Bukkit/Paper short name or a fully qualified class name. |
| `priority` | `LOWEST`, `LOW`, `NORMAL`, `HIGH`, `HIGHEST`, or `MONITOR`; defaults to `NORMAL`. Use `MONITOR` only to observe an event, not to change it. |
| `accept-cancelled` | Defaults to `false`, so already-cancelled events are skipped. Set to `true` to receive them. |
| `async` | Defaults to `false`. `true` runs scripts off the main thread; use it only for work that does not touch Bukkit objects or the event. |
| `condition` | Optional Kether condition; the script is skipped when it evaluates to false. |
| `throttle` | Optional minimum interval in milliseconds. Useful for frequent events such as movement; the limit is tracked per listener and resolved player, or per listener when no player is resolved. |
| `eval` | Script engines and script text. Supported keys include `graaljs`, `javascript`/`js`, `kether`, `fluxon`/`fx`, and `jexl`. |

Scripts receive `event`, `cancellableEvent`, `player`, and `listener`, along with Sertraline's standard script data. `event` is the event object. `cancellableEvent` is null for a non-cancellable event, and `player` may be null when Sertraline cannot find a player on that event. Check those values before using them in a script.

{% hint style="warning" %}
Keep `async: false` when reading or changing inventories, worlds, players, or event state (including cancelling an event). An event without a player can still be handled globally through `event`, but item actions need a player and a triggering item to be resolved.
{% endhint %}
