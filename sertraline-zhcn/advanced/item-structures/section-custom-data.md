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
