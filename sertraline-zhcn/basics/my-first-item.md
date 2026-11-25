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

## 2. 重载并领取

保存文件，在游戏内执行：

```text
/sertraline reload
/sertraline item give beginner_sword
```

第二条命令会给执行命令的玩家一把剑。也可以指定在线玩家和数量，例如 `/sertraline item give beginner_sword Steve 2`；独立命令 `/sertralineitem give beginner_sword Steve 2` 的参数顺序相同。控制台发放时需要指定在线玩家。

如果没有收到物品，请检查控制台重载时的报错、YAML 缩进、材质和物品 ID，确认命令执行者有权限。可在 `workspace/default/` 中查看随插件提供的更多示例。
