---
icon: cursor
---

# 物品管理器系统

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
