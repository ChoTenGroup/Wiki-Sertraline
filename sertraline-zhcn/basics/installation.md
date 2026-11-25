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

| 路径 | 用途 |
| --- | --- |
| `workspace/` | 物品定义，支持子目录 |
| `templates/` | 物品模板 |
| `lore-formats/` | 物品描述格式 |
| `recipes/` | 原版界面中的自定义配方 |
| `crafting-stations/` | 自定义合成站 |
| `tiers/`、`types/` | 品级与类型配置 |
| `config.yml` | 主配置，包括数据库设置 |

编辑后使用 `/sertraline reload` 应用运行期设置；数据库和统计设置在启动时读取，修改后需要重启服务器。接下来可以[创建第一个物品](my-first-item.md)。
