---
icon: cursor
---

# 物品管理器

用物品管理器可以查看已加载的物品定义。要新增或调整长期使用的物品，请编辑工作区里的配置。

## 查看已加载的物品

先按[新手教程](../basics/my-first-item.md)创建并重载 `beginner_sword`。执行以下命令，列表里应该能看到新手之剑，详情会显示它所在的范围及已读取的配置键。

```text
/sertraline manager use public
/sertraline manager list
/sertraline manager info beginner_sword
```

你也可以在游戏中用 `/sertralineitem browse` 查看物品。

## 选择存储范围

| 命令参数 | 用途 |
| --- | --- |
| `public persistent` | 共享物品。通过 `workspace/` 下的配置创建，重载会重新读取 |
| `public temporary` | 共享的临时定义。同次运行期间可跨重载使用，服务器重启后丢失 |
| `private persistent` | 隔离到指定的 UUID，保存在数据库 |
| `private temporary` | 隔离到指定的 UUID，保存在内存，重载或重启后丢失 |

`/sertraline manager use public` 和 `/sertraline manager use private` 分别切换到公共和私有持久范围。3.13.2.2 的命令解析无法识别 `use public temporary` 这样的双参数写法。私有范围默认使用当前玩家的 UUID，也可通过 `/sertraline manager switch <UUID>` 指定。控制台如未切换 UUID，可读 `manager.yml` 中的 `private.auto-uuid`。

`manager.yml` 的 `defaults.scope` 和 `defaults.sub` 设置初始范围。默认禁止命令删除公共持久物品。若希望长期删除该物品，请从原文件移除对应顶层条目后重载。同一文件可能还定义着其他物品，启用 `delete-file` 之前请确认文件内容。

命令名也可用 `/manager` 或 `/mgr`。发放参数见[常用命令](../basics/commands.md)，开发用管理接口见 [SertralineAPI](../refrences/api/SertralineAPI.md#物品管理器)。
