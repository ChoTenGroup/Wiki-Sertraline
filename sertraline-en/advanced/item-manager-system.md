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


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): public temporary items stay in the in-memory cache across config reloads and are cleared at shutdown; they are not serialized to a reload buffer. Private persistent items are database-backed, while private temporary items are memory-only. The `delete-file` setting is read by the config class but has no current caller; deleting a public persistent item removes it from the loaded registry, not its YAML source file.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

---
icon: cursor
---

# Item Manager System

## Item Manager System

Sertraline's item manager provides four storage modes for items: **Public** / **Private** × **Persistent** / **Temporary**. The manager handles item creation, retrieval, deletion, and lifecycle management across reloads.

***

### Architecture

```
ItemManager
├── PublicManager          (visible to all players)
│   ├── PERSISTENT         (survives reload, in itemMap)
│   └── TEMPORARY          (survives reload via serialization buffer)
│
└── PrivateManager         (per-UUID, database-backed)
    ├── PERSISTENT         (stored in database)
    └── TEMPORARY          (in-memory only)
```

#### Storage Types

| Type                  | `SubManagerType` | Scope       | Persistence                                         |
| --------------------- | ---------------- | ----------- | --------------------------------------------------- |
| **Public Persistent** | `PERSISTENT`     | All players | itemMap (memory), cleared & reloaded from workspace |
| Public Temporary      | `TEMPORARY`      | All players | Serialized to buffer during reload, restored after  |
| Private Persistent    | `PERSISTENT`     | Per UUID    | Database                                            |
| Private Temporary     | `TEMPORARY`      | Per UUID    | In-memory, lost on shutdown                         |

#### Enums

```kotlin
enum class ManagerType(val alias: Set<String>) {
    PUBLIC(setOf("public")),
    PRIVATE(setOf("private"));
}

enum class SubManagerType(val alias: Set<String>) {
    PERSISTENT(setOf("persistent", "pers", "p")),
    TEMPORARY(setOf("temporary", "temp", "t"));
}
```

***

### Configuration (`manager.yml`)

```yaml
# Default manager type and sub type
defaults:
  scope: "public"          # public / private
  sub: "persistent"        # persistent / temporary

# Allow deleting public-persistent items
allow-delete-public-persistent: false

# Also remove the file when deleting public-persistent items
delete-file: false

# Private manager settings
private:
  # Auto UUID fallback when no player context available
  auto-uuid: ""
```

***

### Public Items

Public items are shared across all players. They are the standard item type loaded from `workspace/`.

#### Behavior

| Operation | PERSISTENT                         | TEMPORARY                                    |
| --------- | ---------------------------------- | -------------------------------------------- |
| Create    | Not supported via manager          | `createTemporary(id, item)`                  |
| Read      | `getItem(id)` → itemMap            | `getItem(id)` → temporary → itemMap fallback |
| Delete    | Remove from itemMap (config-gated) | Remove from temporary + itemMap              |
| Reload    | Cleared, reloaded from workspace   | Serialized → buffer → restore                |

#### Creation Restriction

Creating public-persistent items through the manager is blocked (`UnsupportedOperationException`). These items are loaded from YAML workspace files only. For runtime registration, use `SertralineAPI.registerItem(id, item)` or the public-temporary path.

***

### Private Items

Private items are scoped to a UUID (player or custom namespace). They support database persistence.

#### Behavior

| Operation | PERSISTENT                       | TEMPORARY      |
| --------- | -------------------------------- | -------------- |
| Create    | Cache + database insert          | In-memory only |
| Read      | Cache → database fallback        | In-memory      |
| Delete    | Cache + database delete          | In-memory only |
| Reload    | Cache cleared, lazy-load from DB | Cleared        |
| Shutdown  | Cleared                          | Cleared        |

#### UUID Resolution

When no UUID is provided, the manager resolves one in this order:

1. Explicit UUID parameter
2. Player UUID context
3. `manager.yml` `private.auto-uuid` setting
4. Throws `IllegalStateException` if none available

***

### API (via `SertralineAPI`)

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
val item = api.getPrivateItem("uuid-here", "my_private_item")

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

### Commands

#### `/sertraline manager` (`/mgr`)

Manage items across public/private, persistent/temporary scopes.

| Subcommand                                      | Description                              |
| ----------------------------------------------- | ---------------------------------------- |
| `help`                                          | Show command usage                       |
| `use <public\|private> [persistent\|temporary]` | Set manager scope                        |
| `switch <uuid>`                                 | Switch private manager UUID              |
| `create <id> <json>`                            | Create item in current scope (JSON body) |
| `clone <templateId> <newId>`                    | Clone an existing item                   |
| `delete <id>`                                   | Delete an item by ID                     |
| `list`                                          | List items in current scope              |
| `info <id>`                                     | Show item details                        |

**Examples:**

```
# Use public-temporary scope
/manager use public temp

# Use private-persistent scope
/manager use private persistent

# Switch to a specific UUID for private operations
/manager switch 550e8400-e29b-41d4-a716-446655440000

# Create a public-temporary item with JSON body
/manager create my_item {"xbuilder":{"material":"stone","name":"&aMy Item"}}

# Clone an existing item
/manager clone sword_of_skyline my_cloned_sword

# List items in current scope
/manager list

# Show item details
/manager info my_item

# Delete an item
/manager delete my_item
```

#### `/sertraline item give`

Give Sertraline items to players.

```
/sertraline item give [player] [amount] [silent]
```

**Examples:**

```
# Give yourself 1 sword_of_skyline
/sertraline item give sword_of_skyline

# Give a player 5 of the item silently
/sertraline item give sword_of_skyline 5 true
```

***

### Kether Integration

#### Kether Variables in Items

Item variables (`vars` section) support inline Kether expressions using the `${kether:...}$` syntax:

```yaml
my_item:
  xbuilder:
    name: "<!i>&7Power Level: ${kether: check get power_level}$"
    lore:
      - "<!i><gray>Current: <yellow>${var:power_level}$"
  sertraline:
    vars:
      power_level: 1
```

#### Conditional Actions with Kether

```yaml
my_item:
  sertraline:
    vars:
      uses: 0
    actions:
      onRightClick:
        - condition: check var uses < 10
          js: |
            var tag = ItemStackUtil.getItemTag(bItem);
            tag.putDeep("sertraline_data.uses", uses + 1);
            ItemStackUtil.setItemTagDirect(bItem, tag);
            player.sendMessage("Used! Remaining: " + (10 - uses - 1));
          kether: |
            tell inline "&aYou used the item!"
```

## Structure

```
reloadCustomConfig()
├── manager.preReload()
│   └── PublicManager: serialize temporary items → buffer
│
├── Clear all caches (itemMap, etc.)
├── Reload workspace items from YAML
│
├── manager.postReload()
│   └── PublicManager: deserialize buffer → restore temporary items
│
└── Reload recipes, scripts, etc.
```

During reload:

* **Public-Persistent**: wiped and reloaded from workspace YAML files
* **Public-Temporary**: serialized to string buffer before clear, deserialized after reload
* **Private-Persistent**: cache cleared, lazy-loaded from database on next access
* **Private-Temporary**: cache cleared, lost if not saved externally

</details>
