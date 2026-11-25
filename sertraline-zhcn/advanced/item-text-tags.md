---
icon: sensor-cloud
---

# 物品文本标签

用 `${标签:参数}$` 把数据插入名称、描述或其他支持标签的位置。下例会显示比赛轮数、奖品和联系人。把配置保存到 `plugins/Sertraline/workspace/event_pass.yml`，重载后运行 `/sertralineitem give event_pass`。

```yaml
event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动通行证"
    lore:
      - "<!i><gray>轮数 ${val:roundsAsIn}$"
      - "<!i><gray>奖品 ${val:rewardsAsLiSplitBy / }$"
      - "<!i><gray>联系人 ${val:contact?:—}$"
  sertraline:
    vals:
      rounds: 2.7
      rewards: ["Gold", "Paper"]
```

描述会显示轮数 `3`、奖品 `Gold / Paper`。由于没有定义 `contact`，联系人一栏会显示 `—`。如果在 `vals` 中添加 `contact: "Steve"`，重载后重新领取即可看到新值。

## 选择和格式化数据

- `val` 读取 `sertraline.vals`，`var` 读取物品存储的变量，`dynamic` 用于按需计算的值。见[物品数据](item-structures/section-sertraline/item-data.md)。
- 在键后面加 `?:默认值` 可提供缺省文本，加 `!!` 可用空文本代替 `null`。如果对应数据段整个为空，标签可能原样保留。示例中的 `vals` 已包含其他键。
- 可在键后附加 `AsIn`（四舍五入为整数）、`AsSt`（文本）、`AsDo` / `AsFl`（浮点数）、`AsBy`（字节）、`AsLo`（长整数）、`AsBo`（布尔值）、`AsBd`（高精度小数）、`AsLi`（拼接列表）或 `AsMa`（拼接键值对）。

## 修改列表和键值对的分隔符

列表默认用逗号拼接。用 `AsLiSplitBy / ` 时，用斜杠两边各带一个空格的分隔符。用 `AsLiSplitByBLANK` 可以直接连写。单个空格也需要写在 `SplitBy` 后面。

键值对可在 `AsMaSplitBy` 后用反引号隔开两种分隔符，前面用于键与值，后面用于各条目。例如标签 ``${val:statsAsMaSplitBy=` / }$`` 会把 `{ hp: 10, speed: 2 }` 显示成 `hp=10 / speed=2`。
