---
icon: infinity
---

# MythicLib / MMOItems 兼容

在现代构建中，可以用 `mmo` 给 Sertraline 物品添加 MMO 属性。服务器需要同时安装 MythicLib 与 MMOItems 或 MMOCore。实际可用属性取决于这些插件的版本。

## 给主手武器增加攻击伤害

把下面的配置保存到 `plugins/Sertraline/workspace/practice_mmo_sword.yml`。

```yaml
practice_mmo_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>练习战剑"
  mmo:
    attack-damage: 10
    allowed: ["mainhand"]
```

重载后用 `/sertralineitem give practice_mmo_sword` 领取。切到剑所在的快捷栏槽位，再查看 MMO 属性或攻击测试目标。剑的 MMO 攻击伤害加成为 10，副手不应用这条加成。

如果安装了 PlaceholderAPI，手持普通钻石剑时执行 `/papi parse me %mythiclib_raw_stat_attack_damage%` 记下数值。切换到练习战剑，等约一秒再执行一次，数值应增加 10。

普通数字表示固定加成，例如 `attack-damage: 10`。支持百分比写法的加成可写为 `max-health: "5%"`。更多配置见随附的 `workspace/default/default_mmoitem.yml`。

<a id="configuration-keys"></a>

## 装备限制与进阶设置

| 字段 | 用途 |
| --- | --- |
| `allowed` | 实际允许触发 MMO 加成的位置。用小写 `mainhand`、`offhand`、`helmet`、`chestplate`、`leggings`、`boots`，或用 `hand` 允许双手。留空时按材质推断护甲位置，其他材质默认主手 |
| `slot` | 向 MythicLib 报告的装备槽。如需覆盖，填 `MAIN_HAND`、`OFF_HAND`、`HEAD`、`CHEST`、`LEGS` 或 `FEET`。省略时跟随实际装备槽 |
| `source` | 向 MythicLib 报告的来源。默认根据佩戴位置使用 `MAINHAND_ITEM`、`OFFHAND_ITEM` 或 `ARMOR`。可用值取决于 MythicLib 版本 |
| `id` | 这件物品的修改器标识。省略时使用物品 ID |

如果遇到攻击速度回弹，可在 `mmo` 下添加 `fix-attack-speed: true` 测试。要撤销这个设置，移除该字段后重载并重新领取物品。
