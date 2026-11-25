---
icon: floppy-disks
---

# 安装与配置

## 选择版本

当前项目的 Standard 和 Spigot 构建以 Minecraft 1.21.4 API、Java 21 为目标；Legacy 构建以 Minecraft 1.12.2、Java 8 为目标，且不包含现代配方和数据组件功能。选择与你的服务端及 Java 版本相匹配的发行文件，并查阅发行说明确认兼容性。

PlaceholderAPI 可用于占位符；PacketEvents 2.10 或更高版本用于发包描述和组件等功能。它们按需安装，不是创建基础物品的必备条件。

{% hint style="warning" %}
物品和合成站配置可能包含脚本。脚本引擎可能访问 Java 类和服务器资源；导入第三方配置前请先检查内容并确认来源可信。
{% endhint %}

## 安装

1. 将适用的 Sertraline 插件文件放入服务器的 `plugins` 目录，按需安装其他插件。
2. 启动服务器，查看控制台确认插件成功加载。首次启动后会生成 `plugins/Sertraline` 目录。
3. 如需中文消息，在 `plugins/Sertraline/config.yml` 中设置 `lang: zh_CN`，然后执行 `/sertraline reload`。

常用目录如下；具体文件可能随版本而变化：

| 路径                                                 | 用途               |
| -------------------------------------------------- | ---------------- |
| `workspace/`                                       | 物品定义，支持子目录       |
| `templates/`                                       | 物品模板             |
| `lore-formats/`                                    | 物品描述格式           |
| `recipes/`                                         | 原版界面中的自定义配方      |
| `crafting-stations/`                               | 自定义合成站           |
| `tiers/`、`types/`、`levels/`                        | 品级、类型与等级         |
| `attributes/`                                      | 自定义属性定义          |
| `custom-events.yml`、`global-listeners/`、`scripts/` | 自定义事件、全局监听与可复用脚本 |
| `lang/`、`internal/`                                | 语言文件与内部资源        |
| `config.yml`、`manager.yml`、`experimental.yml`      | 主配置、物品管理器和实验开关   |

编辑后使用 `/sertraline reload` 应用运行期设置；数据库和统计设置在启动时读取，修改后需要重启服务器。在启动时关闭的发包注册也需要重启才能重新启用。接下来可以[创建第一个物品](my-first-item.md)。

## 可选依赖

| 插件                              | 使用场景                  |
| ------------------------------- | --------------------- |
| PlaceholderAPI                  | 在物品名、描述或其他支持的文本中使用占位符 |
| PacketEvents 2.10 或更高版本         | 发包描述、客户端材质和视觉组件       |
| MythicLib，配合 MMOItems 或 MMOCore | 给物品配置 MMO 属性          |
| ItemBridge 支持的物品插件              | 在配方或合成站里引用其他插件的物品     |

需要兼容新的物品或属性插件时，可在社区反馈需求。[ItemBridge 支持的物品来源](https://github.com/jhqwqmc/ItemBridge) · [服务与项目故事](../jie-shao/services.md)

## 调整主配置

在 `config.yml` 中修改已有字段。保留同一级已有的设置，避免添加第二个同名配置段。

| 选项                           | 修改后的行为                                       |
| ---------------------------- | -------------------------------------------- |
| `debug: true`                | 输出更多诊断信息，便于定位重载和脚本问题                         |
| `metrics.enabled: false`     | 停用插件的 FastStats 统计，重启后应用                     |
| `item-update.enabled: false` | 停用物品配置修改后的自动更新                               |
| `packet.packet-lore: false`  | 关闭发包描述。物品自身的普通描述仍可使用                         |
| `database.enable: false`     | 使用本地 SQLite。设为 `true` 并填写连接资料可改用 MySQL，修改后重启 |

例如，只想禁止每次重载都扫描在线玩家的旧物品时，可将 `item-update.triggers.on-reload` 设为 `false`。登录、切换手持物品和其他自动检查仍由各自的开关控制。
