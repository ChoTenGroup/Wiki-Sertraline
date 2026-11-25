---
icon: tent-double-peak
---

# 描述格式

你可以用同一套描述格式为多件物品显示统一的类型、品级和等级。把下面的配置保存为 `plugins/Sertraline/lore-formats/practice.yml`。

```yaml
practice_lore:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - key: "xbuilder:lore"
      content: "{value}"
    - key: "sertraline:type"
      content: "<!i><gray>类型 {type:name}"
    - key: "sertraline:tier"
      content: "<!i><gray>品级 {tier:format}"
    - key: "sertraline:level"
      content: "<!i><gray>等级 {level:name}"
```

将[物品品级示例](item-tier.md)中 `classified_sword` 的 `lore-format: loreformat` 改为 `lore-format: practice_lore`。重载后重新领取这把剑。原描述和三项分类信息会按 `elements` 中的顺序显示。

## 调整显示的内容

- 只填 `content` 可添加固定文本行。指定 `key` 后，用 `{value}` 显示该键的值。如果值是列表，列表的每项各生成一行。
- `key: "sertraline:vals.note"` 读取物品的 `vals.note`。`key: "*sertraline.vals.note"` 则从原物品配置直接读取，前缀 `*` 后的层级用点号分隔。读取 MMO 属性时使用转换后的大写键，例如 `mmo:ATTACK_DAMAGE`。
- 物品行也可用 `{val:note}` 引用固定值。要把同一描述应用于多件物品，在每件物品的 `sertraline` 段填写相同的 `lore-format` ID。

| `settings` 选项 | 默认值 | 作用 |
| --- | --- | --- |
| `overwrite` | `true` | 覆盖原描述。设为 `false` 时在原描述后追加 |
| `visual` | `false` | 开启后通过发包显示，需要 PacketEvents 和发包描述开关 |
| `auto-skip-blank` | `true` | 合并连续空行 |

## 仅在有数据时显示一行

把下面的条目加到 `practice_lore` 的 `elements` 列表末尾。然后在剑的 `sertraline` 下添加 `vals: { note: "活动奖励" }`。重载后，只有提供了 `note` 的物品才会出现提示。

```yaml
- content: "<!i><yellow>备注 {val:note}"
  lineMode: ALL
  lineRequire:
    - "sertraline:vals.note"
```

当 `lineRequire` 中有多个键时，`ANY` 表示至少有一项存在，`ALL` 表示全部存在，`NOT` 表示全部缺失，`NOT_ALL` 表示至少缺失一项。

## 格式化数值

| 写法 | 显示结果 |
| --- | --- |
| `{value}` | 保留值的原有文本 |
| `{auto}` | 移除末尾的 `.0`，例如 `15.0` 显示为 `15` |
| `{plus}{auto}` | 为正数添加加号，例如 `+15`。负数保留已有的负号 |
| `{minus}` | 在值为负数时单独显示负号 |
| `{round:2}` | 在当前版本显示两位小数，例如 `15.567` 显示为 `15.57`。冒号后的数字是格式宽度 |

用 `config.yml` 中的 `placeholders.plus` 和 `placeholders.minus` 可调整专用符号。有玩家上下文时，`content` 可使用 PlaceholderAPI 占位符，也可用 `{{ ... }}` 插入 Kether。随附的 `lore-formats/compare_example.yml` 和 `lore-formats/switchable_example.yml` 提供装备比较和切换描述的示例。
