---
icon: cursor
---

# 物品管理器

用物品管理器可以在选中的范围内查看、创建、复制和删除物品定义。要发放物品，用 `/sertralineitem give`。要新增或调整公共持久物品，用 `workspace/` 下的配置。

## 查看已加载的物品

先按[新手教程](../ji-chu/my-first-item.md)创建并重载 `beginner_sword`，再依次执行以下命令。

```text
/sertraline manager use public
/sertraline manager list
/sertraline manager info beginner_sword
```

第一条命令选择公共持久范围。第二条列出物品 ID。第三条显示当前查询范围、物品 ID，以及已读取的数据键和配置键。也可以在游戏中用 `/sertralineitem browse` 打开物品浏览器。

## 命令速查

下表中的子命令都接在 `/sertraline manager` 后。也可以使用独立命令 `/manager` 或 `/mgr`，例如 `/mgr list`。尖括号里的内容要换成实际值，在服务器控制台输入时省略开头的 `/`。

| 子命令 | 用途 |
| --- | --- |
| `help` | 显示管理器自带的命令帮助 |
| `use <public 或 private> [persistent 或 temporary]` | 选择存储范围。省略第二个参数时选择 `persistent` |
| `switch <UUID>` | 为后续私有操作指定物品所属的玩家 UUID |
| `list` | 列出当前范围内的物品 ID |
| `info <物品ID>` | 查看一个物品的 ID、查询范围、数据键和配置键 |
| `delete <物品ID>` | 删除当前范围内的物品定义。公共持久范围需要额外启用配置 |
| `create <物品ID> <JSON>` | 在临时或私有范围中，从 JSON 对象创建物品定义 |
| `clone <源物品ID> <新物品ID>` | 把已有定义的数据复制到临时或私有范围中的新 ID |

独立管理器命令的权限为 `sertraline.command.manager`。主命令 `/sertraline` 使用 `sertraline.command.main`。默认向管理员开放。发放物品时，参数顺序见[常用命令](../ji-chu/commands.md)。

## 选择存储范围

| 范围 | 用途 |
| --- | --- |
| `public/persistent` | 可供所有玩家使用的公共物品。重载时从 `workspace/` 读取定义 |
| `public/temporary` | 存在内存中的公共定义，可以跨配置重载使用，服务器重启后清除 |
| `private/persistent` | 按玩家 UUID 隔离并保存在数据库中的定义 |
| `private/temporary` | 按玩家 UUID 隔离并保存在内存中的定义，关闭插件时清除 |

### 查看另一位玩家的私有物品

先选择私有范围，再填写目标玩家的 UUID。替换下方示例中的 UUID，执行 `list` 后，用它显示的完整物品 ID 查询详情。私有物品 ID 通常以 `__` 开头。

```text
/sertraline manager use private
/sertraline manager switch 550e8400-e29b-41d4-a716-446655440000
/sertraline manager list
```

游戏内进行私有操作时，默认使用命令执行者的 UUID。控制台可通过 `switch` 指定，也可以在 `manager.yml` 中设置 `private.auto-uuid`。再次执行 `use` 会清除之前通过 `switch` 指定的 UUID。玩家退出后会清除自己的范围和 UUID 选择。

发放私有物品时，插件会在接收玩家的 UUID 下查找完整物品 ID。例如，给在线玩家 Steve 发放其私有代币，可以执行 `/sertralineitem give __quest_token Steve`。

### 选择临时范围

要管理共享的临时物品，执行 `/sertraline manager use public temporary`。要管理自己的临时物品，执行 `/sertraline manager use private temporary`。`persistent` 可简写为 `pers` 或 `p`，`temporary` 可简写为 `temp` 或 `t`。

要为尚未手动选择范围的命令执行者设置初始范围，可以编辑 `plugins/Sertraline/manager.yml` 中的 `defaults` 并重载。例如，设置 `scope: public` 和 `sub: temporary`，初始范围就是公共临时。

## 创建和复制物品

### 创建一个临时纸质物品

逐条执行下面的命令。`create` 中物品 ID 后面的 JSON 对象定义物品，`clone` 把物品数据复制到一个新的 ID。

```text
/sertraline manager use public temporary
/sertraline manager create manager_demo_paper {"xbuilder": {"material": "PAPER"}}
/sertraline manager clone manager_demo_paper manager_demo_paper_copy
/sertraline manager info manager_demo_paper_copy
```

如果 JSON 字符串需要保留连续空格，可以用 `\u0020` 代表一个空格，例如 `"两个\u0020\u0020空格"`。

