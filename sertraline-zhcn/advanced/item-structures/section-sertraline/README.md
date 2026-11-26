---
icon: banjo
---

# Sertraline

用 `sertraline` 为物品加上分类、等级、可变数据、动作、描述格式和宝石。各字段可以按需组合。配置写在物品 ID 下，与 `xbuilder` 对齐。

```yaml
guide_ranked_blade:
  xbuilder:
    material: IRON_SWORD
  sertraline:
    tier: C
    type: SWORD
    level: normal
    lore-format: comparable_format
    compare-path: MAIN_HAND
    revision-id: 1

guide_plain_coin:
  xbuilder:
    material: GOLD_NUGGET
  sertraline:
    no-sertraline-id: true
```

| 字段 | 怎么填 |
| --- | --- |
| `tier` | 填 `tiers/` 中的品级 ID，如默认 `C` |
| `type` | 填 `types/` 中的类型 ID，如默认 `SWORD` |
| `level` | 填 `levels/` 中的等级 ID，如默认 `normal` |
| `lore-format` | 选择 `lore-formats/` 中的格式 ID。省略时不指定专用格式 |
| `compare-path` | 对比描述所读取的装备槽，如 `MAIN_HAND`、`OFF_HAND`、`HEAD`、`CHEST`、`LEGS` 或 `FEET` |
| `revision-id` | 手动修订号，填整数。未指定时由启用的自动跟踪功能管理 |
| `no-sertraline-id` | 设为 `true` 可生成普通用途的物品，省略构建时写入的 Sertraline ID。依赖这个 ID 的动作、识别、限制和自动更新将无法通过它关联到定义 |
| `vals`、`vars`、`dynamics`、`keep-context` | 设置固定值、保存的变量、动态计算和需要保留的生成上下文，见[物品数据](item-data.md) |
| `actions` | 按触发器填写动作列表，见[物品动作](item-actions.md) |
| `gem`、`gem-slots` | 定义宝石或物品插槽，见下方示例 |
| `gem-click` / `gem-click-socket` | 物品级的点击镶嵌开关，默认开启。用 `false` 只关闭点击镶嵌。也可在插槽和宝石各自的 `click` 中分别控制 |

品级、类型、等级是分类和显示用的标签。要给某个品级加伤害，另外设置属性或动作。详见[品级、类型与等级](../../item-tier.md)和[描述格式](../../lore-format.md)。

同一字段可选用默认映射提供的别名，例如 `action`、`loreFormat`、`comparePath` / `compare`、`gemSlots`、`gemClick` 和 `gemClickSocket`。同一物品中，每个字段选用一种写法。

### 指定更新版本

启用 `config.yml` 的 `item-update.enabled` 后，物品的修订号可以用来触发自动更新。例如先发放 `guide_ranked_blade`，修改配置后把 `revision-id: 1` 改为 `2`，再重载。在配置启用的检查时机，例如切换手持物品时，旧物品会重建。生成后已保存的变量默认会保留。需要快速查看新配置时，可以重载后直接领取一件。

### 创建宝石与插槽

`gem-slots` 写在承载宝石的装备上，`gem` 写在宝石上。可先体验插件自带的 `gem_example_sword` 和 `gem_example_ruby`。完整的插槽条件、兼容类型、修改方式、镶嵌交互和描述占位符见[宝石系统](../../gems.md)。

```yaml
guide_socketed_pickaxe:
  xbuilder:
    material: IRON_PICKAXE
    lore:
      - "<!i><gray>Power ${var:powerAsIn}$"
      - "<!i>${gem-slot:utility}$"
  sertraline:
    vars:
      power: 5
    gem-click: true
    gem-slots:
      utility:
        name: "<aqua>Utility Socket"
        gem-color: "blue"

guide_utility_gem:
  xbuilder:
    material: BLUE_DYE
  sertraline:
    gem:
      color: blue
      click: true
      modify-data:
        "sertraline:vars.power": "+1"
```

把镐放在主手，宝石放在副手，执行 `/sertralinegem socket utility`。镶嵌后描述中的数值从 5 变为 6；用 `/sertralinegem unsocket utility` 拆下后会恢复。
