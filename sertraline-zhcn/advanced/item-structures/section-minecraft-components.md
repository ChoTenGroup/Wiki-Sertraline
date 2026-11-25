---
icon: file-zipper
---

# Minecraft 数据组件

Minecraft 1.20.5 及以后的版本使用数据组件保存物品数据。组件能否使用取决于服务端版本；Legacy 构建不包含现代组件功能。旧版本的自定义数据用法见[自定义数据](section-custom-data.md)。

你可以在物品中使用 `minecraft` 段。例如，以下配置限制物品最多堆叠一个：

```yaml
single_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>单个代币"
  minecraft:
    max_stack_size: 1
```

保存并重载后，用 `/sertraline item give single_token` 领取并检查堆叠效果。组件字段及适用版本可查阅 [Minecraft Wiki 数据组件格式](https://minecraft.wiki/w/Data_component_format)。不同服务端版本对组件的支持不同；如果没有生效，请先检查控制台和该版本的组件格式。

## 添加模型组件

现代客户端会按照 `item_model` 选择显示模型。以下配置创建一颗看起来像铁锭的钻石，且每组最多堆叠一个。请在支持 `item_model` 的服务端和客户端上测试。

```yaml
model_item:
  xbuilder:
    material: DIAMOND
    name: "<!i><white>铁锭外观的钻石"
  minecraft:
    item_model: "minecraft:iron_ingot"
    max_stack_size: 1
```

## 移除装备组件

把字段写成 YAML 的 `null` 可移除相应组件。比如在支持 `equippable` 的服务端上，可以用以下配置阻止胸甲穿戴。

```yaml
unequippable_chestplate:
  xbuilder:
    material: DIAMOND_CHESTPLATE
    name: "<!i><aqua>展示用胸甲"
  minecraft:
    equippable: null
```

重载、领取后，手持物品使用 `/sertraline debug dumpComponent` 检查组件，或加上 `true` 查看完整组件列表。在提供 `dumpitem` 的 Paper 构建上，也可使用 `/paper dumpitem` 或 `/paper dumpitem all`。
