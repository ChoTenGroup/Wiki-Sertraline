---
icon: plug-circle-plus
---

# 合成站\[Expr]

合成站让玩家在图形界面中通过配方制作物品，你可以使用 Kether 和 JavaScript 定义条件。

此外，合成站的灵感源自 `MMOItems`，但我们的合成站更具可扩展性、更易于维护~~并且漏洞更少~~（例如， `MMOItems` 取消配方将刷新合成材料的属性……）

{% hint style="warning" %}
正在完善中
{% endhint %}

## 合成站系统

### 概述

合成站系统提供基于GUI的合成机制，具有以下功能：

* 自定义 GUI 布局（6 行箱子界面）
* 分页浏览配方
* 多条件检查（Kether脚本、数值比较、权限）
* 异步制作计时器
* 生命周期脚本（代理）— Kether / GraalJS / JSR 223 JavaScript
* 多种物品来源（原版、Sertraline自定义物品、ItemsAdder/Oraxen/Nexo/CraftEngine等）
* 合成进度持久性（玩家退出/服务器重启时可恢复）

***

### 文件位置

配置文件位于 `crafting-stations/` 在服务器目录中：

```
plugins/Sertraline/crafting-stations/
  ├── craftingStation.yml
  └── ... (any filename, supports yaml/yml/toml/json)
```

如果首次启动时该文件夹不存在，则会自动生成一个示例文件。

***

### 配置结构

#### 顶级结构

```yaml
<station_id>:
  option:                          # 合成站选项
    enable-tracker: true
  display:                         # GUI 布局配置
    title: "..."
    layout: [...]
    key: "..."
    elements: {...}
  recipes:                         # 配方列表
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

### 显示配置 — GUI 布局

#### 字段

| 字段      | 类型          | 说明                                                 |
| ---------- | ------------- | ----------------------------------------------------------- |
| `title`    | String        | GUI标题，支持MiniMessage格式                      |
| `layout`   | List\<String> | 行字符串数组，每行9个字符，N行（N行宝箱GUI） |
| `key`      | String        | 单字符标记配方物品显示位置      |
| `elements` | Map           | 背景/导航元素定义                   |

#### 布局示例

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

* `#` — 对应 `background` 中的元素 `elements`
* `I` — 由 `key` 字段指定，用于标记配方物品填充位置（自动分页）
* `<`, `>` — 映射到页面导航元素

#### 界面元素

| 字段      | 类型                 | 说明                                                              |
| ---------- | -------------------- | ------------------------------------------------------------------------ |
| `char`     | 字符串（单个字符） | 布局中使用的字符（用于背景型元素）                  |
| `slot`     | Int                  | 原始槽位索引（0-53），用于未通过布局字符定位的元素 |
| `material` | String               | 物品材质名称（Bukkit材质）                                     |
| `name`     | String               | 显示名称，支持 MiniMessage                                       |
| `lore`     | String               | 描述文本， `\n` 用于换行，支持 MiniMessage                    |
| `roll`     | Boolean              | 页面按钮是否环绕（从最后一页返回到第一页）          |
| `agents`   | Map                  | 点击脚本，排序依据： `onLeft` 等                                    |

**占位符**（可用于名称和描述）：

| 占位符       | 说明                   |
| ----------------- | ----------------------------- |
| `{page}`          | 当前页码（从 1 开始） |
| `{total_pages}`   | 总页数              |
| `{next_page}`     | 下一页页码              |
| `{previous_page}` | 上一页页码          |

***

### 配方

```yaml
recipes:
  <recipe_id>:
    displayName: "..."            # 配方显示名称
    plural: "..."                 # 复数形式（用于消息）
    displayTime: "1~10s"         # 界面显示的时间（留空使用实际 time）
    time: "{{ random 1 to 10 }}" # 合成时间，支持 Kether 表达式

    options:
      cancellable: false          # Whether it can be cancelled (default true)

    conditions:                   # 条件列表（可选）
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

    messages:                     # 条件失败消息覆盖（可选）
      ConditionNotMetScaled: "..."

    display:                      # 界面显示物品（可选，默认使用第一个产物）
      item: "..."                 # OmniItem
      parameters: {...}

    inputs:                       # 输入材料列表
      - item: "minecraft:paper"
        displayName: "Paper"
        plural: "Papers"

    outputs:                      # 产物列表
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

#### 制作时间

`time` 字段支持两种格式：

1. **Kether 表达式**（推荐）： `"{{ random 1 to 10 }}"` — 每次合成随机 1\~10 秒
2. **数字**： `"30"` — 固定 30 秒

系统先尝试计算 Kether 表达式，失败时再尝试用 `toDouble()` 转换数字。

#### 条件类型

| 类型                 | 说明                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `Scaled`             | 数字比较。 `amount`和 `required` 都通过 Kether 计算，然后按 `amount >= required` |
| `boolean` / `kether` | 将 `condition` 作为 Kether 表达式执行，结果应为 true 或 false                                                  |
| `permission`         | 检查玩家权限： `player.hasPermission(condition)`                                                      |

#### 条件消息覆盖

可在配方的 `messages` 段按类型覆盖失败消息：

```yaml
messages:
  ConditionNotMetScaled: "Need {condition_amount}/{condition_required}, insufficient"
  ConditionNotMetPermission: "Missing permission: {condition_condition}"
