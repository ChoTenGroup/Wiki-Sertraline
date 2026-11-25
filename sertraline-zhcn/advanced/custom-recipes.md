---
icon: wrench
---

# 自定义配方

在 `plugins/Sertraline/recipes/` 里创建配方，玩家就能用工作台、熔炉等原版设施制作物品。Legacy 构建不提供现代配方功能。要制作带独立菜单、材料数量和等待时间的流程，请使用[合成站](crafting-stations.md)。

## 试用自带配方

打开 `recipes/sample.yml`，找到 `star_from_vanilla`。先确认 `workspace/default/default_misc.yml` 里的 `star_of_recall` 已加载。以下示例省略了可选的 `provider` 字段。

```yaml
star_from_vanilla:
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
    id: "sertraline:star_of_recall"
    count: 1
```

在工作台中央放一块金锭，周围八格各放一张纸，即可制作回忆之星。`pattern` 的每行对应网格的一行，`ingredients` 把字母对应到材料。空格表示空槽。需要修改这条配方时，直接编辑默认文件里的条目。

## 添加无序配方

先按[新手教程](../basics/my-first-item.md)创建 `beginner_sword`，再将下面的配置保存为 `recipes/practice.yml`：

```yaml
practice_sword_recipe:
  type: CRAFTING
  format: SHAPELESS
  ingredients:
    - "minecraft:iron_ingot"
    - "minecraft:iron_ingot"
    - "minecraft:stick"
  result:
    id: "sertraline:beginner_sword"
    count: 1
```

执行 `/sertraline reload`，把两块铁锭和一根木棍分别放入三个格子，顺序任意，即可得到新手之剑。每个配方槽消耗一件物品。多件相同材料应占多个格子，配置里也要写多次。

## 添加熔炉配方

把下面这条加到 `recipes/practice.yml` 的顶层，重载后把泥巴和燃料放进熔炉。200 游戏刻约为 10 秒，取出产物时这条配方提供经验。

```yaml
bake_practice_brick:
  type: FURNACE
  ingredient: "minecraft:mud"
  experience: 0.5
  cookingTime: 200
  result:
    id: "minecraft:brick"
    count: 1
```

## 选择材料和产物

- 原版物品用 `minecraft:paper`，Sertraline 物品用 `sertraline:beginner_sword`，外部物品用相应插件的前缀。省略前缀时优先匹配原版材质，再查找 Sertraline 物品。自定义物品的名称、附魔或其他数据变化可能影响材料匹配。
- 材料可写成 Minecraft 标签（例如 `"#minecraft:planks"`）或 `{ options: ["minecraft:oak_planks", "minecraft:birch_planks"] }`，表示一个槽可接受任一候选材料。
- 结果由 `result.id` 和 `result.count` 指定，数量省略时为 1。在工作台配方中，`result.function` 可在玩家取出产物时执行脚本或命令，写法见自带的 `craft_eternal_with_kether` 与 `any_sword_to_star`。

| `type` | 所用设施 | 主要字段 |
| --- | --- | --- |
| `CRAFTING` | 工作台 | 有序 `SHAPED` 用 `pattern` 和字符映射。无序 `SHAPELESS` 用材料列表 |
| `FURNACE`、`BLASTING`、`SMOKING`、`CAMPFIRE` | 熔炉、高炉、烟熏炉、营火 | `ingredient`、`experience`、`cookingTime` |
| `STONECUTTING` | 切石机 | `ingredient` |
| `SMITHING_TRANSFORM`、`SMITHING_TRIM` | 锻造台 | `template`、`base`、`addition` |

配方没有出现时，先检查重载日志是否指出了未找到的物品或注册失败的配方，再检查配方 ID 是否重复。

## 配方系统

### 概述

配方系统使用原版机制（工作台、熔炉、高炉、烟熏炉、篝火、切石机和锻造台）来注册自定义的Minecraft配方。它**不**使用合成台系统的自定义GUI。

主要特点

* **8种配方类型**：合成（有序/无序）、熔炉、烧炼、熏制、篝火、切石、锻造转化、锻造装饰
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

每条配方使用唯一的顶层 ID。按下面的类型表选择字段，并使用本页对应类型的完整示例。所有类型共用 `result`，其中填写产物 `id` 和 `count`。`group` 可用于配方书分组，`showNotification` 控制工作台配方通知。

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

### 配方注册方式

创建配方时可以省略 `provider`。在已核对的 Sertraline 3.13.2.2 中，省略该字段会使用默认值 `SIMPLE`。本页示例均采用默认方式。

| `provider` 值 | 如何注册 | 适用情况 |
| --- | --- | --- |
| `SIMPLE`（默认） | 通过 Bukkit API 注册 | 常规配方。可按物品元数据匹配 Sertraline 和外部插件的材料 |
| `COMPLEX`（实验性） | 直接注册到 Minecraft 的服务端配方管理器（NMS） | 材料按基础材质或标签匹配，适合能接受这种匹配方式的配方 |

显式写 `provider: SIMPLE` 与省略该字段效果相同。`recipes/sample.yml` 中的 `COMPLEX` 示例默认注释。用这种方式注册时，自定义材料的名称、附魔和组件不会参与匹配。例如，若自定义卡牌的底材是纸，普通纸也可作为材料。`COMPLEX` 会拒绝设置了 `exact: true` 的材料。

插件在玩家进服以及重载配方后使用 NMS 解锁已注册的配方，让它们能出现在配方书里。这一步适用于两种注册方式。

***

### 材料

#### 格式

**1. 字符串格式（简单）**

```yaml
ingredient: "sertraline:doe_card"
```

**2. 对象格式**

```yaml
ingredient:
  itemId: "sertraline:doe_card"
  amount: 1
```

**3. 标签格式**

```yaml
ingredient: "#minecraft:planks"     # Bukkit 标签
```

```yaml
ingredient: "tag:planks"            # Shorthand (assumes minecraft namespace)
```

标签对象格式：

```yaml
ingredient:
  tagId: "#minecraft:planks"
  amount: 1
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

省略前缀时，插件先匹配原版材质，再尝试 Sertraline 物品。

#### 材料匹配

| 材料 | 默认注册方式（`SIMPLE`） | 直接 NMS 注册（`COMPLEX`） |
| --- | --- | --- |
| 原版物品，如 `minecraft:stone` | 按材质匹配。对象写法可加 `exact: true` 以匹配没有额外元数据的原版堆栈 | 按材质匹配 |
| Sertraline 物品，如 `sertraline:doe_card`，或其他插件的物品 | 按构建出的物品匹配，包括名称、附魔与其他元数据。建议选用每次生成结果一致的物品 | 按该物品的基础材质匹配 |
| 标签，如 `#minecraft:planks` | 接受标签中的任一材质 | 接受标签中的任一材质 |
| 多选 `options` | 按各候选材料的匹配规则，满足其中一项即可 | 接受任一候选的基础材质或标签成员 |

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

当前加载器会读取下列字段，但它们尚未参与实际材料匹配。需要按物品元数据匹配时，请用自定义物品 ID 作为材料，并使用默认注册方式。

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
| `data-filter`         | List\<String>      | 预留的数据键列表 |
| `components.mode`     | `REMOVE` 或 `KEEP` | 是否移除或保留列出的组件                             |
| `components.elements` | List\<String>      | 数据组件命名空间（例如， `minecraft:enchantments`)              |

***

### 按玩家生成产物预览

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
  type: STONECUTTING

  ingredient: "sertraline:ultimate_pickaxe"
  result:
    id: sertraline:block_wand
    count: 2
```

#### 多选配料（或）+指令功能

```yaml
any_sword_to_star:
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
