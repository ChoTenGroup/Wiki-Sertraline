---
icon: desktop-arrow-down
---

# 自定义数据

你可以通过 `custom_data` 为物品附加自定义数据，例如：

```yaml
marked_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>标记代币"
  custom_data:
    owner: example
    number: 42
    details:
      category: event
```

在 1.20.4 及更早版本中，这些值写入自定义 NBT；在 1.20.5 及以后的版本中，写入 `custom_data` 数据组件。支持嵌套对象和列表。保存后重载，再用 `/sertraline item give marked_token` 获取物品。

## 嵌套对象、列表与对象列表

下面是一张带活动分类、标签和奖励记录的代币。普通列表用 `-` 写每个值。对象列表则在每个 `-` 后填写该条记录的字段。

```yaml
event_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动代币"
  custom_data:
    details:
      category: event
      rank: 1
    tags:
      - summer
      - reward
    rewards:
      - item: emerald
        amount: 2
      - item: gold_ingot
        amount: 1
```

重载后使用 `/sertralineitem give event_token` 获取。你可以借助物品数据查看工具核对这些字段是否保存。配置里的标签和奖励记录由读取这些数据的脚本或插件使用。

## 值的类型和查验方式

支持 YAML 数字、布尔值、带引号的文本、嵌套对象、普通列表和对象列表。NBT 用字节保存布尔值，因此数据查看器可能把 `true` 显示成 `1b`，把 `false` 显示成 `0b`。同一个列表尽量使用同一种值。请使用本服专用的键，避免占用 `sertraline_id` 和 `sertraline_data` 等已有数据键。

其他插件或脚本可读取这些值。如果需要让物品动作更新变量并显示到描述里，用 [Sertraline 物品数据](section-sertraline/item-data.md)。在现代服务端，手持物品执行 `/sertraline debug dumpComponent`，查找 `minecraft:custom_data` 组件。
