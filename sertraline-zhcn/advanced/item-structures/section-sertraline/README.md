---
icon: banjo
---

# Sertraline 配置段

`sertraline` 段可以为物品设置品级、数据和动作。下面的剑引用默认品级 `C`，并在攻击时将本次事件伤害设为 100：

```yaml
example_power_sword:
  sertraline:
    tier: C
    actions:
      onAttack:
        - async: false
          js: |
            event.damage = 100
  xbuilder:
    material: GOLDEN_SWORD
    name: "<!i><gold>力量之剑"
```

保存、重载后用 `/sertraline item give example_power_sword` 领取并攻击实体测试。修改或取消事件时需要设置 `async: false`；该例改的是触发时的事件伤害。完整默认示例可在 `workspace/default/default_weapon.yml` 中找到。

你还可以使用 `vals`、`vars` 保存物品所需的数据，并在名称或描述中引用。详见[物品数据](item-data.md)和[物品动作](item-actions.md)。
