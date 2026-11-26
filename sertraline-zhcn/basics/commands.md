# 常用命令

在游戏聊天框输入带 `/` 的命令。在服务器控制台执行时去掉开头的 `/`。下表中的尖括号表示必填参数，方括号表示可选参数，输入时省略括号。

| 命令 | 用途 |
| --- | --- |
| `/sertraline help` | 查看可用的子命令 |
| `/sertraline about` | 查看插件版本 |
| `/sertraline reload` | 重新读取配置。重载后查看控制台有无具体文件的报错 |
| `/sertralineitem browse` | 在游戏内浏览已加载的物品 |
| `/sertraline manager help` | 查看[物品管理器](../advanced/item-manager-system.md)的命令 |
| `/sertralineitem give <物品ID> [玩家] [数量] [静默]` | 向在线玩家发放物品。省略玩家时发给自己，数量默认为 1 |
| `/sertraline station sertraline:<合成站ID>` | 在游戏内打开指定的[合成站](../advanced/crafting-stations.md) |
| `/sertralinegem list` | 查看主手物品的宝石槽位。镶嵌与拆卸方法见[宝石系统](../advanced/gems.md) |

例如，执行 `/sertralineitem give beginner_sword Steve 2 true` 会给在线玩家 Steve 两把[新手之剑](my-first-item.md)，并隐藏收到物品时的提示。控制台发放时请填写玩家名。发放数量支持 1–2304。

主命令、独立物品命令和宝石命令的权限分别为 `sertraline.command.main`、`sertraline.command.item` 和 `sertraline.command.gem`，默认向管理员开放。
