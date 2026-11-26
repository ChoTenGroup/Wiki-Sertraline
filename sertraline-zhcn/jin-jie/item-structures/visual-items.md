---
icon: network-wired
---

# 视觉物品

`visual` 改变客户端看到的样子。发放、合成和装备行为仍使用服务器上的物品。需要支持数据组件的服务端、PacketEvents 2.10+，以及 `config.yml` 中的 `packet.register: true` 和 `packet.packet-component: true`。如果启动时关闭了包注册，启用后要重启。

```yaml
guide_visual_card:
  xbuilder:
    material: PAPER
    name: "Server-side card"
  visual:
    material: DIAMOND
    auto_name: "<!i><aqua>Display card"
    auto_lore:
      - "<!i><gray>Server material: paper"
      - "<!i><gray>Client material: diamond"
    auto_custom_model_data: 10000

guide_visual_model:
  xbuilder:
    material: PAPER
  visual:
    auto_model: "minecraft:iron_ingot"

guide_visual_components:
  xbuilder:
    material: PAPER
  visual:
    custom_name: '{"text":"Client name","color":"gold"}'
    lore:
      - '{"text":"Client lore","color":"gray"}'
    enchantment_glint_override: true
```

重载后分别领取三个示例。第一张卡的真实材质是纸，客户端显示钻石。第二张卡保留纸材质，尝试使用铁锭的模型。第三张卡使用原版文本组件，并通过客户端组件开启发光。

| 字段 | 用法 |
| --- | --- |
| `material` | 只为客户端换用材质，例如 `DIAMOND` |
| `auto_name` | 用 MiniMessage 或传统颜色设置客户端名称 |
| `auto_lore` | 用字符串、列表或 YAML 多行文本设置客户端描述 |
| `auto_custom_model_data` | 填整数模型数据，效果由客户端资源包决定 |
| `auto_model` | 填模型 ID，例如 `"minecraft:iron_ingot"`，客户端和服务端需支持该组件 |
| 其他组件名 | 例如 `custom_name`、`lore`、`custom_model_data`、`item_model`、`enchantment_glint_override`，按[原版组件格式](section-minecraft-components.md)填写。用 `null` 可只从客户端副本移除组件 |

顶层字段名也可以用连字符，例如 `auto-name`。同一个名称、描述或模型属性建议只选便捷字段或原始组件中的一种。`packet.packet-lore` 单独控制描述格式的发包刷新。

安装 [PlaceholderAPI](../../can-kao/placeholderapi.md) 和 Player 扩展后，可让名称随观察者变化，例如 `auto_name: "<yellow>${papi:player_name?:访客}$"`。

<figure><img src="../../.gitbook/assets/image (5).png" alt="客户端材质示例"><figcaption>用客户端材质显示钻石外观。</figcaption></figure>
<figure><img src="../../.gitbook/assets/image (1).png" alt="按玩家变化的视觉名称"><figcaption>根据玩家显示不同的名称。</figcaption></figure>
<figure><img src="../../.gitbook/assets/image (2).png" alt="视觉名称和描述示例"><figcaption>结合名称、描述与模型字段。</figcaption></figure>