在游戏中运行 `/sertralineitem give manager_demo_paper_copy` 即可领取副本。这两个临时定义会在服务器重启时清除。

### 添加公共持久物品

在 `plugins/Sertraline/workspace/manager_example.yml` 中保存以下内容。

```yaml
manager_example_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>管理器示例代币"

manager_example_token_copy:
  xbuilder:
    material: PAPER
    name: "<!i><aqua>管理器示例代币副本"
```

执行 `/sertraline reload`，再依次执行 `/sertraline manager use public` 和 `/sertraline manager info manager_example_token_copy` 查看第二个物品。要复用配置，可以像示例那样复制一份，并为它填写新的顶层 ID。

### 创建一个私有持久物品

从控制台操作时，先将示例 UUID 换成物品所属玩家的 UUID。在游戏内为自己创建物品时，可以省略 `switch`。

```text
/sertraline manager use private persistent
/sertraline manager switch 550e8400-e29b-41d4-a716-446655440000
/sertraline manager create manager_demo_private {"xbuilder":{"material":"PAPER"}}
/sertraline manager info __manager_demo_private
```

插件会将物品定义保存在选中的 UUID 下。查询或复制时，使用 `list` 显示的完整物品 ID。

原版 3.13.2.2 的 `use`、`create` 和 `clone` 存在双参数解析问题。如果仍在运行该版本，可以在 `manager.yml` 设置初始临时范围，在 `workspace/` 添加公共持久物品，或用下面的 Kether 命令创建示例私有持久物品。替换示例 UUID 后整行输入。

```text
/sertraline api eval smanager use private persistent smanager switch 550e8400-e29b-41d4-a716-446655440000 smanager create manager_demo_private by {"xbuilder":{"material":"PAPER"}}
```

