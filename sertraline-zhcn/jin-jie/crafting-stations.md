---
icon: plug-circle-plus
---

# 合成站（实验性）

合成站是插件提供的独立制作菜单。玩家点击配方后，插件从背包扣除材料，计时结束便自动发放产物。合成站属于实验性功能。

## 用纸制作蓝色代币

先按[物品结构](../ji-chu/item-structure.md)的示例创建 `blue_token`，再将下面的配置保存为 `plugins/Sertraline/crafting-stations/practice.yml`。

```yaml
practice_station:
  display:
    title: "<blue>练习合成站"
    layout:
      - "         "
      - "    I    "
      - "         "
    key: "I"
  recipes:
    press_paper:
      displayName: "蓝色代币"
      time: 3
      inputs:
        - item: "minecraft:paper"
          amount: 2
      outputs:
        - item: "sertraline:blue_token"
          amount: 1
```

1. 执行 `/sertraline reload`，在背包里放至少两张普通纸。
2. 执行 `/sertraline station sertraline:practice_station`。命令里的站点 ID 是配置最外层的 `practice_station`。
3. 点击菜单中央的蓝色代币。背包会扣除两张纸，约三秒后收到一个代币。

## 调整菜单

`display.layout` 每行写 9 个字符，行数决定箱子菜单的大小。用引号保留空格，`display.key` 对应的字符表示放置配方图标的槽位。配方默认用第一件产物作为图标。你也可以给配方加上 `display.item: "minecraft:book"`，让菜单显示一本书。

要添加背景，用 `#` 填充布局中的空白位置，并在 `display` 下、与 `layout` 同级的位置加入以下配置。

```yaml
elements:
  background:
    char: "#"
    material: GRAY_STAINED_GLASS_PANE
    name: "<gray>装饰"
```

配方多于一页时，可添加名为 `next_page` 和 `previous_page` 的元素，把各自的 `char` 放入布局。设置 `roll: true` 可在首尾页之间循环。元素名称和描述可使用 `{page}`、`{total_pages}`、`{next_page}` 和 `{previous_page}`。

## 调整配方

* 在 `recipes` 下为每条配方使用一个独立 ID。`time` 以秒为单位，也可写成 `"{{ random 1 to 10 }}"`。
* `inputs` 与 `outputs` 都是列表。每项用 `item` 写物品标识，例如 `minecraft:paper`、`sertraline:blue_token` 或 `itemsadder:ruby`。省略前缀时按原版材质读取。数量用 `amount`，省略时默认 1。自定义材料按完整物品数据匹配，建议先用生成结果固定的物品测试。
* 每个玩家同时只能进行一项制作。制作期间再次打开合成站，通常会取消当前制作并返还材料。给配方设置 `options.cancellable: false` 可禁止通过重新打开菜单来取消。

例如，你可以给 `press_paper` 添加权限限制。将下面这段加在该配方下，与 `inputs` 同级，再用没有该权限的账号点击测试。

```yaml
conditions:
  - name: "蓝色代币制作权限"
    type: permission
    condition: "example.craft.blue_token"
messages:
  ConditionNotMetPermission: "<red>你还没有{condition_name}。</red>"
```

`type: kether` 或 `boolean` 会检查 `condition` 脚本的结果。`type: Scaled` 则比较 `amount` 与 `required` 的数值，达到或超过要求时通过。`messages` 可按类型覆盖提示，例如 `ConditionNotMetScaled`。可用的提示变量有 `{condition_name}`、`{condition_amount}`、`{condition_required}` 和 `{condition_type}`。

## 制作完成时显示消息

把下面的配置加到 `press_paper`，与 `time` 同级。重载后再制作一次，领取代币时会收到消息。

```yaml
agents:
  onClaim: |-
    tell "蓝色代币制作完成！"
```

`onStart` 在开始计时时执行，`onClaim` 在发出产物后执行。条件也可在自己的 `agents` 里配置 `onSuccess` 和 `onFail`。你可以在站点或单条配方下添加制作脚本，两处都写时先执行配方级脚本。无前缀的脚本使用 Kether。使用 JavaScript 时，在脚本第一行加 `gjs:`（GraalJS）或 `js:`（JSR 223），例如 `gjs: player.sendMessage("制作完成！")`。

## 排查问题

