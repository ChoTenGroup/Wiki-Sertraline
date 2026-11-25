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

## 字段速查

| 字段 | 使用场景 |
| --- | --- |
| `tier`、`type`、`level` | 为物品设置品级、类型和等级 |
| `lore-format` | 选择 `lore-formats/` 中已经配置的描述格式 |
| `vals`、`vars`、`dynamics` | 分别设置固定值、可保存的变量和按需计算的值 |
| `keep-context` | 选择在生成物品时要保存为变量的上下文键 |
| `actions` | 设置使用物品时触发的动作 |
| `revision-id` | 手动指定物品配置修订号 |
| `no-sertraline-id` | 设为 `true` 时省略物品上的 Sertraline ID |
| `gem`、`gem-slots` | 定义宝石本身或该物品可用的宝石槽位 |

例如，你可以为药片设置初始数量，并在描述里显示剩余数量：

```yaml
counter_pills:
  sertraline:
    vars:
      pills: 14
      total: 14
  xbuilder:
    material: COOKED_BEEF
    name: "<!i><green>药片"
    lore: |
      <!i><gray>剩余药片 <green>${var:pillsAsIn}$</green><gray> / ${var:totalAsIn}$
```

领取时两个数值均为 14。要让剩余量随使用变化，可在 `onConsume` 动作里更新 `pills`，然后重建描述。配套写法见[物品数据](item-data.md)及[脚本工具](../../../refrences/script-util/README.md)。

## 累计伤害示例

插件自带的 `workspace/default/default_weapon.yml` 中，`sword_of_counter` 会把每次攻击的伤害加到 `count` 中，并更新物品描述。执行 `/sertralineitem give sword_of_counter` 后攻击实体，即可看到累计值。需要改计数初值时，编辑物品在 `sertraline.vars` 下的设置。

<figure><img src="../../../.gitbook/assets/image.png" alt="配置完成后的物品示例"><figcaption>可以在游戏中查看物品的名称和描述。</figcaption></figure>
