---
icon: plug-circle-plus
---

# Crafting Stations \[Expr]

Crafting station allows players to create items through recipes in the GUI interface, and you can freely define conditions such as Kether and Javascript.&#x20;

In addition, the crafting station draws inspiration from `MMOItems`, but our crafting station is more scalable, maintainable ~~and has fewer bugs~~ (such as bugs in `MMOItems` where canceling a recipe will refresh the properties of synthetic materials...)

{% hint style="warning" %}
WORK IN PROGRESS
{% endhint %}

## Crafting Station System

### Overview

The crafting station system provides a GUI-based crafting mechanism with the following features:

* Custom GUI layouts (6-row Chest interface)
* Paginated recipe browsing
* Multi-condition checks (Kether scripts, numeric comparison, permissions)
* Asynchronous crafting timers
* Lifecycle scripts (agents) — Kether / GraalJS / JSR 223 JavaScript
* Multiple item sources (vanilla, Sertraline custom items, ItemsAdder / Oraxen / Nexo / CraftEngine, etc.)
* Crafting progress persistence (recoverable on player quit / server restart)

***

### File Location

Configuration files are placed under `crafting-stations/` in the server directory:

```
plugins/Sertraline/crafting-stations/
  ├── craftingStation.yml
  └── ... (any filename, supports yaml/yml/toml/json)
```

If the folder does not exist on first startup, a sample file is generated automatically.

***

### Configuration Structure

#### Top-Level Structure

```yaml
<station_id>:
  option:                          # Station options
    enable-tracker: true
  display:                         # GUI layout config
    title: "..."
    layout: [...]
    key: "..."
    elements: {...}
  recipes:                         # Recipe list
    <recipe_id>: {...}
  agents:                          # Station-level lifecycle scripts
    onStart: |-
      ...
    onCancel: |-
      ...
    onClaim: |-
      ...
```

***

### Display — GUI Layout

#### Fields

| Field      | Type          | Description                                                 |
| ---------- | ------------- | ----------------------------------------------------------- |
| `title`    | String        | GUI title, supports MiniMessage format                      |
| `layout`   | List\<String> | Row string array, 9 chars per row, N rows (N-row Chest GUI) |
| `key`      | String        | Single character marking recipe item display positions      |
| `elements` | Map           | Background/navigation element definitions                   |

#### layout Example

```yaml
layout:
  - "#########"
  - "#IIIIIII#"
  - "#IIIIIII#"
  - "#IIIIIII#"
  - "#IIIIIII#"
  - "##<###>##"
key: "I"
```

* `#` — Maps to the `background` element in `elements`
* `I` — Defined by the `key` field, marks recipe item fill positions (auto-paginated)
* `<`, `>` — Maps to page navigation elements

#### Elements

| Field      | Type                 | Description                                                              |
| ---------- | -------------------- | ------------------------------------------------------------------------ |
| `char`     | String (single char) | Character used in layout (for background-type elements)                  |
| `slot`     | Int                  | Raw slot index (0-53), for elements not positioned via layout characters |
| `material` | String               | Item material name (Bukkit Material)                                     |
| `name`     | String               | Display name, supports MiniMessage                                       |
| `lore`     | String               | Lore text, `\n` for line breaks, supports MiniMessage                    |
| `roll`     | Boolean              | Whether page buttons wrap around (back to first page from last)          |
| `agents`   | Map                  | Click scripts, keyed by `onLeft` etc.                                    |

**Placeholders** (usable in name / lore):

| Placeholder       | Description                   |
| ----------------- | ----------------------------- |
| `{page}`          | Current page number (1-based) |
| `{total_pages}`   | Total page count              |
| `{next_page}`     | Next page number              |
| `{previous_page}` | Previous page number          |

***

### Recipes

```yaml
recipes:
  <recipe_id>:
    displayName: "..."            # Recipe display name
    plural: "..."                 # Plural form (for messages)
    displayTime: "1~10s"         # Display time in GUI (leave empty to use actual `time`)
    time: "{{ random 1 to 10 }}" # Crafting time, supports Kether expression evaluation

    options:
      cancellable: false          # Whether it can be cancelled (default true)

    conditions:                   # Condition list (optional)
      - name: "..."
        type: Scaled              # Type: Scaled / boolean / kether / permission
        amount: "%player_health%"
        required: "20"
        condition: "..."
        agents:
          onSuccess: |-
            ...
          onFail: |-
            ...

    messages:                     # Condition failure message overrides (optional)
      ConditionNotMetScaled: "..."

    display:                      # Display item in GUI (optional, defaults to first output)
      item: "..."                 # OmniItem
      parameters: {...}

    inputs:                       # Input item list
      - item: "minecraft:paper"
        displayName: "Paper"
        plural: "Papers"

    outputs:                      # Output item list
      - item: "sertraline:doe_card"
        parameters:
          name: "..."

    agents:                       # Recipe-level lifecycle scripts
      onStart: |-
        ...
      onCancel: |-
        ...
      onClaim: |-
        ...
```

