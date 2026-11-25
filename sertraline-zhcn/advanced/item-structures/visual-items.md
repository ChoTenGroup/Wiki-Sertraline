---
icon: network-wired
---

# 视觉物品

视觉物品改变客户端看到的显示，不改变服务器上的真实物品。`visual` 需要支持数据组件的服务端、PacketEvents，以及 `config.yml` 中的 `packet.register: true` 和 `packet.packet-component: true`。大量物品需要处理时，客户端容器显示可能短暂延迟。

## 显示不同材质

```yaml
test_visual:
  xbuilder:
    name: "<!i><white>服务器上的纸"
    material: PAPER
    lore: |
      <!i><gray>客户端看到的是钻石。
  visual:
    material: DIAMOND
```

服务器仍将它视为纸，客户端则显示钻石。保存并重载后，可以领取物品检查效果。

## 名称和描述

`visual` 也提供 `auto_name`、`auto_lore`、`auto_custom_model_data` 和 `auto_model` 等便捷字段。使用 PlaceholderAPI 时，可以在名称中引用玩家占位符：

```yaml
test_visual_name:
  xbuilder:
    material: PAPER
    name: "服务器名称"
  visual:
    auto_name: "<yellow>%player_name%<white> 的物品"
    auto_lore: |
      <gray>每位玩家看到自己的名字。
    auto_custom_model_data: 114514
    auto_model: "minecraft:iron_ingot"
```

模型效果取决于客户端资源包与服务端版本。也可以用 `custom_name` 指定原始 JSON 文本组件。

## 使用原始组件

如果需要直接填写 JSON 组件，可用 `visual.custom_name`。下面的纸在服务器上用固定的名称，客户端上则在名称中显示当前玩家名。使用 `%player_name%` 前，需要安装 PlaceholderAPI 和提供此占位符的 Player 扩展。

```yaml
visual_player_name:
  xbuilder:
    material: PAPER
    name: "服务器上的名称"
  visual:
    custom_name: '{"text":"你好，%player_name%！"}'
```

<figure><img src="../../.gitbook/assets/image (5).png" alt="客户端材质示例"><figcaption>用客户端材质显示钻石外观。</figcaption></figure>
<figure><img src="../../.gitbook/assets/image (1).png" alt="按玩家变化的视觉名称"><figcaption>根据玩家显示不同的名称。</figcaption></figure>
<figure><img src="../../.gitbook/assets/image (2).png" alt="视觉名称和描述示例"><figcaption>结合名称、描述与模型字段。</figcaption></figure>