```

如果没有覆盖，则使用 `config.yml` 中的默认消息。

可用的占位符： `{condition_name}`, `{condition_amount}`, `{condition_required}`, `{condition_type}`

***

### 物品参考（OmniItem）

输入、输出和显示中的物品使用统一的 `OmniItem` 格式：

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

#### 物品来源

| 来源                                                | 说明             | 示例                       |
| ----------------------------------------------------- | ----------------------- | ----------------------------- |
| `mc` / `minecraft` / `vanilla`                        | 原版物品           | `item: "minecraft:diamond"`   |
| `sertraline` / `depazitems`                           | Sertraline定制物品 | `item: "sertraline:doe_card"` |
| Other (`itemsadder`, `oraxen`, `nexo`, `craftengine`) | 外部插件物品   | `item: "itemsadder:ruby"`     |

省略来源时默认为 `mc`。

***

### 代理（Agents）— 脚本系统

代理支持三种脚本语言，通过前缀自动检测：

| 前缀                | 引擎                                           | 示例                            |
| --------------------- | ------------------------------------------------ | ---------------------------------- |
| 无前缀             | **Kether**（默认，向后兼容）        | `player.sendMessage("hello")`      |
| `gjs:` / `graaljs:`   | **GraalJS** (GraalVM Polyglot, ES2023+)          | `gjs: player.sendMessage("hello")` |
| `js:` / `javascript:` | **标准 JSR 223**（Nashorn / GraalVM JSR 223） | `js: player.sendMessage("hello")`  |

#### 可用变量

| 变量                  | 类型                 | 作用范围                                |
| ------------------------- | -------------------- | ------------------------------------ |
| `player` / `bukkitPlayer` | `Player`             | 所有代理                           |
| `recipe`                  | String (displayName) | 配方级/合成站级代理  |
| `station_id`              | String               | 合成站级/元件级代理 |
| `page`                    | Int                  | 元素级代理                 |
| `total_pages`             | Int                  | 元素级代理                 |

#### JS 实用程序类

GraalJS 和 JSR 223 JS 环境还会额外注入一个 `defaultData` 全局对象，包括：

* `PlayerUtil`, `ItemStackUtil`, `EventUtil`, `ThreadUtil`
* `Bukkit`, `Math`, `System`, `Gson`
* `SertralineAPI`, `ExternalItemHelper`, `FluxonShell`
* `UUIDUtil`, `scheduler`, `bukkitPlugin`

#### 生命周期钩子

| 钩子               | 触发时机                                  | 适用范围                 |
| ------------------ | ---------------------------------------- | ---------------------------- |
| `onStart`          | 合成开始后（计时器开始前） | 配方级 → 合成站级 |
| `onClaim`          | 合成完成、领取产物时     | 配方级 → 合成站级 |
| `onCancel`         | 合成取消或中断           | 配方级 → 合成站级 |
| `onSuccess`        | 条件检查通过                   | 条件级              |
| `onFail`           | 条件检查失败                    | 条件级              |
| `onLeft` 任意键 | 已点击 GUI 元素                      | 元素级                |

**执行顺序**（正常合成流程）：

```
Click recipe → condition check → onSuccess/onFail → consume items
  → execute onStart (recipe-level → station-level)
  → wait for crafting time → produce outputs
  → execute onClaim (recipe-level → station-level)
```

**取消流程**：

```
→ onCancel agents execute (recipe-level → station-level)
→ materials returned to inventory
→ DB session cleared
```

***

### 命令

| 命令                           | 说明                             |
| --------------------------------- | --------------------------------------- |
| `/sertraline station <stationId>` | 打开合成站界面               |
| `/sertraline debug getStations`   | 查看所有已加载的工作站配置         |
| `/sertraline reload`              | 重新加载所有配置（包括工作站） |

***

### 数据持久性

合成站系统使用TabooLib DataContainer来持久化玩家的合成进度。

#### 存储数据

* `crafting.session` — 会话元数据： `stationId|recipeId|startTime|totalSeconds`
* `crafting.items.0`, `crafting.items.1`, ... — Base64 序列化的已消耗物品

#### 恢复流程

```
Player quits → cancelPlayerSession() cancels in-memory tasks, DB preserved
Server restarts → shutdownAll() cancels all tasks, DB preserved
Player joins → onJoin → restoreSession():
  ├─ Craft complete → give outputs + execute onClaim agents
  └─ Craft incomplete → return materials + execute onCancel agents
  → Clear DB session
```

#### 重新加载流程

```
/sertraline reload:
  → cancelAll(): sync cancel + return items + clear DB (blocking wait)
  → craftingStations.clear()
  → loadCraftingStationFiles()
```

***

### 示例

#### 基础合成站

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

#### GraalJS 合成站

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

### 调试

在 `config.yml` 中设置 `debug: true`，查看详细调试日志。

使用 `/sertraline debug getStations` 检查已加载的合成站配置。

#### 常见问题

| 现象                      | 排查方法                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------- |
| GUI 显示空气物品          | 检查 `elements` 中的 `material` 拼写                                      |
| 配方未显示          | 检查 `key` 字符是否与元素的 `char` 冲突，启动时查看警告  |
| 合成无法启动            | 检查所有 `conditions` 是否满足，以及输入材料是否匹配                  |
| JS 脚本未执行     | 验证前缀为 `gjs:`或 `js:`，确认已加载 GraalJS/Nashorn 引擎 |
| 重启后材料丢失 | 检查 `restoreSession` 日志及 DataContainer 存储状态     |
