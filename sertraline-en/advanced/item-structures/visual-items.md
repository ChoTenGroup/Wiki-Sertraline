---
icon: network-wired
---

# Visual Items

`visual` controls what the client sees. Giving, crafting and equipment behavior use the server's item. This section needs a server with data components, PacketEvents 2.10+, and `packet.register: true` and `packet.packet-component: true` in `config.yml`. If packet registration was disabled at startup, restart after enabling it.

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

Reload and give yourself each example. The first card is paper on the server and looks like a diamond. The second keeps paper as its material and uses an iron ingot model on supported clients. The third applies native text and glint components on the client.

| Field | How to use it |
| --- | --- |
| `material` | Client material, such as `DIAMOND` |
| `auto_name` | Client display name in MiniMessage or legacy color syntax |
| `auto_lore` | Client lore as a string, list, or YAML block |
| `auto_custom_model_data` | A numeric model value used by the client's resource pack |
| `auto_model` | A model ID such as `"minecraft:iron_ingot"` on server and client versions that support it |
| Other component names | Use `custom_name`, `lore`, `custom_model_data`, `item_model`, `enchantment_glint_override` and others in [native component format](section-minecraft-components.md). Set `null` to remove a component from the client copy |

Top-level names also accept hyphens, as in `auto-name`. Pick one form for each property when a shortcut and raw component target the same name, lore or model. `packet.packet-lore` separately controls lore format updates sent to the client.

With [PlaceholderAPI](../../references/placeholderapi.md) and its Player expansion installed, use an item tag to vary the name by viewer, such as `auto_name: "<yellow>${papi:player_name?:Guest}$"`.

<figure><img src="../../.gitbook/assets/image (5).png" alt="Paper displayed with a diamond appearance"><figcaption></figcaption></figure>
<figure><img src="../../.gitbook/assets/image (1).png" alt="Visual name containing the viewer’s name"><figcaption></figcaption></figure>
<figure><img src="../../.gitbook/assets/image (2).png" alt="Visual item name and lore"><figcaption></figcaption></figure>
