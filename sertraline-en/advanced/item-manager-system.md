---
icon: cursor
---

# Item Manager

Use the item manager to inspect, create, copy, and delete item definitions in the selected scope. Use `/sertralineitem give` to give items. Add or edit files under `workspace/` to manage public persistent definitions.

## Inspect a loaded item

Create and reload `beginner_sword` by following [My First Item](../basics/my-first-item.md). Then run these commands in order.

```text
/sertraline manager use public
/sertraline manager list
/sertraline manager info beginner_sword
```

The first command selects the public persistent scope. The second lists the item IDs. The third shows the scope being queried, the item ID, and the loaded data and configuration keys. In game, `/sertralineitem browse` opens the item browser.

## Command reference

Put each subcommand below after `/sertraline manager`. You can also use the standalone `/manager` or `/mgr` command, for example `/mgr list`. Replace values in angle brackets with the actual arguments. Omit the leading slash in the server console.

| Subcommand | Purpose |
| --- | --- |
| `help` | Shows the manager's built-in command help |
| `use <public or private> [persistent or temporary]` | Selects the scope. Omit the second argument to select `persistent` |
| `switch <UUID>` | Selects the player's UUID for subsequent private operations |
| `list` | Lists item IDs in the current scope |
| `info <item-id>` | Shows an item's ID, query scope, data keys, and configuration keys |
| `delete <item-id>` | Deletes a definition in the current scope. Deletion in the public persistent scope requires a config setting |
| `create <item-id> <JSON>` | Creates a definition from a JSON object in a temporary or private scope |
| `clone <source-id> <new-id>` | Copies a definition's data to a new ID in a temporary or private scope |

The standalone manager command uses `sertraline.command.manager`. The main `/sertraline` command uses `sertraline.command.main`. Both grant access to operators by default. See [Common Commands](../basics/commands.md) for the arguments used to give items.

## Choose a storage scope

| Scope | Purpose |
| --- | --- |
| `public/persistent` | Shared items. Their workspace definitions are loaded again on reload |
| `public/temporary` | Shared in-memory definitions. They survive a configuration reload and expire on restart |
| `private/persistent` | Definitions stored in the database under a player's UUID |
| `private/temporary` | Definitions stored in memory under a player's UUID and cleared when the plugin shuts down |

### View another player's private items

Select the private scope, then specify the player's UUID. Replace the sample UUID below. Use the full item ID shown by `list` when looking up details. Private item IDs normally begin with `__`.

```text
/sertraline manager use private
/sertraline manager switch 550e8400-e29b-41d4-a716-446655440000
/sertraline manager list
```

Private operations in game use the current player's UUID by default. From the console, use `switch` or set `private.auto-uuid` in `manager.yml`. Running `use` clears the UUID selected with `switch`. A player's scope and UUID selection reset on logout.

When you give a private item, Sertraline looks up the full item ID under the recipient's UUID. For example, use `/sertralineitem give __quest_token Steve` to give online player Steve an item from their private scope.

### Select a temporary scope

Run `/sertraline manager use public temporary` for shared temporary items, or `/sertraline manager use private temporary` for your private temporary items. You can shorten `persistent` to `pers` or `p`, and `temporary` to `temp` or `t`.

To set the initial scope for anyone who has yet to choose one, edit `defaults` in `plugins/Sertraline/manager.yml` and reload. For example, set `scope: public` and `sub: temporary` for a public temporary default.

## Create and copy items

### Create a temporary paper item

Run these commands one line at a time. The JSON object after the item ID defines the paper item. The third command copies the item's data to a new ID.

```text
/sertraline manager use public temporary
/sertraline manager create manager_demo_paper {"xbuilder": {"material": "PAPER"}}
/sertraline manager clone manager_demo_paper manager_demo_paper_copy
/sertraline manager info manager_demo_paper_copy
```

For consecutive spaces inside a JSON string, write one `\u0020` per space. For example, use `"two\u0020\u0020spaces"`.

In game, run `/sertralineitem give manager_demo_paper_copy` to receive the copy. Both definitions expire when the server restarts.

