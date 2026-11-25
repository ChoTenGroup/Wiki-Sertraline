# Recipe API

This reference covers modern Sertraline 3.13.2.2 builds. Plugin developers can look up registered Sertraline recipes, read their definitions, and register recipes at runtime. Server owners can define recipes in YAML. [Custom Recipes](../advanced/custom-recipes.md)

## Add the compile-time dependency

Run `./gradlew :project:common:apiJar` from the Sertraline source tree. Use `gradlew.bat` on Windows. Add `project/common/build/libs/Sertraline-3.13.2.2-api.jar` and the target Bukkit or Paper API to your compile classpath. Use `compileOnly` in Gradle. Install the matching Sertraline runtime JAR in the server's `plugins/` directory.

Declare the dependency in `src/main/resources/plugin.yml`. This example matches the Java listener below. [Plugin dependency declaration](https://docs.papermc.io/paper/dev/plugin-yml/#depend)

```yaml
name: RecipeApiExample
version: 1.0.0
main: example.sertraline.RecipeApiExample
api-version: '1.21'
depend: [Sertraline]
```

## Look up recipes

Use `SertralineRecipeManager.INSTANCE` in Java or `SertralineRecipeManager` in Kotlin. Query recipes after initial loading or a reload completes.

| Java method signature               | Return value and use                                                                                                                                                         |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `getRecipeData(NamespacedKey key)`  | Looks up a successfully registered Sertraline recipe. Returns `null` when the key is absent                                                                                  |
| `getAllRecipeData()`                | Returns a live view of registered recipe definitions. Treat it as read-only. Use `List.copyOf(...)` after loading if you need to keep the current result                     |
| `registerRecipe(RecipeData recipe)` | Registers a runtime recipe and returns `true` on success                                                                                                                     |
| `unlockAllForPlayer(Player player)` | Attempts to unlock all registered Sertraline recipes for an online player and returns the unlock count. Can return `0` when nothing is newly unlocked or the operation fails |

For example, `getRecipeData(new NamespacedKey("sertraline", "star_from_vanilla"))` looks up the bundled Star of Recall recipe. If you already have a Bukkit recipe event, use its recipe key. When you only need the Bukkit recipe object, call `Bukkit.getRecipe(key)`. [Bukkit recipe lookup](https://jd.papermc.io/paper/1.21.4/org/bukkit/Bukkit.html#getRecipe\(org.bukkit.NamespacedKey\))

### Observe a crafting table recipe

Save this class as `src/main/java/example/sertraline/RecipeApiExample.java`. Install your example plugin and craft the bundled Star of Recall at a crafting table. The console logs `sertraline:star_from_vanilla -> sertraline:star_of_recall`.

```java
package example.sertraline;

import io.github.zzzyyylllty.sertraline.data.RecipeData;
import io.github.zzzyyylllty.sertraline.util.SertralineRecipeManager;
import org.bukkit.Keyed;
import org.bukkit.event.EventHandler;
import org.bukkit.event.EventPriority;
import org.bukkit.event.Listener;
import org.bukkit.event.inventory.CraftItemEvent;
import org.bukkit.plugin.java.JavaPlugin;

public final class RecipeApiExample extends JavaPlugin implements Listener {
    @Override
    public void onEnable() {
        getServer().getPluginManager().registerEvents(this, this);
    }

    @EventHandler(priority = EventPriority.MONITOR, ignoreCancelled = true)
    public void onCraft(CraftItemEvent event) {
        if (!(event.getRecipe() instanceof Keyed keyed)) {
            return;
        }

        RecipeData recipe = SertralineRecipeManager.INSTANCE.getRecipeData(keyed.getKey());
        if (recipe == null) {
            return;
        }

        getLogger().info(keyed.getKey() + " -> " + recipe.getResult().getItemId());
    }
}
```

`CraftItemEvent` observes a crafting action. A shift-click may process multiple batches, so the event count cannot be used as the output quantity. Use `PrepareItemCraftEvent` to observe previews, which can update more than once as the player moves ingredients. [Bukkit craft event](https://jd.papermc.io/paper/1.21.4/org/bukkit/event/inventory/CraftItemEvent.html)

## Register a recipe at runtime

This method adds a shapeless recipe that turns one copper ingot and one amethyst shard into one glow ink sac. Save the class as `src/main/java/example/sertraline/RecipeRegistration.java`. After configuration loading completes, invoke `RecipeRegistration.registerGlowInkRecipe()` on the server thread from your own command or task. It returns `false` if the key is already in use or registration fails.

```java
package example.sertraline;

import io.github.zzzyyylllty.sertraline.data.InputOptions;
import io.github.zzzyyylllty.sertraline.data.RecipeData;
import io.github.zzzyyylllty.sertraline.data.RecipeIngredient;
import io.github.zzzyyylllty.sertraline.data.RecipeProvider;
import io.github.zzzyyylllty.sertraline.data.RecipeResult;
import io.github.zzzyyylllty.sertraline.util.SertralineRecipeManager;
import java.util.List;
import org.bukkit.Bukkit;
import org.bukkit.NamespacedKey;

public final class RecipeRegistration {
    private RecipeRegistration() {}

    public static boolean registerGlowInkRecipe() {
        if (!Bukkit.isPrimaryThread()) {
            throw new IllegalStateException("Register recipes on the server thread");
        }

        String id = "example/glow_ink_from_copper";
        NamespacedKey key = new NamespacedKey("sertraline", id);
        SertralineRecipeManager manager = SertralineRecipeManager.INSTANCE;
        if (manager.getRecipeData(key) != null || Bukkit.getRecipe(key) != null) {
            return false;
        }

        RecipeData recipe = new RecipeData.Shapeless(
            id,
            new RecipeResult("minecraft:glow_ink_sac", 1, List.of()),
            new InputOptions(),
            RecipeProvider.SIMPLE,
            List.of(
                new RecipeIngredient.Item("minecraft:copper_ingot", 1, false),
                new RecipeIngredient.Item("minecraft:amethyst_shard", 1, false)
            ),
            true,
            null
        );

        return manager.registerRecipe(recipe);
    }
}
```

Sertraline registers these recipes in the `sertraline` namespace. Use your add-on name as a path prefix to avoid colliding with recipes defined in YAML. The registered key in this example is `sertraline:example/glow_ink_from_copper`. Sertraline attempts to unlock recipes when a player joins. To refresh the book for an online player after a successful registration, call `unlockAllForPlayer(player)` on the server thread.

Runtime registrations are stored in memory. Register them again after a restart or a recipe configuration reload. Use `plugins/Sertraline/recipes/` for recipes that administrators should maintain on disk. The manager's `unregisterAll()` and `commitRecipes(...)` affect the entire Sertraline recipe registry. Use the single-recipe registration method for add-on integrations.

## Data model

| Class                                    | Common fields                                                                                                                                    |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `RecipeData.Shaped`                      | `pattern` supplies the grid rows and `key` maps characters to ingredients                                                                        |
| `RecipeData.Shapeless`                   | `ingredients` lists items that can be placed in any order                                                                                        |
| `RecipeData.Cooking`                     | `type` selects the furnace, blast furnace, smoker, or campfire. Other fields include `ingredient`, `experience`, and `cookingTime` in game ticks |
| `RecipeData.Stonecutting`                | `ingredient` supplies the stonecutter input                                                                                                      |
| `RecipeData.Smithing`                    | `type` selects smithing transformation or trim. `template`, `base`, and `addition` select the three inputs                                       |
| `RecipeIngredient.Item`, `Tag`, `Choice` | Represent an item, tag, or alternatives. Each ingredient slot consumes one item                                                                  |
| `RecipeResult`                           | Holds `itemId`, `count`, and `functions`. Crafting table recipes can use `RecipeFunction.Kether`, `JavaScript`, or `Command`                     |

The models are in `io.github.zzzyyylllty.sertraline.data`. Recipe item IDs use the same single-colon format as YAML, such as `minecraft:copper_ingot` or `sertraline:star_of_recall`. Use `RecipeProvider.SIMPLE` for regular recipes. With `RecipeProvider.COMPLEX`, concrete ingredients match by base material, and the provider rejects `exact = true`. Use the no-argument `InputOptions` constructor for the current defaults. The loader reads these fields, but ingredient matching does not use them.

For crafting-table result callbacks, `RecipeFunction.Kether` runs with the crafter as its sender. `RecipeFunction.Command` runs as the console and substitutes the crafter’s name for `%player%`. `RecipeFunction.JavaScript` runs without a provided `player` binding.

### Recipe loading events

`SertralineRecipeFileLoadPreEvent` fires before candidate parsing and can run off the main thread. `SertralineRecipeFileLoadPostEvent` fires after the candidate registration attempt. Its `candidateCount` counts entries submitted for registration. Use `getRecipeData(key)` to confirm that a specific recipe was registered. Both events extend the TabooLib `BukkitProxyEvent` used by Sertraline.

For item construction, inventory operations, and templates, see the [Sertraline API reference](sertralineapi.md).
