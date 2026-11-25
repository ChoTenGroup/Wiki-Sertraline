# Custom Events and Global Listeners

Register extra item triggers in `custom-events.yml`, or add rules for the whole server in `global-listeners/`.

## React to a change in experience level

Add this top-level entry to `plugins/Sertraline/custom-events.yml`. Keep any existing entries and give the trigger a unique name.

```yaml
onPracticeLevelChange:
  class: PlayerLevelChangeEvent
  item-source:
    - main-hand
```

Create `workspace/practice_compass.yml` and use the same `onPracticeLevelChange` name in the item's actions:

```yaml
practice_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><aqua>Practice Compass"
  sertraline:
    actions:
      onPracticeLevelChange:
        - async: false
          graaljs: |
            player.sendMessage("Your new level is " + event.getNewLevel());
```

Reload, collect the compass with `/sertralineitem give practice_compass`, and hold it in your main hand. Gain an experience level or run `/experience add @s 1 levels` on a modern test server. The new level should appear in chat. Put the compass away and repeat the level change to confirm that the item no longer fires.

`class` selects the event. `item-source` tells Sertraline where to look for items, such as `off-hand`, `both-hands`, `armor`, or `inventory`. See the bundled `custom-events.yml` for `custom-source` examples. Choose a new trigger name because built-in action names are reserved.

## Welcome players when they join

Save this as `global-listeners/practice_welcome.yml`. The example uses GraalJS on a modern build.

```yaml
practice_welcome:
  bind: PlayerJoinEvent
  async: false
  eval:
    graaljs: |
      player.sendMessage("Welcome to the server!");
```

Run `/sertraline reload`, then reconnect to see the message. Global listeners respond to their bound event without an item. They skip cancelled events by default. Use `async: false` when accessing Bukkit objects or changing an event.