### Add a public persistent item

Save the following in `plugins/Sertraline/workspace/manager_example.yml`.

```yaml
manager_example_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Manager Example Token"

manager_example_token_copy:
  xbuilder:
    material: PAPER
    name: "<!i><aqua>Manager Example Token Copy"
```

Run `/sertraline reload`, then use `/sertraline manager use public` and `/sertraline manager info manager_example_token_copy` to inspect the second item. You can reuse a configuration by copying it under a new top-level ID, as shown above.

### Create a private persistent item

From the console, replace the example UUID with the item owner's UUID before running these commands. In game, you can skip `switch` when creating an item for yourself.

```text
/sertraline manager use private persistent
/sertraline manager switch 550e8400-e29b-41d4-a716-446655440000
/sertraline manager create manager_demo_private {"xbuilder":{"material":"PAPER"}}
/sertraline manager info __manager_demo_private
```

Sertraline saves the definition under the selected UUID. Use the full ID shown by `list` when querying it or copying it.

The original 3.13.2.2 release has a parsing bug in the two-argument forms of `use`, `create`, and `clone`. On that release, choose a default temporary scope in `manager.yml`, add public persistent items in `workspace/`, or use the following Kether command to create the private persistent example. Replace the example UUID and enter the command on one line.

```text
/sertraline api eval smanager use private persistent smanager switch 550e8400-e29b-41d4-a716-446655440000 smanager create manager_demo_private by {"xbuilder":{"material":"PAPER"}}
```

For integrations, see the [item manager API](../references/sertralineapi.md#item-manager).

## Delete a definition

Before running `/sertraline manager delete <item-id>`, use `info` to confirm the ID and scope. Deletion in the private persistent scope removes the database record for the selected UUID. Deletion in a temporary scope removes the in-memory definition.

Deletion in the public persistent scope is disabled by default. The `allow-delete-public-persistent` option in `manager.yml` enables it. The command removes the loaded definition from memory. The `delete-file` setting does not remove the YAML. To remove a public item permanently, delete its top-level entry from the original YAML and run `/sertraline reload`. You can keep other definitions in the same file.

## Developer API

```kotlin
val api = Sertraline.api()
```

#### Public Item Operations

```kotlin
// Create a public-temporary item
api.createPublicItem("my_item", data, SubManagerType.TEMPORARY)

// Read
val item = api.getPublicItem("my_item")              // PERSISTENT (default)
val tempItem = api.getPublicItem("my_item", SubManagerType.TEMPORARY)

// Delete
api.deletePublicItem("my_item", SubManagerType.TEMPORARY)

// List
val allPersistent = api.getAllPublicItems(SubManagerType.PERSISTENT)
val allTemporary = api.getAllPublicItems(SubManagerType.TEMPORARY)
```

#### Private Item Operations

```kotlin
// Create
api.createPrivateItem("uuid-here", "my_private_item", data, SubManagerType.TEMPORARY)
api.createPrivateItem("uuid-here", "my_db_item", data, SubManagerType.PERSISTENT)

// Read
val item = api.getPrivateItem("uuid-here", "my_private_item", SubManagerType.TEMPORARY)

// Delete
api.deletePrivateItem("uuid-here", "my_private_item", SubManagerType.TEMPORARY)

// List
val all = api.getAllPrivateItems("uuid-here", SubManagerType.PERSISTENT)
```

#### General Operations

```kotlin
// Register/remove items directly in itemMap (public-persistent)
val item = ModernSItem(key = "custom_item", data = linkedMapOf(), config = linkedMapOf())
api.registerItem("custom_item", item)

val removed = api.unregisterItem("custom_item")

// Count
val total: Int = api.getItemCount()

// UUID resolution
val uuid = api.resolvePrivateUuid(null, player.uniqueId.toString())
```

#### Internal Manager Access

```kotlin
// Direct access to the underlying managers
val publicMgr = Sertraline.manager.public
val privateMgr = Sertraline.manager.privateManager
```

***
