# 自定义事件与全局监听

如果希望 Sertraline 物品响应额外事件，请使用 `custom-events.yml`；如果希望不依赖特定 Sertraline 物品、对整台服务器生效，请在 `global-listeners/` 中创建监听配置。

## 选择事件类名

这两种方式都注册 Bukkit 事件类。Bukkit / Paper 自带事件通常可以写简短类名，例如 `PlayerJoinEvent`；也可以填写包含包名的 Java 全限定类名。

{% hint style="success" %}
监听其他插件的事件时，请在 `class` 或 `bind` 中填写事件的**全限定类名**，例如 `com.vendor.plugin.event.SomeEvent`（请替换成真实包名和类名）。简写查找只会搜索已知的 Bukkit / Paper 包，不会搜索任意插件的包。目标插件必须已安装，并且 Sertraline 在运行时能够加载该事件类；该类还必须继承 Bukkit 的 `org.bukkit.event.Event`。
{% endhint %}

可以从目标插件的 API、Javadocs 或源代码中确认准确类名。重载后若监听器没有注册，请检查服务端日志中的类加载或事件类错误，并确认插件版本和包名正确。

## 让物品响应事件

在 `plugins/Sertraline/custom-events.yml` 顶层添加一个唯一名称的触发器。下面的示例会在玩家切换世界时触发：

```yaml
onPracticeWorldChange:
  class: org.bukkit.event.player.PlayerChangedWorldEvent
  item-source:
    - main-hand
```

在物品动作中使用相同的触发器名称。例如，将以下内容保存为 `workspace/practice_compass.yml`：

```yaml
practice_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><aqua>练习指南针"
  sertraline:
    actions:
      onPracticeWorldChange:
        - async: false
          graaljs: |
            player.sendMessage("你刚离开了 " + event.getFrom().getName() + "。");
```

执行 `/sertraline reload` 重载，用 `/sertralineitem give practice_compass` 领取指南针，然后手持它切换世界。聊天栏应显示你离开的世界名称。

`class` 用于指定事件类，也兼容使用 `listen`。`priority` 和 `accept-cancelled` 与全局监听的含义相同；省略时优先级为 `NORMAL`，已取消事件会被忽略。触发器名称必须唯一，且不能与内置动作名称冲突。

`item-source` 指定从哪里查找触发动作的物品。默认同时检查主手和副手；也可以使用 `main-hand`、`off-hand`、`both-hands`、`armor`、`inventory` 或 `event`。第三方事件可能没有 Sertraline 能自动识别的玩家或物品字段，此时可以用事件 getter / 字段路径通过 `custom-source` 显式指定；配置 `custom-source` 后会覆盖 `item-source`：

```yaml
onPluginItemUse:
  class: com.vendor.plugin.event.PluginItemEvent
  custom-source:
    player: event.player
    bItem: event.item
```

请将示例类名和路径替换为目标事件 API 中的实际值。`custom-source` 必须能解析出玩家和 `ItemStack`（或掉落物），物品动作才会执行。更多示例（包括指定攻击者和物品）可参考插件自带的 `custom-events.yml`。

## 添加服务器级监听

将下面的配置保存到 `plugins/Sertraline/global-listeners/practice_welcome.yml`。这里使用 Bukkit 玩家进服事件的全限定类名：

```yaml
practice_welcome:
  bind: org.bukkit.event.player.PlayerJoinEvent
  async: false
  eval:
    graaljs: |
      player.sendMessage("欢迎来到服务器！");
```

执行 `/sertraline reload`，重新登录即可看到消息。全局监听器不需要物品。每个配置文件的顶层键都是一个监听器；建议使用 `bind` 指定事件类，同时兼容 `class` 和 `listen`。

| 字段 | 用途 |
| --- | --- |
| `bind` | 事件类名：可用 Bukkit / Paper 简写或全限定类名。 |
| `priority` | `LOWEST`、`LOW`、`NORMAL`、`HIGH`、`HIGHEST` 或 `MONITOR`；默认 `NORMAL`。`MONITOR` 只应用于观察事件，不要在此优先级修改事件。 |
| `accept-cancelled` | 默认 `false`，跳过已取消事件；设为 `true` 可接收已取消事件。 |
| `async` | 默认 `false`。设为 `true` 会在异步线程执行脚本；仅用于不访问 Bukkit 对象或事件的计算。 |
| `condition` | 可选的 Kether 条件；条件为 false 时跳过脚本。 |
| `throttle` | 可选的最小触发间隔，单位为毫秒。移动等高频事件可用它限流；按监听器与已解析的玩家分别计时，无法解析玩家时则按监听器计时。 |
| `eval` | 脚本引擎和脚本内容。支持 `graaljs`、`javascript` / `js`、`kether`、`fluxon` / `fx` 和 `jexl`。 |

脚本会收到 `event`、`cancellableEvent`、`player` 和 `listener` 变量，以及 Sertraline 的常用脚本数据。`event` 是事件对象；对于不可取消的事件，`cancellableEvent` 为 null。如果 Sertraline 无法从事件中找到玩家，`player` 也可能为 null。使用前请先检查这些变量。

{% hint style="warning" %}
读取或修改背包、世界、玩家或事件状态（包括取消事件）时，请保持 `async: false`。没有玩家的事件仍可通过 `event` 交由全局监听处理；但物品动作需要成功解析出玩家和触发物品。
{% endhint %}
