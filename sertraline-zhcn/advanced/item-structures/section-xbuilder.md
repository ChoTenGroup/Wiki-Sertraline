---
icon: hammer-war
---

# XBuilder

`xbuilder` 用于设置物品的材质、名称、描述和常用属性。例如，在 `workspace` 的物品文件中添加：

```yaml
practice_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>练习剑"
    lore: |
      <!i><gray>适合练习的剑。
    unbreakable: true
    attributes:
      ATTACK_DAMAGE:
        amount: 10
        operation: ADD_NUMBER
        slot: HAND
```

重载后使用 `/sertraline item give practice_sword` 领取。`material` 指定原版材质；`name` 和 `lore` 分别设置名称与描述；`unbreakable` 设置无法损坏；`attributes` 示例为主手添加攻击伤害。数值和装备槽位可以按需要调整。更多字段见 [XBuilder 参考](../../refrences/xbuilder.md)，也可以参考 `workspace/default/default_weapon.yml`。

需要使用更多原版组件时，可继续阅读 [Minecraft 数据组件](section-minecraft-components.md)。
