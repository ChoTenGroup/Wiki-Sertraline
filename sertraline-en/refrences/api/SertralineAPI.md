# Sertraline API

Use `SertralineItemAPI` to look up and build items from Sertraline, Minecraft, and other item plugins. Use `Sertraline.api()` for inventory counting and removal, item rebuilding, public and private item management, and templates.

## Add the dependency

Add Sertraline from Maven Central. For Gradle Kotlin DSL, use the following.

```kotlin
repositories {
    mavenCentral()
}

dependencies {
    implementation("io.github.zzzyyylllty.sertraline:Sertraline:3.13.2.2")
}
```

For Maven, add this dependency under `<dependencies>` in `pom.xml`.

```xml
<dependency>
    <groupId>io.github.zzzyyylllty.sertraline</groupId>
    <artifactId>Sertraline</artifactId>
    <version>3.13.2.2</version>
</dependency>
```

Install the matching Sertraline runtime JAR in the server's `plugins/` directory and declare `depend: [Sertraline]` in your plugin's `plugin.yml`.

### Build the API from source

Run `./gradlew tabooBuildApi` from the Sertraline source root. On Windows, use `.\gradlew.bat tabooBuildApi`. The default build writes `project/common/build/libs/Sertraline-X.X.X.X-api.jar`, where `X.X.X.X` is the project version.

## Look up and give an item

This Java example gives a player the `beginner_sword` from [My First Item](../../basics/my-first-item.md), if it is loaded. Call it after Sertraline finishes loading its item definitions and on the thread that owns the player's inventory.

```java
import io.github.zzzyyylllty.sertraline.api.SertralineItemAPI;
import io.github.zzzyyylllty.sertraline.data.AbstractSItem;
import org.bukkit.entity.Player;
import org.bukkit.inventory.ItemStack;

public final class GiveBeginnerSword {
    public static boolean give(Player player) {
        SertralineItemAPI api = SertralineItemAPI.INSTANCE;
        AbstractSItem definition = api.getItem("beginner_sword");
        if (definition == null) {
            return false;
        }

        ItemStack sword = api.build(definition, player);
        if (sword.getType().isAir()) {
            return false;
        }

        for (ItemStack leftover : player.getInventory().addItem(sword).values()) {
            player.getWorld().dropItemNaturally(player.getLocation(), leftover);
        }
        return true;
    }
}
```

## Multi-source item API

| Method | Use |
| --- | --- |
| `getItem(id)` | Finds a registered definition. Uses a bare Sertraline ID, `minecraft::id`, or `provider::id`. Returns `null` when the key is absent |
| `getItemFromStack(itemStack, player)` | Looks up the definition represented by a stack, including private items |
| `getAllItems()` | Returns a read-only snapshot of the current item registry |
| `parse(reference)` | Resolves a reference without looking it up in the registry |
| `build(item, player)` | Builds an `ItemStack` from an `AbstractSItem`. Returns air if construction fails |
| `register(item, id)` | Registers a definition using its storage key by default. Passing an existing key replaces the entry |

Run registry writes and single-file loads after configuration reload completes. During a reload, event handlers can write through the `LoadContext` supplied with the event. Call synchronous methods that touch player inventory or data on the thread that owns the player.

For recipe lookup, events, and runtime registration, see the [Recipe API](recipe-api.md).

---

## Enums

### ManagerRange

Inventory item query scope:

- `PUBLIC` — match only public items (ID does not start with `__`)
- `PRIVATE` — match only private items (ID starts with `__`)
- `BOTH` — match both public and private items

### InventorySlotRange

Inventory slot scope:

- `STORAGE` — storage area (main inventory + hotbar)
- `ALL` — all slots (additionally includes armor slots and off-hand)

### SubManagerType

Item sub-manager type:

- `TEMPORARY` — stored in memory. Public definitions survive configuration reload. Private definitions expire on reload
- `PERSISTENT` — private definitions are stored in the database. The public scope is populated from the workspace

---

## Item Query & Build

### getItem

Deprecated compatibility method. Use SertralineItemAPI.getItem.

Get a Sertraline item object (`ModernSItem`); returns `null` if not registered.

`getItem(id: String): ModernSItem?`

### getAllItems

Deprecated compatibility method. Use SertralineItemAPI.getAllItems.

Get all registered public items as a `Map<ID, ModernSItem>`.

`getAllItems(): Map<String, ModernSItem>`

### buildItem

Build an item for a player.

`buildItem(sItem: String, player: Player?, source: ItemStack? = null, amount: Int = 1, overrideData: Map<String, Any?>? = null): ItemStack?`

- `source`: source item; `null` builds automatically.
- `overrideData`: fields that override item data.