* 找不到站点时，先核对顶层 ID，再用 `/sertraline debug getStations` 查看已加载的站点，并检查重载日志。
* 菜单没显示配方时，检查 `display.key` 是否出现在布局中，以及 `elements` 是否占用了同一槽位。
* 合成无法开始时，检查玩家权限、材料数量和材料数据。使用自定义材料时，先用配置里的同一物品 ID 重新领取材料。
* 离线期间完成计时，重新登录后会尝试补发产物。若登录时尚未完成，则会尝试退回材料。重载会取消在线玩家正在进行的合成并退还材料。

随附的 `crafting-stations/craftingStation.yml` 展示了更多布局和物品引用。`displayTime`、`plural` 和 `option.enable-tracker` 会被读取，但当前版本的菜单与制作流程未使用它们。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：进行中的合成保存在数据库 `CraftingSession` 记录中。当前流程会读取但不使用 `displayTime`、配方 `plural` 和 `option.enable-tracker`。重新打开站点会取消并退还材料，除非配方设置 `options.cancellable: false`。恢复旧会话时，只有仍能找到站点和配方才会运行 `onCancel`。站点级生命周期脚本中的 `{station_id}` 当前取菜单标题；元素脚本拿到的才是站点键。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: plug-circle-plus
---

# 合成站

合成站是插件提供的独立制作菜单。玩家点击配方后，插件从背包扣除材料，等待配方设定的时间，然后发放产物。合成站属于实验性功能。

先按[物品结构](../basics/item-structure.md)的例子创建 `blue_token`，再将下面的配置保存到 `plugins/Sertraline/crafting-stations/practice.yml`：

```yaml
practice_station:
  display:
    title: "<blue>练习合成站"
    layout:
      - "         "
      - "    I    "
      - "         "
    key: "I"
  recipes:
    press_paper:
      displayName: "蓝色代币"
      time: 3
      inputs:
        - item: "minecraft:paper"
          amount: 2
      outputs:
        - item: "sertraline:blue_token"
          amount: 1
```

1. 执行 `/sertraline reload`。
2. 把至少两张纸放进自己的背包，再执行 `/sertraline station sertraline:practice_station`。
3. 点击菜单中央的蓝色代币，约三秒后检查背包中的产物。

## 调整布局和配方

| 字段 | 用途 |
| --- | --- |
| `display.layout` | 每行写 9 个字符，并用引号保留空格。每个与 `display.key` 相同的字符代表一个配方槽 |
| `display.elements` | 定义背景和翻页按钮。可使用 `char` 对应布局字符，并填写 `material` 与 `name` |
| `recipes` | 每个配方用独立的 ID。配方中的 `displayName` 是菜单名称，`time` 的单位是秒 |
| `inputs`、`outputs` | 分别设置要消耗和发放的物品，通过 `item` 和 `amount` 指定来源、ID 和数量 |
| `conditions` | 限制谁能开始合成。可以配置权限、脚本或数值条件 |

自带的 `crafting-stations/craftingStation.yml` 展示了背景、翻页、条件及脚本的写法。打开菜单时如果提示找不到站点，保留命令里的 `sertraline:` 前缀，并检查后面的 ID 是否与 YAML 顶层键一致。

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
    enable-tracker: true  # 随附示例中的保留选项
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
→ materials returned to inventory
→ DB session cleared
```

***

### 命令

| 命令                           | 说明                             |
| --------------------------------- | --------------------------------------- |
| `/sertraline station sertraline:<stationId>` | 打开合成站界面               |
| `/sertraline debug getStations`   | 查看所有已加载的工作站配置         |
| `/sertraline reload`              | 重新加载所有配置（包括工作站） |

***

### 数据持久性

合成站系统使用数据库会话存储来持久化玩家的合成进度。

#### 存储数据

会话表保存玩家 UUID、合成站与配方 ID、开始时间、时长以及已扣除的材料。

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
      - "#########"
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
        char: ">"
        roll: true
        material: ARROW
        name: "<green>Next Page {next_page}"
      previous_page:
        char: "<"
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
          js: player.sendMessage("Conversion complete! Got Diamond x1")
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
        char: ">"
        roll: true
        material: ARROW
        name: "<green>Page {next_page}"
        agents:
          onLeft: |-
            gjs: player.sendMessage("Current page " + (page + 1))

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
        - item: "minecraft:netherite_sword"
          amount: "1"

      agents:
        onStart: |-
          gjs: player.sendMessage("Forging " + recipe + "...")
        onClaim: |-
          gjs:
            var msg = "§6★ Congratulations! You got the Legendary Sword! ★";
            player.sendMessage(msg);
            player.sendMessage("You forged " + recipe + "!");
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

</details>
