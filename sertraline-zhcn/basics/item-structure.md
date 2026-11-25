---
icon: cubes
---

# 物品结构

一个配置文件可以包含多个物品。每个顶层名称是一个物品 ID，其下的配置段分别描述物品的不同部分：

```yaml
red_token:
  xbuilder:
    material: PAPER
    name: "<!i><red>红色代币"

blue_token:
  xbuilder:
    material: PAPER
    name: "<!i><blue>蓝色代币"
```

保存到 `workspace` 后重载，再用 `/sertraline item give red_token` 领取红色代币。给其他玩家时写作 `/sertraline item give red_token Steve 1`，顺序是物品 ID、玩家、数量。

常见配置段：

* [xbuilder](../advanced/item-structures/section-xbuilder.md)：材质、名称、描述及常用物品属性。
* [sertraline](../advanced/item-structures/section-sertraline/README.md)：品级、物品数据和触发动作。
* [minecraft](../advanced/item-structures/section-minecraft-components.md)：受服务端版本支持的数据组件。
* [custom_data](../advanced/item-structures/section-custom-data.md)：自定义数据。

不必为每件物品填写所有段。你可以先从 `xbuilder` 开始，再按需要添加功能。配置中的 ID 应保持唯一；使用字母、数字和下划线有助于避免命令输入问题。
