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
