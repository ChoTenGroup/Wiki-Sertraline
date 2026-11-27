---
icon: gem
---

# 创建第一个物品

下面创建一把名为“新手之剑”的铁剑。请先完成[安装](installation.md)，并使用有管理权限的账号操作。

## 1. 创建配置

在 `plugins/Sertraline/workspace/` 下新建 `my_first_item.yml`，写入：

```yaml
beginner_sword:
  xbuilder:
    material: IRON_SWORD
    name: "<!i><green>新手之剑"
    lore: |
      <!i><gray>一把送给新冒险者的剑。
      <!i><yellow>祝你旅途愉快！
```

`beginner_sword` 是物品 ID，发放时要使用它；文件名只是用来组织配置，不是物品 ID。`material` 是原版材质，`name` 是显示名称，`lore` 是物品描述。`<!i>` 用于取消默认斜体，`<green>` 等是文本颜色标签。YAML 使用空格缩进，同级字段应对齐。

{% hint style="success" %}
命令、配方和引用中使用的是顶层物品 ID，不是文件名。ID 应保持唯一，并避免使用保留的 `__` 前缀。工作区文件可以放在多层子目录中，默认扫描深度最多为 10 层。
{% endhint %}

## 2. 重载并领取

保存文件，在游戏内执行：

```
/sertraline reload
/sertralineitem give beginner_sword
```

第二条命令会给执行命令的玩家一把剑。也可以指定在线玩家和数量，例如 `/sertralineitem give beginner_sword Steve 2`。控制台发放时需要指定在线玩家。

如果没有收到物品，请检查控制台重载时的报错、YAML 缩进、材质和物品 ID，确认命令执行者有权限。可在 `workspace/default/` 中查看随插件提供的更多示例。

{% hint style="info" %}
若物品没有加载，请查看重载日志中的文件名和物品 ID。默认文件筛选器接受 `.yaml`、`.yml`、`.toml`、`.tml`、`.json` 和 `.conf`；可在[安装与配置](installation.md)中了解 `file-load.item`。
{% endhint %}

## 自定义模型和颜色

你还可以在同一个物品文件中加入第二个物品。下面的熟牛肉使用 `custom-model-data` 提供资源包可识别的模型数据，并在名称和描述中混合使用 MiniMessage 与传统颜色代码。

```yaml
Embian:
  xbuilder:
    material: COOKED_BEEF
    name: "<!i><#555555>Embian</#555555> &6Pills"
    custom-model-data: 10000
    lore: |
      <!i><gray>”though i guess it was designed to be like that… face without mouth emoji”
      <!i>&7-Ame
```

重载后用 `/sertraline item give Embian` 领取。名称、描述和材质可直接查看。自定义外观需要对应的客户端资源包。

![游戏中的物品](https://github.com/user-attachments/assets/51175153-2a45-4820-b96a-d4f45cbb8c0d) ![物品描述示例](https://github.com/user-attachments/assets/819bb686-796a-449b-8f81-bae757d5ba7c)

工作区还支持 JSON 和 TOML。你可以按类型创建子目录，例如 `workspace/my_pack/food/embian.yml`。`config.yml` 的 `file-load.item` 控制哪些文件名会被读取。自定义模型、附魔和更多基础字段见 [XBuilder](../jin-jie/item-structures/section-xbuilder.md)。
