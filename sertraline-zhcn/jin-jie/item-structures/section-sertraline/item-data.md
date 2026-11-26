---
icon: binary-circle-check
---

# 物品数据

## 选择数据种类

| 配置字段 | 文本标签 | 用途 |
| --- | --- | --- |
| `vals` / `val` | `${val:键}$` | 同一物品定义共用的常量，支持文本、数字、列表和对象 |
| `vars` / `var` | `${var:键}$` | 每次生成时的初始值。存入实际物品，可以用动作在之后修改 |
| `dynamics` / `dynamic` | `${dynamic:键}$` | 渲染或读取时执行 Kether。可填一个字符串、YAML 多行文本或脚本行列表 |
| `keep-context` / `keepContext` | `${var:键}$` | 把生成上下文里指定的键保存到物品变量。通过 `${context:键}$` 也可读取当次生成过程的上下文 |

```yaml
guide_roll_card:
  xbuilder:
    material: PAPER
    lore:
      - "<!i><gray>${val:guide_title}$"
      - "<!i><gray>Uses: ${var:usesAsIn}$"
      - "<!i><gray>Preview: ${dynamic:rollAsIn}$"
  sertraline:
    vals:
      guide_title: "Practice card"
      rewards: [Gold, Paper]
      stats: {points: 10, bonus: 2}
    vars:
      uses: 3
    dynamics:
      roll: "random 1 to 6"

guide_drop_medal:
  xbuilder:
    material: GOLD_NUGGET
    lore:
      - "<!i><gray>Killer: ${var:killer?:—}$"
  sertraline:
    keep-context: [killer]
```

重载后用 `/sertralineitem give guide_roll_card` 领取。`uses` 从 3 开始。`roll` 是在求值时产生的 1 到 6 之间的数字，已生成的描述不会仅因玩家盯着它就自动刷新。如果想随机一次后保存结果，应把抽到的值写回变量。

`guide_drop_medal` 通过 [MythicMobs 掉落](../../../can-kao/universal-mythicmobs.md)生成且击杀者是玩家时，可以保存 `killer`。直接用发放命令领取时没有这个生成上下文，会显示缺省文本 `—`。

## 修改变量并刷新描述

变量保存在 `sertraline_data` 中。只修改脚本里的临时变量不会写回物品。下面给 `guide_roll_card` 增加右键动作。把 `actions` 放在原有的 `sertraline` 下，与 `vars` 对齐，避免写两个 `sertraline` 段。

```yaml
actions:
  onRightClick:
    - async: false
      js: |
        if (Number(uses) > 0) {
          var tag = ItemStackUtil.getItemTag(bItem);
          tag.putDeep("sertraline_data.uses", Number(uses) - 1);
          ItemStackUtil.setItemTagDirect(bItem, tag);
          ItemStackUtil.rebuildLore(bItem, player);
        }
```

重载、重新领取，然后手持右键。次数会从 3 降到 2，刷新描述时也会重新计算动态值。多行格式、转换和缺省文本见[物品文本标签](../../item-text-tags.md)，累计伤害的默认例子见 `workspace/default/default_weapon.yml` 中的 `sword_of_counter`。
