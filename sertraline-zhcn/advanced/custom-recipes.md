---
icon: wrench
---

# 自定义配方

下面先看默认配置中的配方。确认 `workspace/default/default_misc.yml` 中存在 `star_of_recall`，然后在 `plugins/Sertraline/recipes/sample.yml` 中找到 `star_from_vanilla`：

```yaml
star_from_vanilla:
  provider: SIMPLE
  format: SHAPED
  type: CRAFTING
  pattern:
    - "PPP"
    - "PGP"
    - "PPP"
  ingredients:
    P: "minecraft:paper"
    G: "minecraft:gold_ingot"
  result:
    id: sertraline:star_of_recall
    count: 1
```

重载后，在工作台将一块金锭放在中央，周围放八张纸。`pattern` 表示摆放形状，`ingredients` 将字符对应到材料，`result` 是产物。`minecraft:` 表示原版物品，`sertraline:` 表示 Sertraline 物品；显式前缀可以避免同名歧义。这是插件自带的配方；如果想修改它，请直接编辑该条目，避免再创建一个相同配方。

{% hint style="info" %}
以下参考内容请结合安装版本核对。配置请优先参照安装版本自带的 `recipes/sample.yml`；其中 `COMPLEX` 示例被注释并标为尚未支持。
{% endhint %}


自定义配方让您可以为Sertraline物品自定义配方！

## 配方系统

### 概述

配方系统使用原版机制（工作台、熔炉、高炉、烟熏炉、篝火、切石机和锻造台）来注册自定义的Minecraft配方。它**不**使用合成台系统的自定义GUI。

主要特点

* **8种配方类型**：合成（有序/无序）、熔炉、烧炼、熏制、篝火、切石、锻造转化、锻造装饰
* **双提供商模式**： `SIMPLE` （Bukkit API）或 `COMPLEX` （直接NMS 注入）
* **多来源物品**：Sertraline自定义物品、原版物品、Bukkit标签、外部插件物品
* **合成后函数**：Kether 脚本、JavaScript 或控制台命令
* **按玩家生成的结果**：在预览时，根据玩家上下文重新生成Sertraline 自定义物品结果

***

### 文件位置

配置文件位于插件数据目录的 `recipes/` 文件夹中：

```
plugins/Sertraline/recipes/
  ├── example.yml
  └── ... (any filename, supports yaml/yml/toml/json)
```

在首次启动时（如果文件夹不存在）， `recipes/sample.yml` 会自动释放，供参考。

***

### 配置结构

```yaml
<recipe_id>:
  provider: SIMPLE              # RecipeProvider: SIMPLE (Bukkit API) or COMPLEX (NMS injection)
  type: CRAFTING                # RecipeType (see below)
  format: SHAPED                # RecipeFormat: SHAPED or SHAPELESS (CRAFTING only)

  # --- Crafting (SHAPED) ---
  pattern:                      # 1-3 rows, each 1-3 chars
    - "SSS"
    - "SWS"
    - "SSS"
  ingredients:                  # 字符对应材料
    S: "sertraline:star_of_recall"
    W: "sertraline:sword_of_skyline"

  # --- Crafting (SHAPELESS) ---
  ingredients:                  # List of ingredients
    - "sertraline:doe_card"
    - "minecraft:diamond"

  # --- Cooking (FURNACE / BLASTING / SMOKING / CAMPFIRE) ---
  ingredient: "sertraline:doe_card"  # 单一材料
  experience: 0.5               # XP granted (float)
  cookingTime: 200              # Ticks (200 = 10s)

  # --- STONECUTTING ---
  ingredient: "sertraline:ultimate_pickaxe"

  # --- SMITHING_TRANSFORM / SMITHING_TRIM ---
  template: "sertraline:star_of_recall"
  base: "sertraline:sword_of_skyline"
  addition: "sertraline:blade_of_mythic"

  # --- Result ---
  result:
    id: sertraline:sword_of_eternal_frozen   # 产物 ID
    count: 1                                  # 产物数量
    function:                                 # Post-craft functions (optional)
      - type: kether
        kether: tell "&aCraft successful!"

  # --- Input Options (optional) ---
  input-options:
    data-filter:
      - "sertraline_data"
      - "sertraline_revision"
    components:
      mode: REMOVE              # REMOVE or KEEP
      elements:
        - "minecraft:enchantments"

  # --- Optional ---
  group: "my_group"             # 配方书分组
  showNotification: true        # 显示进度样式通知（仅 CRAFTING）
```

***

### 配方类型

