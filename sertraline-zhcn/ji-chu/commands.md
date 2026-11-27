# 常用命令

在游戏聊天框输入带 `/` 的命令，在服务器控制台执行时省略开头的 `/`。本页用 `<...>` 标记必填参数、用 `[...]` 标记可选参数；输入命令时不要输入括号。

| 命令 | 用途 |
| --- | --- |
| `/sertraline help` | 显示当前执行者可用的主命令树 |
| `/sertraline about` | 显示已安装的插件版本和运行平台 |
| `/sertraline reload` | 重载配置和物品定义；查看控制台中的具体文件报错 |
| `/sertralineitem browse` | 在游戏内浏览已加载的物品 |
| `/sertralineitem give <物品ID> [玩家] [数量] [静默]` | 给自己或在线玩家发放物品 |
| `/manager help` | 查看[物品管理器](../jin-jie/item-manager-system.md)命令 |
| `/sertraline station sertraline:<合成站ID>` | 在游戏内打开指定的[合成站](../jin-jie/crafting-stations.md) |
| `/sertralinegem list` | 列出主手物品上的宝石槽位；另见[宝石系统](../jin-jie/gems.md) |

例如，`/sertralineitem give beginner_sword Steve 2 true` 会给 Steve 两把[新手之剑](my-first-item.md)，并关闭收件提示。数量必须在 1 到 2304 之间。从控制台执行发放命令时，请提供在线玩家名；只有由玩家执行时，省略玩家参数才表示给自己。

{% hint style="info" %}
物品命令也有 `/items` 等别名；宝石命令的别名为 `/sgem`。管理员默认拥有权限。主命令、物品、管理器和宝石命令根节点分别使用 `sertraline.command.main`、`sertraline.command.item`、`sertraline.command.manager` 和 `sertraline.command.gem`。`/sertraline help` 只显示当前执行者有权限使用的子命令。
{% endhint %}
