---
icon: infinity
---

# MythicLib / MMOItems 兼容

物品的 `mmo` 段可以配置 MMO 属性。原手册提示，仅安装 MythicLib 时属性可能不会生效，还需要 MMOItems 或 MMOCore。实际可用属性取决于安装的插件及版本。

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