| 类型                 | 工作设施        | 说明                                                                  |
| -------------------- | -------------- | ---------------------------------------------------------------------------- |
| `CRAFTING`           | 工作台 | 有形状（网格图案）或无形状。需要 `format: SHAPED`或 `SHAPELESS` |
| `FURNACE`            | 熔炉        | 用经验值冶炼。 `cookingTime` 单位为 tick（20 tick = 1 秒）                           |
| `BLASTING`           | 高炉  | 更快地冶炼。 `cookingTime` 单位为 tick                                      |
| `SMOKING`            | 烟熏炉         | 食物熔炼。 `cookingTime` 单位为 tick                                        |
| `CAMPFIRE`           | 营火       | 慢速烹调 `cookingTime` 单位为 tick                                         |
| `STONECUTTING`       | 切石机    | 一种材料 → 一种产物，无经验值                                               |
| `SMITHING_TRANSFORM` | 锻造台 | 模板 + 基础物品 + 附加材料 → 产物                                          |
| `SMITHING_TRIM`      | 锻造台 | 模板 + 基准 + 附加项 → 修整后的物品                                    |

***

### 配方提供方式

#### 简单（Bukkit API）

默认模式。通过以下方式注册配方： `Bukkit.addRecipe()`. 适用于所有标准的Bukkit配方机制，并兼容与其他与Bukkit配方管理器交互的插件。

#### 复杂（NMS 注入）

将配方直接注入《我的世界》原生 `RecipeManager.recipes` 通过反射映射。使用时机：

* 作为成分的Sertraline自定义物品需要精确的NBT/组件匹配
* Bukkit API 匹配对于自定义物品数据来说不足够
* 需要绕过其他插件的Bukkit配方拦截

**注意**：COMPLEX 提供商示例在 sample.yml 中被注释掉。使用需自行承担风险。

***

### 材料

#### 格式

**1. 字符串格式（简单）**

```yaml
ingredient: "sertraline:doe_card"
```

**2. 对象格式（带数量）**

```yaml
ingredient:
  itemId: "sertraline:doe_card"
  amount: 2
```

**3. 标签格式**

```yaml
ingredient: "#minecraft:planks"     # Bukkit 标签
ingredient: "tag:planks"            # Shorthand (assumes minecraft namespace)
```

标签对象格式：

```yaml
ingredient:
  tagId: "#minecraft:planks"
  amount: 2
```

**4. 多选材料（或逻辑）**

```yaml
ingredient:
  options:
    - "sertraline:sword_of_skyline"
    - "sertraline:sword_of_counter"
    - "sertraline:sword_of_soulbound"
```

#### 物品来源

| 来源            | 标识符             | 示例                       |
| ----------------- | ---------------------- | ----------------------------- |
| 原版《我的世界》 | `minecraft:<material>` | `minecraft:diamond`           |
| Sertraline定制 | `sertraline:<id>`      | `sertraline:doe_card`         |
| ItemsAdder        | `itemsadder:<id>`      | `itemsadder:ruby`             |
| Oraxen            | `oraxen:<id>`          | `oraxen:sapphire`             |
| Nexo              | `nexo:<id>`            | `nexo:mythril_ingot`          |
| MythicMobs        | `mythicmobs:<id>`      | `mythicmobs:some_item`        |
| MMOItems          | `mmoitems:<type>/<id>` | `mmoitems:SWORD/flame_katana` |

省略源将默认为 `minecraft:`.

#### 解决逻辑

| 成分类型                         | 解析为                              | 匹配方式                                   |
| --------------------------------------- | ---------------------------------------- | ------------------------------------------ |
| 普通物品（`minecraft:stone`)        | `MaterialChoice`                         | 按材质匹配                        |
| Sertraline物品（`sertraline:doe_card`) | `ExactChoice`                            | 按完全相同的物品堆（包括NBT）匹配 |
| 外部插件物品                    | `ExactChoice`                            | 按确切的物品堆栈匹配                 |
| Tag (`#minecraft:planks`)               | `MaterialChoice(Tag)`                    | 与标签中的任何材质匹配                |
| 多选 (`options: [...]`)               | 合并的 `ExactChoice` 或 `MaterialChoice` | 匹配任何选项（“或”逻辑）              |

***

### 产物

#### 字段

| 字段      | 类型   | 必填 | 说明                                                                      |
| ---------- | ------ | -------- | -------------------------------------------------------------------------------- |
| `id`       | String | 是      | 输出物品 ID（例如， `sertraline:sword_of_eternal_frozen`, `minecraft:diamond`) |
| `count`    | Int    | 否       | 输出数量（默认值：1）                                                       |
| `function` | List   | 否       | 合成后函数列表（见下文）                                             |

#### 制作后功能

当玩家从工作台中取出结果时执行。有三种类型：

```yaml
result:
  id: sertraline:sword_of_eternal_frozen
  count: 1
  function:
    - type: kether
      kether: tell "&aCraft successful! Obtained Eternal Frozen Sword."
    - type: javascript
      script: player.sendMessage("§aCraft successful!")
    - type: command
      command: "say %player% crafted a legendary sword!"
```

**函数类型**

| 类型                | 字段                | 说明                                           |
| ------------------- | -------------------- | ----------------------------------------------------- |
| `kether`            | `kether: "<script>"` | 以玩家为发送者执行Kether脚本    |
| `javascript` / `js` | `script: "<js>"`     | 执行 JavaScript 代码段                         |
| `command`           | `command: "<cmd>"`   | 执行控制台命令（`%player%` 表示玩家名） |

短格式（仅脚本字符串，默认为Kether）：

