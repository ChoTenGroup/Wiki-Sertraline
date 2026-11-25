---
icon: binary-circle-check
---

# 物品数据

你可以在物品的 `sertraline` 段定义数据，并用[物品文本标签](../../item-text-tags.md)把值显示在名字或描述里。

| 字段 | 读取标签 | 适用场景 |
| --- | --- | --- |
| `vals` | `${val:键}$` | 同一种物品共用的固定信息，如活动名称 |
| `vars` | `${var:键}$` | 初次生成时写入这件物品的数据，如剩余次数 |
| `dynamics` | `${dynamic:键}$` | 读取时运行 Kether 脚本计算的值 |

## 显示固定数据

将下面的内容保存到 `plugins/Sertraline/workspace/practice_badge.yml`。重载后执行 `/sertralineitem give practice_badge`，悬停查看描述中的活动名。

```yaml
practice_badge:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动徽章"
    lore:
      - "<!i><gray>所属活动 ${val:season}$"
  sertraline:
    vals:
      season: "夏日活动"
```

## 保存剩余次数

在同一个文件的顶层添加下面的条目。重载后用 `/sertralineitem give practice_ticket` 领取。每张体验券的剩余次数从 3 开始，可以在之后的物品动作中分别修改。

```yaml
practice_ticket:
  xbuilder:
    material: PAPER
    name: "<!i><aqua>体验券"
    lore:
      - "<!i><gray>剩余次数 ${var:usesAsIn}$"
  sertraline:
    vars:
      uses: 3
```

`uses` 保存在物品的 `sertraline_data` 中，默认重建配置会保留它。可以添加动作修改这个值，再更新描述。用于累加攻击伤害的完整写法可在 `workspace/default/default_weapon.yml` 中找到，物品 ID 为 `sword_of_counter`。