#### Crafting Time

The `time` field supports two formats:

1. **Kether expression** (recommended): `"{{ random 1 to 10 }}"` — random 1\~10 seconds per craft
2. **Numeric**: `"30"` — fixed 30 seconds

The system attempts Kether evaluation first, falling back to `toDouble()` on failure.

#### Condition Types

| Type                 | Description                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Scaled`             | Numeric comparison. `amount` and `required` are both evaluated via Kether, then compared as `amount >= required` |
| `boolean` / `kether` | Executes `condition` as a Kether expression, expects true/false                                                  |
| `permission`         | Checks player permission: `player.hasPermission(condition)`                                                      |

#### Condition Message Overrides

Failure messages can be overridden per type in the recipe's `messages` section:

```yaml
messages:
  ConditionNotMetScaled: "Need {condition_amount}/{condition_required}, insufficient"
  ConditionNotMetPermission: "Missing permission: {condition_condition}"
```

If not overridden, defaults from `config.yml` are used.

Available placeholders: `{condition_name}`, `{condition_amount}`, `{condition_required}`, `{condition_type}`

***

### Item References (OmniItem)

Items in inputs, outputs, and display use the unified `OmniItem` format:

```yaml
item: "<source>:<id>"
parameters:
  name: "..."
  lore:
    - "..."
  item-model: "..."
components:       # 1.20.5+ Data Component
  minecraft:custom_name: "..."
amount: "1"
```

#### Item Sources

| Source                                                | Description             | Example                       |
| ----------------------------------------------------- | ----------------------- | ----------------------------- |
| `mc` / `minecraft` / `vanilla`                        | Vanilla items           | `item: "minecraft:diamond"`   |
| `sertraline` / `depazitems`                           | Sertraline custom items | `item: "sertraline:doe_card"` |
| Other (`itemsadder`, `oraxen`, `nexo`, `craftengine`) | External plugin items   | `item: "itemsadder:ruby"`     |

Omitting source defaults to `mc`.

***

### Agents — Script System

Agents support three script languages, auto-detected by prefix:

| Prefix                | Engine                                           | Example                            |
| --------------------- | ------------------------------------------------ | ---------------------------------- |
| No prefix             | **Kether** (default, backward compatible)        | `player.sendMessage("hello")`      |
| `gjs:` / `graaljs:`   | **GraalJS** (GraalVM Polyglot, ES2023+)          | `gjs: player.sendMessage("hello")` |
| `js:` / `javascript:` | **Standard JSR 223** (Nashorn / GraalVM JSR 223) | `js: player.sendMessage("hello")`  |

#### Available Variables

| Variable                  | Type                 | Scope                                |
| ------------------------- | -------------------- | ------------------------------------ |
| `player` / `bukkitPlayer` | `Player`             | All agents                           |
| `recipe`                  | String (displayName) | Recipe-level / station-level agents  |
| `station_id`              | String               | Station-level / element-level agents |
| `page`                    | Int                  | Element-level agents                 |
| `total_pages`             | Int                  | Element-level agents                 |

#### JS Utility Classes

The GraalJS and JSR 223 JS environments additionally inject a `defaultData` global, including:

* `PlayerUtil`, `ItemStackUtil`, `EventUtil`, `ThreadUtil`
* `Bukkit`, `Math`, `System`, `Gson`
* `SertralineAPI`, `ExternalItemHelper`, `FluxonShell`
* `UUIDUtil`, `scheduler`, `bukkitPlugin`

#### Lifecycle Hooks

| Hook               | Trigger                                  | Available At                 |
| ------------------ | ---------------------------------------- | ---------------------------- |
| `onStart`          | After craft begins (before timer starts) | Recipe-level → Station-level |
| `onClaim`          | After craft completes, outputs given     | Recipe-level → Station-level |
| `onCancel`         | Craft cancelled or interrupted           | Recipe-level → Station-level |
| `onSuccess`        | Condition check passes                   | Condition-level              |
| `onFail`           | Condition check fails                    | Condition-level              |
| `onLeft` / any key | GUI element clicked                      | Element-level                |

**Execution order** (normal craft flow):

```
Click recipe → condition check → onSuccess/onFail → consume items
  → execute onStart (recipe-level → station-level)
  → wait for crafting time → produce outputs
  → execute onClaim (recipe-level → station-level)