```yaml
function:
  - tell "&aSuccess!"
```

***

### 输入选项

控制在合成过程中如何处理输入物品数据：

```yaml
input-options:
  data-filter:
    - "sertraline_data"
    - "sertraline_revision"
  components:
    mode: REMOVE              # REMOVE or KEEP
    elements:
      - "minecraft:enchantments"
```

#### 字段

| 字段                 | 类型               | 说明                                                             |
| --------------------- | ------------------ | ----------------------------------------------------------------------- |
| `data-filter`         | List\<String>      | 要筛选的 NBT 数据键（根据 `components.mode`) |
| `components.mode`     | `REMOVE` 或 `KEEP` | 是否移除或保留列出的组件                             |
| `components.elements` | List\<String>      | 数据组件命名空间（例如， `minecraft:enchantments`)              |

***

### 玩家感知结果预览

当玩家打开带有Sertraline配方的合成台时， `onPrepareCraft` 事件监听器使用玩家的上下文重建结果项——替换占位符、应用玩家特定变量并更新动态说明。

限制措施包括：

* Sertraline自定义物品，结果显示预览栏中显示了正确的玩家特定数据
* 带有 `sertraline:vars` 的物品会按玩家重建
* 随着玩家的变化，预览会实时更新

***

### 命令

| 命令              | 说明                           |
| -------------------- | ------------------------------------- |
| `/sertraline reload` | 重新加载所有配置，包括配方 |

每次重新加载时，配方都会从磁盘重新加载并重新注册。

***

### 完整示例

#### 定型合成 — 9 张卡牌 → 1 星

```yaml
star_from_cards:
  provider: SIMPLE
  format: SHAPED
  type: CRAFTING

  pattern:
    - "DDD"
    - "DDD"
    - "DDD"
  ingredients:
    D: "sertraline:doe_card"
  result:
    id: sertraline:star_of_recall
    count: 1
```

#### 无形状 — 混合两件物品

```yaml
mixed_sugar:
  provider: SIMPLE
  format: SHAPELESS
  type: CRAFTING

  ingredients:
    - "sertraline:bagged_zero_calorie_sugar"
    - "sertraline:bagged_crystal_sugar"
  result:
    id: sertraline:bagged_crystal_sugar
    count: 2
```

#### 熔炉 — 用经验值熔炼物品

```yaml
smelt_pills_from_cookie:
  provider: SIMPLE
  type: FURNACE

  ingredient: "sertraline:speed_potion_cookie"
  result:
    id: sertraline:sertraline_pills_14
    count: 1
  experience: 0.5
  cookingTime: 200
```

#### 高炉 — 更快的冶炼速度

```yaml
blast_pills_from_card:
  provider: SIMPLE
  type: BLASTING

  ingredient: "sertraline:doe_card"
  result:
    id: sertraline:sertraline_pills_42
    count: 1
  experience: 0.5
  cookingTime: 100
```

#### 锻造 — 升级物品

```yaml
mythic_sword_upgrade:
  provider: SIMPLE
  type: SMITHING_TRANSFORM

  template: "sertraline:star_of_recall"
  base: "sertraline:sword_of_skyline"
  addition: "sertraline:blade_of_mythic"
  result:
    id: sertraline:test_choten_sword
    count: 1
```

#### 石匠 — 1 个输入 → 多个输出

```yaml
cut_pickaxe_to_wands:
  provider: SIMPLE
  type: STONECUTTING

  ingredient: "sertraline:ultimate_pickaxe"
  result:
    id: sertraline:block_wand
    count: 2
```

#### 多选配料（或）+指令功能

```yaml
any_sword_to_star:
  provider: SIMPLE
  format: SHAPELESS
  type: CRAFTING

  ingredients:
    - options:
        - "sertraline:sword_of_skyline"
        - "sertraline:sword_of_counter"
        - "sertraline:sword_of_soulbound"
  result:
    id: sertraline:star_of_recall
    count: 1
    function:
      - type: command
        command: "say %player% recycled a sword!"
```

***

### 比较：配方系统与合成站系统

| 比较项                | 配方系统                                     | 合成站系统               |
| --------------------- | ------------------------------------------------- | ------------------------------------- |
| 配置文件夹(&C)     | `recipes/`                                        | `crafting-stations/`                  |
| **UI**                | 原版《我的世界》（工作台、熔炉等） | 自定义 GUI（6 行箱子）              |
| **制作时间**     | 即时                                           | 可配置的定时（秒）          |
| **条件**        | 无（如果成分相符，任何人都可以使用）        | Kether/分级/权限检查       |
| **生命周期脚本** | 仅限后期制作（通过 `result.function`)           | onStart/onCancel/onClaim 代理       |
| **持久化**       | 无                                              | DataContainer（退出/重启后仍存在） |
| **分页**        | 不适用（原味配方书）                         | 内置页面浏览功能                |
| 物品来源      | Sertraline、原版、外部插件             | 相同（通过 OmniItem）                   |
| **用途**          | 标准《我的世界》合成                       | 自定义计时进度系统      |

