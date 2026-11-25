---
icon: star
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# 物品品级、类型与等级

你可以用 `tier` 标记稀有度、用 `type` 分类、用 `level` 标记档位。设置这些字段后，可以在物品浏览器中筛选，也可以在[描述格式](lore-format.md)中显示。攻击力和使用条件仍由相应的属性或脚本配置控制。

| 字段 | 默认文件 | 可直接使用的 ID |
| --- | --- | --- |
| `sertraline.tier` | `tiers/default.yml` | `E`、`D`、`C`、`B`、`A`、`S` |
| `sertraline.type` | `types/default.yml` | `WEAPON`、`SWORD`、`DAGGER`、`TOOL` |
| `sertraline.level` | `levels/default.yml` | `normal`、`advanced`、`super` |

## 创建并显示新品级

先在 `plugins/Sertraline/tiers/practice.yml` 中定义一个品级：

```yaml
legendary:
  name: "<gold>传说</gold>"
  description: "稀有的冒险装备"
  color: "<gold>"
```

再在 `plugins/Sertraline/workspace/classified_sword.yml` 中添加物品：

```yaml
classified_sword:
  sertraline:
    tier: legendary
    type: SWORD
    level: advanced
    lore-format: loreformat
  xbuilder:
    material: IRON_SWORD
    name: "<!i><gold>冒险者之剑"
    lore:
      - "<!i><gray>适合进阶冒险的装备。"
```

重载后执行 `/sertralineitem give classified_sword`。自带的 `lore-formats/loreGenerator.yml` 中，`loreformat` 会显示类型、品级和等级。自定义格式也可以用 `{type:name}`、`{tier:format}` 和 `{level:name}` 显示这三项。

定义自己的类型或等级时，沿用对应目录中默认文件的结构。类型可用 `parent` 指定父类型，例如默认的 `SWORD` 归属 `WEAPON`。每个目录中的 ID 应保持唯一。
