---
icon: wrench
---

# 自定义配方

在 `plugins/Sertraline/recipes/` 里创建配方，就能让玩家用工作台、熔炉等原版设施制作物品。该功能适用于现代构建。需要独立菜单、背包扣料或自定义制作计时，可以创建[合成站](crafting-stations.md)。

## 试用自带配方

打开 `recipes/sample.yml`，找到 `star_from_vanilla`。它使用 `workspace/default/default_misc.yml` 中的 `star_of_recall` 作为产物。下面省略了使用默认值的 `provider`。

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

在工作台中央放一块金锭，周围八格各放一张纸，即可制作回忆之星。`pattern` 的每行对应网格的一行，`ingredients` 定义每个字母代表的材料，空格表示空槽。要改这条随附配方，可直接编辑该条目。

## 添加无序配方

按[新手教程](../ji-chu/my-first-item.md)创建 `beginner_sword`，再将下面的配置保存为 `plugins/Sertraline/recipes/practice.yml`。

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

执行 `/sertraline reload`。把两块铁锭和一根木棍分别放入三个格子，顺序任意，即可得到新手之剑。原版设施的每个配方槽消耗一件物品。`amount` 可省略，写成大于 1 的值会导致注册失败。无序配方要重复列出相同材料，有序配方则在图案里重复对应字母。

## 添加熔炉配方

把下面的条目加到 `recipes/practice.yml` 的顶层。重载后将泥巴和燃料放入熔炉，等待烧成红砖。正常速度下 200 游戏刻约为 10 秒，取出产物时由熔炉结算经验。

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

## 选择材料

| 写法                                           | 含义                                 |
| -------------------------------------------- | ---------------------------------- |
| `"minecraft:paper"`                          | 任意以纸为基础材质的物品                       |
| `"sertraline:beginner_sword"`                | 名称、附魔等数据与配方要求匹配的新手之剑               |
| `"itemsadder:ruby"`                          | 由相应物品插件提供的自定义物品                    |
| `"#minecraft:planks"` 或 `"tag:planks"`       | Minecraft 木板标签中的任一材质。带 `#` 的写法要加引号 |
| `{ itemId: "minecraft:paper", exact: true }` | 只接受和普通纸数据一致的物品                     |
| `{ tagId: "#minecraft:planks" }`             | 标签的对象写法                            |

省略物品前缀时，插件先匹配原版材质，再查找 Sertraline 物品。引用自定义材料时，建议使用每次生成结果一致的物品。随机属性、玩家专属名称和物品更新都可能改变匹配所需的数据。

例如，你可以把下面的 `ingredients` 用在一条无序配方里。第一格接受橡木或白桦木板，第二格使用普通纸。

```yaml
ingredients:
  - options:
      - "minecraft:oak_planks"
      - "minecraft:birch_planks"
  - itemId: "minecraft:paper"
    exact: true
```

## 选择设施和产物

| `type`                                    | 设施           | 配置                                              |
| ----------------------------------------- | ------------ | ----------------------------------------------- |
| `CRAFTING`                                | 工作台          | `SHAPED` 使用 `pattern` 和字符映射。 `SHAPELESS` 使用材料列表 |
| `FURNACE`、`BLASTING`、`SMOKING`、`CAMPFIRE` | 熔炉、高炉、烟熏炉、营火 | `ingredient`、`experience`、`cookingTime`         |
| `STONECUTTING`                            | 切石机          | `ingredient`                                    |
| `SMITHING_TRANSFORM`                      | 锻造台升级        | `template`、`base`、`addition`                    |
| `SMITHING_TRIM`                           | 锻造台盔甲纹饰      | `template`、`base`、`addition`。选择支持此类型的服务端与注册方式   |

例如，可用切石机把一个黑曜石切成两块黑石台阶。

```yaml
cut_practice_slabs:
  type: STONECUTTING
  ingredient: "minecraft:obsidian"
  result:
    id: "minecraft:blackstone_slab"
    count: 2
```

产物由 `result.id` 和 `result.count` 指定，数量省略时为 1。`SMITHING_TRIM` 也要求填写有效的 `result.id`，实际产物由模板和盔甲决定。工作台预览会用当前玩家的上下文生成 Sertraline 产物。工作台、烧炼和切石配方可用 `group` 设置分组。工作台配方可设置 `showNotification: false`，在支持该选项的平台上隐藏配方书通知。

## 合成后显示消息

把下面的 `function` 添加到一条工作台配方的 `result` 下，与 `id` 同级。重载后取出产物，聊天栏会显示消息。

```yaml
function:
  - type: kether
    kether: tell "合成成功！"
```

也可以使用 `type: command` 和 `command: "say %player% 完成了合成！"`。命令以控制台身份执行，`%player%` 代表玩家名。脚本类型 `javascript` 或 `js` 使用 `script` 字段，当前版本的配方 JavaScript 不注入 `player`。

## 注册方式和排查

* 省略 `provider` 时使用 `SIMPLE`，按上面的匹配规则处理材料。`COMPLEX` 属于实验性方式，按基础材质或标签匹配，例如以纸为底材的卡牌可被普通纸替代。它拒绝 `exact: true`。
* `input-options` 中的数据过滤和组件过滤是预留项。要按完整数据匹配，请使用 `SIMPLE` 和自定义物品 ID。
* 配方 ID 在目录内保持唯一。建议只用小写字母、数字和下划线。重载后检查控制台的注册结果。如果可手工制作但配方书尚未刷新，请重新登录。

更多完整示例见随附的 `recipes/sample.yml`。开发插件时可查阅[配方 API](../can-kao/recipe-api.md)。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：工作台配方的每个材料槽只消耗一件物品，因此材料 `amount` 必须为 `1`；需要更多材料时应重复占用配方槽。只有 `SIMPLE` 注册方式支持 `exact: true`。`COMPLEX` 会拒绝精确匹配材料；当前 `input-options` 会被解析，但尚未应用过滤。

<details>
<summary>旧版中文原文（原样保留）</summary>

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

</details>