### buildDataItem

Build an item for a player (with variables). `vars` are template variables (persisted to NBT); `context` is the runtime context (Player, Event, etc., accessible via `${context:xxx}$` item tags and `{context:xxx}` lore placeholders, available during construction).

`buildDataItem(sItem: String, player: Player?, source: ItemStack? = null, amount: Int = 1, overrideData: Map<String, Any?>? = null, vars: Map<String, Any?>? = null, context: Map<String, Any?>? = null): ItemStack?`

### getId

Get an item's Sertraline ID; returns `null` for non-Sertraline items.

`getId(itemStack: ItemStack): String?`

### getVal

Read a val value from a Sertraline item object (`sertraline:vals`); returns `null` if the key does not exist.

`getVal(sItem: ModernSItem, valId: String): Any?`

### getVar

Read a persisted var value from an item's NBT (`sertraline_data`); returns `null` if the key does not exist.

`getVar(itemStack: ItemStack, varId: String): Any?`

### isValidItem

Check whether an item is a Sertraline item (has a Sertraline ID).

`isValidItem(itemStack: ItemStack): Boolean`

### isRegisteredItemStack

Check whether the stack's Sertraline ID is registered in `itemMap`.

`isRegisteredItemStack(itemStack: ItemStack): Boolean`

### isRegisteredItemById

Check whether the given ID is registered in `itemMap`. Both original `isRegisteredItem` signatures remain available.

`isRegisteredItemById(id: String): Boolean`

---

## Player Inventory Items

The following methods match items in the inventory exactly by Sertraline ID; the private scope verifies that the ID belongs to the target player's private manager. All are synchronous APIs and must be called on the player's thread.

### countItem

Count the Sertraline items in a player's inventory.

`countItem(id: String, player: Player, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Int`

### hasItem

Check whether the player has at least the given amount of the item. Non-positive `amount` always returns `false`.

`hasItem(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Boolean`

### takeItem

Take as many of the item from the inventory as possible, returning the actually-taken amount. Non-positive `amount` returns `0`.

`takeItem(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Int`

### takeItemExactly

Take the full amount only when enough items are available. If the amount is insufficient or non-positive, the inventory is left unmodified and `false` is returned.

`takeItemExactly(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Boolean`

### countItemAnyOwner

Count the items without verifying private-item ownership (classifies by ID only).

`countItemAnyOwner(id: String, player: Player, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Int`

### hasItemAnyOwner

Check whether the amount is sufficient without verifying private-item ownership. Non-positive `amount` returns `false`.

`hasItemAnyOwner(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Boolean`

### takeItemAnyOwner

Take as many items as possible without verifying private-item ownership, returning the actually-taken amount.

`takeItemAnyOwner(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Int`

### takeItemAnyOwnerExactly

Take the full amount only when enough items are available, without verifying private-item ownership. If the amount is insufficient or non-positive, the inventory is left unmodified and `false` is returned.

`takeItemAnyOwnerExactly(id: String, player: Player, amount: Int = 1, range: ManagerRange = ManagerRange.PUBLIC, slots: InventorySlotRange = InventorySlotRange.STORAGE): Boolean`

---

## Item Rebuild

### rebuildLore

Regenerate an item's lore. **Modifies the input item in place**.

`rebuildLore(itemStack: ItemStack, player: Player?): Unit`

### rebuildName

Regenerate an item's display name. **Modifies the input item in place**.

`rebuildName(itemStack: ItemStack, player: Player?): Unit`

### rebuild

Rebuilds the entire Sertraline item and returns a new `ItemStack`.

`rebuild(itemStack: ItemStack, player: Player?): ItemStack`

### rebuildUnsafe

Rebuild an item via ItemMeta and write into the original item. This updates the input item in place and can lose some data components.

`rebuildUnsafe(itemStack: ItemStack, player: Player?): Unit`

---

## Item Manager

### createPublicItem

Create a public temporary item. For a persistent public definition, use a workspace configuration file.

`createPublicItem(id: String, data: Map<String, Any?>, sub: SubManagerType = SubManagerType.TEMPORARY): Unit`

### getPublicItem

Deprecated compatibility method. Use SertralineItemAPI.getItem.

Get a public item (reads from the persistent manager by default).

`getPublicItem(id: String, sub: SubManagerType = SubManagerType.PERSISTENT): ModernSItem?`

### deletePublicItem

Delete a public item; returns whether the deletion succeeded.

`deletePublicItem(id: String, sub: SubManagerType): Boolean`

### getAllPublicItems

Deprecated compatibility method. Use SertralineItemAPI.getAllItems.

Get all public items of the given sub-type.