开发者可查阅[物品管理 API](../can-kao/sertralineapi.md#物品管理器)。

## 删除物品定义

执行 `/sertraline manager delete <物品ID>` 前，用 `info` 检查 ID 和当前范围。私有持久范围中的删除会移除该 UUID 的数据库记录，临时范围中的删除会移除对应的内存定义。

公共持久范围默认禁止命令删除，由 `manager.yml` 中的 `allow-delete-public-persistent` 控制。启用后，命令只会移除当前加载的定义，`delete-file` 不会使它删除 YAML。要永久移除一个公共物品，可以编辑原 YAML，删掉该物品的顶层条目，然后执行 `/sertraline reload`。同一文件中的其他物品可以保留。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：公共临时物品在配置重载时保留于内存缓存，插件关闭时清除；当前实现没有将它们序列化到重载缓冲区。私有持久物品存入数据库，私有临时物品只保存在内存中。配置类会读取 `delete-file`，但当前没有调用处；删除公共持久物品只会从已加载的注册表移除，不会删除 YAML 源文件。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: cursor
---

# 物品管理器系统

## 查看已加载的物品

先在 `workspace/` 创建并重载一个物品。执行 `/sertraline manager use public` 选择公共持久范围，然后执行 `/sertraline manager list` 查看已加载的物品。要查看[新手之剑](../basics/my-first-item.md)的详情，可执行 `/sertraline manager info beginner_sword`。在游戏中也可以用 `/sertralineitem browse` 打开浏览器。

## 物品管理器系统

Sertraline 的物品管理器为物品提供四种存储模式：**公开** / **私有** × **持久** / **临时**。管理器处理物品的创建、检索、删除以及跨重载的生命周期管理。

***

### 架构

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

#### 存储类型

| 类型                  | `SubManagerType` | 范围       | 持久性                                         |
| --------------------- | ---------------- | ----------- | --------------------------------------------------- |
| **公开持久** | `PERSISTENT`     | 所有玩家 | itemMap（内存），已清除并从工作区重新加载 |
| 公共临时      | `TEMPORARY`      | 所有玩家 | 在重新加载期间序列化到缓冲区，之后恢复  |
| 私有持久    | `PERSISTENT`     | 按 UUID    | 数据库                                            |
| 私有临时     | `TEMPORARY`      | 按 UUID    | 仅内存，关闭后丢失                         |

#### 枚举

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

### 配置（`manager.yml`）

```yaml
# 默认管理器类型和子类型
defaults:
  scope: "public"          # public / private
  sub: "persistent"        # persistent / temporary

# 允许删除公共持久物品
allow-delete-public-persistent: false

# 删除公共持久物品时同时删除文件
delete-file: false

# 私有管理器设置
private:
  # 没有玩家上下文时的自动 UUID
  auto-uuid: ""
```

***

### 公共物品

公共物品由所有玩家共享。它们是从以下位置加载的标准物品类型： `workspace/`.

#### 行为

| 操作 | PERSISTENT                         | TEMPORARY                                    |
| --------- | ---------------------------------- | -------------------------------------------- |
| 创建    | 不支持通过管理平台操作          | `createTemporary(id, item)`                  |
| 读取      | `getItem(id)` → itemMap            | `getItem(id)` → 临时 → itemMap 备用方案 |
| 删除    | 从 itemMap 中移除（受配置限制） | 从临时 + itemMap 中移除              |
| 重载    | 已清除，已从工作区重新加载   | 序列化 → 缓冲 → 恢复                |

#### 创建限制

通过管理器创建公共持久物品已被阻止（`UnsupportedOperationException`）。这些物品仅从 YAML 工作区文件加载。对于运行时注册，请使用 `SertralineAPI.registerItem(id, item)` 或公共临时路径。

***

### 私有物品

私有项的范围限定为 UUID（玩家或自定义命名空间）。它们支持数据库持久性。

#### 行为

| 操作 | PERSISTENT                       | TEMPORARY      |
| --------- | -------------------------------- | -------------- |
| 创建    | 缓存 + 数据库插入          | 仅限内存中 |
| 读取      | 缓存 → 数据库回退        | 仅内存      |
| 删除    | 缓存 + 数据库删除          | 仅限内存中 |
| 重载    | 缓存已清除，从数据库懒加载 | 已清除        |
| 关闭  | 已清除                          | 已清除        |

#### UUID 解析

如果未提供 UUID，则管理器将按以下顺序解析一个 UUID：

1. 显式 UUID 参数
2. 玩家 UUID 上下文
3. `manager.yml` `private.auto-uuid` 设置
4. 不可用时抛出 `IllegalStateException`

***

### API（通过 `SertralineAPI`）

```kotlin
val api = Sertraline.api()
```

#### 公开物品操作

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

#### 私有物品操作

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

#### 通用操作

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

#### 内部管理器访问

```kotlin
// Direct access to the underlying managers
val publicMgr = Sertraline.manager.public
val privateMgr = Sertraline.manager.privateManager
```

***

### 命令

#### `/sertraline manager` (`/mgr`)

管理公共/私有、永久/临时范围内的物品。

| 子命令                                      | 说明                              |
| ----------------------------------------------- | ---------------------------------------- |
| `help`                                          | 显示命令用法                       |
| `use <public\|private> [persistent\|temporary]` | 设置管理器范围                        |
| `switch <uuid>`                                 | 切换私有管理器 UUID              |
| `create <id> <json>`                            | 在当前范围内创建物品（JSON 正文） |
| `clone <templateId> <newId>`                    | 克隆现有物品                   |
| `delete <id>`                                   | 按 ID 删除物品                     |
| `list`                                          | 列出当前范围内的物品              |
| `info <id>`                                     | 显示物品详细信息                        |

**示例：**

```
# 使用公共临时范围
/manager use public temp

# 使用私有持久范围
/manager use private persistent

# Switch to a specific UUID for private operations
/manager switch 550e8400-e29b-41d4-a716-446655440000

# Create a public-temporary item with JSON body
/manager create my_item {"xbuilder":{"material":"stone","name":"&aMy Item"}}

# Clone an existing item
/manager clone sword_of_skyline my_cloned_sword

# 列出当前范围内的物品
/manager list

# 显示物品详情
/manager info my_item

# Delete an item
/manager delete my_item
```

#### `/sertraline item give`

将Sertraline物品交给玩家。

```
/sertraline item give [player] [amount] [silent]
```

**示例：**

```
# 给自己一把 sword_of_skyline
/sertraline item give sword_of_skyline

# Give a player 5 of the item silently
/sertraline item give sword_of_skyline 5 true
```

***

### Kether 集成

#### 物品中的Kether变量

物品变量（`vars` 部分）支持使用 `${kether:...}$` 语法：

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

#### 使用 Kether 执行条件操作

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

## 结构

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

重新加载期间：

* **公开-持久**：从工作空间的 YAML 文件中擦除并重新加载
* **公共-临时**：在清除之前序列化为字符串缓冲区，在重新加载后反序列化
* **私有-持久**：缓存已清除，下次访问时从数据库懒加载
* **私有-临时**：缓存已清除，如果不另行保存，则会丢失

</details>
