---
icon: infinity
---

# MythicLib / MMOItems 兼容

物品的 `mmo` 段可以配置 MMO 属性。请同时启用 MythicLib 与 MMOItems 或 MMOCore，让 MMO 属性能够生效。实际可用属性取决于安装的插件及版本。

```yaml
example_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "<!i><green>示例剑"
  mmo:
    attack-damage: 10
    attack-speed: -1
    max-health: 5%
```

<a id="configuration-keys"></a>

## 特殊字段

| 字段 | 用途 | 默认值 |
| --- | --- | --- |
| `id` | 属性修改器的标识 | 物品 ID |
| `source` | MythicLib 的修改器来源，如 `OTHER`、`MELEE_WEAPON` | `OTHER` |
| `slot` | 装备槽位，如 `MAIN_HAND`、`OFF_HAND`、`HELMET` | 根据物品材质检测，或使用主手 |
| `allowed` | 允许属性生效的装备槽位列表 | 根据物品材质检测 |

```yaml
mmo:
  id: "custom_identifier"
  source: "OTHER"
  slot: "MAIN_HAND"
  allowed:
    - "hand"
    - "helmet"
  fix-attack-speed: true
```

`fix-attack-speed` 可用于处理攻击速度回弹问题。请在测试服务器验证实际效果，更多示例见插件自带的 `workspace/default/default_mmoitem.yml`。

## 来源与装备槽可选值

- `source` 使用 MythicLib 定义的来源，例如 `MELEE_WEAPON`、`RANGED_WEAPON`、`MAINHAND_ITEM`、`OFFHAND_ITEM`、`HAND_ITEM`、`ARMOR`、`ACCESSORY`、`ORNAMENT`、`OTHER`、`VOID`。可用值取决于服务器上的 MythicLib 版本。
- `slot` 可使用 `MAIN_HAND`、`OFF_HAND`、`HELMET`、`CHESTPLATE`、`LEGGINGS` 或 `BOOTS`。
- 如果 `allowed` 没有设置或为空，插件按材质推断生效位置。例如钻石靴默认对应靴子槽。需要多个位置时，在 `allowed` 中逐行填写。
