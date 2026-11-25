---
icon: cursor
---

# Item Manager

Use the item manager to inspect loaded item definitions. Edit files in the workspace to add or change the items your server uses.

## Inspect a loaded item

Create `beginner_sword` and reload by following [My First Item](../basics/my-first-item.md). These commands list the sword and show its scope and loaded configuration keys.

```text
/sertraline manager use public
/sertraline manager list
/sertraline manager info beginner_sword
```

In game, you can also use `/sertralineitem browse` to view loaded items.

## Choose a scope

| Arguments | Use |
| --- | --- |
| `public persistent` | Shared items created in `workspace/` files and read again on reload |
| `public temporary` | Shared temporary definitions. They survive a config reload and expire on server restart |
| `private persistent` | Definitions scoped to a UUID and stored in the database |
| `private temporary` | Definitions scoped to a UUID and stored in memory until reload or restart |

Use `/sertraline manager use public` or `/sertraline manager use private` to select the corresponding persistent scope. The 3.13.2.2 command parser rejects two-argument forms such as `use public temporary`. A private scope uses the current player's UUID by default. Use `/sertraline manager switch <UUID>` to choose another UUID. The console can also use the `private.auto-uuid` fallback in `manager.yml`.

In `manager.yml`, `defaults.scope` and `defaults.sub` set the starting scope. Command deletion for public persistent items is disabled by default. To remove a definition permanently, remove its top-level entry from the original file and reload. Check for other definitions in the same file before enabling `delete-file`.

You can also use `/manager` or `/mgr`. See [Common Commands](../basics/commands.md) for the arguments used to give items.

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
