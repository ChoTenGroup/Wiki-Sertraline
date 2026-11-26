# 配方 API

本文以 Sertraline 3.13.2.2 的现代构建为参考。插件开发者可以查询已注册的 Sertraline 配方、读取材料与产物定义，或在运行时注册一条配方。服主可以直接用 YAML 添加配方。[自定义配方](../jin-jie/custom-recipes.md)

## 添加编译依赖

按 [Sertraline API 的依赖说明](sertralineapi.md#添加依赖)从 Maven Central 引入 Sertraline，并添加目标平台的 Bukkit 或 Paper API。需要本地 API JAR 时，运行 `tabooBuildApi`。

在你的插件的 `src/main/resources/plugin.yml` 声明依赖。下面的声明对应本页的 Java 监听器示例。[插件依赖声明](https://docs.papermc.io/paper/dev/plugin-yml/#depend)

```yaml
name: RecipeApiExample
version: 1.0.0
main: example.sertraline.RecipeApiExample
api-version: '1.21'
depend: [Sertraline]
```

## 查询配方

通过 Java 的 `SertralineRecipeManager.INSTANCE` 或 Kotlin 的 `SertralineRecipeManager` 访问配方管理器。初次加载或重载完成后再查询。

| 方法签名（Java）                          | 返回值及用法                                                         |
| ----------------------------------- | -------------------------------------------------------------- |
| `getRecipeData(NamespacedKey key)`  | 查找已成功注册的 Sertraline 配方，找不到时返回 `null`                           |
| `getAllRecipeData()`                | 返回当前已注册配方的集合视图。按只读使用。需要保留当次查询结果时，在重载结束后用 `List.copyOf(...)` 复制 |
| `registerRecipe(RecipeData recipe)` | 注册一条运行时配方，成功时返回 `true`                                         |
| `unlockAllForPlayer(Player player)` | 尝试为指定在线玩家解锁全部已注册的 Sertraline 配方，返回解锁数量。没有新解锁或解锁失败时可以返回 `0`     |

例如，用 `getRecipeData(new NamespacedKey("sertraline", "star_from_vanilla"))` 查找随附的回忆之星配方。已有 Bukkit 事件时直接用事件里的配方键。若只需要 Bukkit 配方对象，可以调用 `Bukkit.getRecipe(key)`。[Bukkit 配方查询](https://jd.papermc.io/paper/1.21.4/org/bukkit/Bukkit.html#getRecipe\(org.bukkit.NamespacedKey\))

### 监听工作台合成

把这个类保存为 `src/main/java/example/sertraline/RecipeApiExample.java`。安装示例插件后，用工作台制作随附的回忆之星，控制台会记录 `sertraline:star_from_vanilla -> sertraline:star_of_recall`。

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

`CraftItemEvent` 适合观察合成操作。一次 Shift 点击可能处理多份材料，事件次数不能作为产物件数。`PrepareItemCraftEvent` 可用于观察产物预览，玩家调整材料时可能触发多次。[Bukkit 合成事件](https://jd.papermc.io/paper/1.21.4/org/bukkit/event/inventory/CraftItemEvent.html)

## 在运行时注册配方

下面的方法注册一条无序配方，使用铜锭和紫水晶碎片各一件，产出一个荧光墨囊。把类保存为 `src/main/java/example/sertraline/RecipeRegistration.java`。在配置加载完成后，从自己的命令或任务在服务器主线程调用 `RecipeRegistration.registerGlowInkRecipe()`。已占用的配方键或注册失败都会返回 `false`。

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

Sertraline 会为配方加上 `sertraline` 命名空间。建议用自己的插件名作路径前缀，避免与配置文件里的配方重名。上例的注册键是 `sertraline:example/glow_ink_from_copper`。玩家进服时会尝试自动解锁配方。如果需要立即刷新在线玩家的配方书，可在注册成功后于服务器主线程调用 `unlockAllForPlayer(player)`。

运行时注册只保存在内存里。服务器重启或配方配置重载后，需要重新注册。希望交给管理员长期维护的配方，可以放在 `plugins/Sertraline/recipes/`。管理器的 `unregisterAll()` 和 `commitRecipes(...)` 作用于整套配方表，请在集成代码中用单条注册接口。

## 数据模型

| 类                                      | 常用字段                                                                                     |
| -------------------------------------- | ---------------------------------------------------------------------------------------- |
| `RecipeData.Shaped`                    | `pattern` 是网格行，`key` 是字符到材料的映射                                                           |
| `RecipeData.Shapeless`                 | `ingredients` 是可无序摆放的材料列表                                                                |
| `RecipeData.Cooking`                   | `type` 选择熔炉、高炉、烟熏炉或营火，另有 `ingredient`、`experience` 和以游戏刻为单位的 `cookingTime`               |
| `RecipeData.Stonecutting`              | `ingredient` 是切石材料                                                                       |
| `RecipeData.Smithing`                  | `type` 选择锻造转化或装饰，`template`、`base` 和 `addition` 对应锻造台的三个输入槽                              |
| `RecipeIngredient.Item`、`Tag`、`Choice` | 分别表示物品、标签或任选一项。每个材料槽消耗一件                                                                 |
| `RecipeResult`                         | `itemId`、`count` 和 `functions`。工作台配方可使用 `RecipeFunction.Kether`、`JavaScript` 或 `Command` |

配方模型位于 `io.github.zzzyyylllty.sertraline.data`。物品引用沿用配置中的单冒号格式，例如 `minecraft:copper_ingot` 或 `sertraline:star_of_recall`。常规配方可选择 `RecipeProvider.SIMPLE`。选择 `RecipeProvider.COMPLEX` 时，指定物品的材料按其基础材质匹配，`exact = true` 会被拒绝。`InputOptions` 可用空构造器初始化。当前版本读取其中的配置，但材料匹配还未使用这些字段。

为工作台产物配置回调时，`RecipeFunction.Kether` 以合成玩家为发送者，`RecipeFunction.Command` 以控制台执行并将 `%player%` 替换成玩家名。`RecipeFunction.JavaScript` 的脚本没有自动注入 `player`。

### 配方加载事件

`SertralineRecipeFileLoadPreEvent` 在解析候选配方前触发，处理器应考虑它可能在异步线程上运行。`SertralineRecipeFileLoadPostEvent` 在提交候选配方后触发。其 `candidateCount` 是进入提交阶段的条目数。核对某条配方是否成功注册时，请用 `getRecipeData(key)` 查询。两类事件均继承 Sertraline 使用的 TabooLib `BukkitProxyEvent`。

物品构建、背包操作或模板处理请参阅[物品与模板 API](sertralineapi.md)。