```

**Cancel flow**:

```
→ onCancel agents execute (recipe-level → station-level)
→ materials returned to inventory
→ DB session cleared
```

***

### Commands

| Command                           | Description                             |
| --------------------------------- | --------------------------------------- |
| `/sertraline station <stationId>` | Open crafting station GUI               |
| `/sertraline debug getStations`   | View all loaded station configs         |
| `/sertraline reload`              | Reload all configs (including stations) |

***

### Data Persistence

The crafting station system uses TabooLib DataContainer to persist player crafting progress.

#### Stored Data

* `crafting.session` — Session metadata: `stationId|recipeId|startTime|totalSeconds`
* `crafting.items.0`, `crafting.items.1`, ... — Base64 serialized consumed items

#### Recovery Flow

```
Player quits → cancelPlayerSession() cancels in-memory tasks, DB preserved
Server restarts → shutdownAll() cancels all tasks, DB preserved
Player joins → onJoin → restoreSession():
  ├─ Craft complete → give outputs + execute onClaim agents
  └─ Craft incomplete → return materials + execute onCancel agents
  → Clear DB session
```

#### Reload Flow

```
/sertraline reload:
  → cancelAll(): sync cancel + return items + clear DB (blocking wait)
  → craftingStations.clear()
  → loadCraftingStationFiles()
```

***

### Examples

#### Basic Crafting Station

```yaml
simple_workbench:
  display:
    title: "<gold>Simple Workbench"
    layout:
      - "##########"
      - "#IIIIIII#"
      - "#IIIIIII#"
      - "##<###>##"
    key: "I"
    elements:
      background:
        char: "#"
        material: GRAY_STAINED_GLASS_PANE
        name: ""
      next_page:
        slot: 50
        roll: true
        material: ARROW
        name: "<green>Next Page {next_page}"
      previous_page:
        slot: 48
        roll: true
        material: ARROW
        name: "<green>Previous Page {previous_page}"

  recipes:
    iron_to_diamond:
      displayName: "Iron → Diamond"
      time: "5"

      inputs:
        - item: "minecraft:iron_ingot"
          displayName: "Iron Ingot"
          amount: "32"

      outputs:
        - item: "minecraft:diamond"
          amount: "1"

      agents:
        onClaim: |-
          player.sendMessage("&aConversion complete! Got Diamond x1")
```

#### GraalJS Crafting Station

```yaml
advanced_station:
  display:
    title: "<gradient:gold:red>Advanced Station"
    layout:
      - "#########"
      - "#IIIIIII#"
      - "#IIIIIII#"
      - "#IIIIIII#"
      - "#IIIIIII#"
      - "##<###>##"
    key: "I"
    elements:
      background:
        char: "#"
        material: BLACK_STAINED_GLASS_PANE
        name: ""
      next_page:
        slot: 50
        roll: true
        material: ARROW
        name: "<green>Page {next_page}"
        agents:
          onLeft: |-
            gjs: player.sendMessage("Turned to page " + (page + 1))

  recipes:
    legendary_sword:
      displayName: "Legendary Sword"
      time: "{{ random 30 to 60 }}"
      options:
        cancellable: false

      conditions:
        - name: "Level ≥ 50"
          type: kether
          condition: "check level >= 50"

      inputs:
        - item: "minecraft:netherite_ingot"
          displayName: "Netherite Ingot"
          amount: "8"
        - item: "minecraft:diamond"
          displayName: "Diamond"
          amount: "32"

      outputs:
        - item: "sertraline:legendary_sword"
          amount: "1"

      agents:
        onStart: |-
          gjs: PlayerUtil.sendMessage(player, "Forging Legendary Sword, estimated " + time + " seconds...")
        onClaim: |-
          gjs:
            var msg = "§6★ Congratulations! You got the Legendary Sword! ★";
            player.sendMessage(msg);
            Bukkit.broadcastMessage(player.getName() + " forged the Legendary Sword!");
```

***

### Debugging

Enable `debug: true` in `config.yml` for detailed load logs.

Use `/sertraline debug getStations` to inspect all loaded station configs.

#### Common Issues

| Symptom                      | Diagnosis                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------- |
| GUI shows air items          | Check `material` spelling in `elements`                                      |
| Recipes not showing          | Check if `key` character conflicts with element `char` (warning on startup)  |
| Craft won't start            | Check if all `conditions` are met, verify input items match                  |
| JS scripts not executing     | Verify prefix is `gjs:` or `js:`, confirm GraalJS / Nashorn engine is loaded |
| Materials lost after restart | Check `restoreSession` logs, verify DataContainer storage is functioning     |