`getAllPublicItems(sub: SubManagerType): Map<String, ModernSItem>`

### createPrivateItem

Create a private item for the given UUID.

`createPrivateItem(uuid: String, id: String, data: Map<String, Any?>, sub: SubManagerType = SubManagerType.TEMPORARY): Unit`

### getPrivateItem

Deprecated compatibility method. Use the private manager to read private scope.

Get a private item for the given UUID (reads from the persistent manager by default).

`getPrivateItem(uuid: String, id: String, sub: SubManagerType = SubManagerType.PERSISTENT): ModernSItem?`

### deletePrivateItem

Delete a private item for the given UUID.

`deletePrivateItem(uuid: String, id: String, sub: SubManagerType): Unit`

### getAllPrivateItems

Deprecated compatibility method. Use the private manager to read private scope.

Get all private items of the given UUID and sub-type.

`getAllPrivateItems(uuid: String, sub: SubManagerType): Map<String, ModernSItem>`

### updateSItemS

Update a private item for the given UUID and explicit ID. The persistent manager writes the update to the database. Returns the previous item, or `null` when inserting a new entry.

`updateSItemS(uuid: String, id: String, item: ModernSItem, sub: SubManagerType = SubManagerType.PERSISTENT): ModernSItem?`

### updateSItemM

Update a private item for the given UUID using `ModernSItem.key` as its ID. Persistence and return values follow `updateSItemS`.

`updateSItemM(uuid: String, item: ModernSItem, sub: SubManagerType = SubManagerType.PERSISTENT): ModernSItem?`

Kotlin callers can omit `sub`. Java and JS callers must pass a `SubManagerType` explicitly.

### registerItem

Deprecated compatibility method. Use SertralineItemAPI.register.

Register an item in the live public registry.

`registerItem(id: String, item: ModernSItem): Unit`

### updatePublicSItemS

Write an item to the public registry in `itemMap` using an explicit ID. Returns the previous Sertraline item, or `null` if the ID had no entry or held an item from another source.

`updatePublicSItemS(id: String, item: ModernSItem): ModernSItem?`

### updatePublicSItemM

Write a public registry entry using `ModernSItem.key` as its ID. Return values follow `updatePublicSItemS`.

`updatePublicSItemM(item: ModernSItem): ModernSItem?`

### updateSItem

All four original `updateSItem` signatures remain available as deprecated compatibility methods. The two-argument call `updateSItem(string, item)` writes to the public registry. Use `updateSItemM` to update a private item by its key.

### unregisterItem

Deprecated compatibility method. Use SertralineItemAPI.unregister.

Remove an item from `itemMap`; returns the removed item (`null` if not found).

`unregisterItem(id: String): ModernSItem?`

### getItemCount

Counts registered items from Sertraline, Minecraft, and external providers.

`getItemCount(): Int`

### resolvePrivateUuid

Resolve the first available UUID from the explicit argument, the player argument, or `private.auto-uuid` in `manager.yml`. Throws `IllegalStateException` when all are blank.

`resolvePrivateUuid(uuid: String?, playerUuid: String?): String`

---

## Template System

### getTemplate

Get a loaded template (no parsing); returns `null` if it does not exist.

`getTemplate(name: String): Map<String, Any?>?`

### getTemplateNames

Get the names of all loaded templates.

`getTemplateNames(): Set<String>`

### getAllTemplates

Get all loaded templates (immutable snapshot).

`getAllTemplates(): Map<String, Map<String, Any?>>`

### resolveTemplate

Manually resolve a template: deep-copy → parameter substitution → recursive resolve.

`resolveTemplate(name: String, args: Map<String, String>): Map<String, Any?>?`

### getTemplateCount

Get the number of loaded templates.

`getTemplateCount(): Int`

---

## Template Processor Registration

Used to register custom `$t` transformers, `$c` converters, and argument-level directives.

### registerTransformer

Register a custom `$t` transformer type.

`registerTransformer(type: String, provider: TemplateManager.TransformerProvider): Unit`

### unregisterTransformer

Unregister a custom `$t` transformer type.

`unregisterTransformer(type: String): Unit`

### registerConverter

Register a custom `$c` converter type.

`registerConverter(type: String, provider: TemplateManager.ConverterProvider): Unit`

### unregisterConverter

Unregister a custom `$c` converter type.

`unregisterConverter(type: String): Unit`

### registerDirective

Register a custom argument-level directive (same level as `$t`/`$c`, e.g. `$myDirective`).

`registerDirective(name: String, provider: TemplateManager.DirectiveProvider): Unit`

### unregisterDirective

Unregister a custom argument-level directive.

`unregisterDirective(name: String): Unit`
