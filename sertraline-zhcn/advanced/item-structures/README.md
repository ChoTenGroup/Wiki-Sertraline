# 物品配置段

物品可以由多个配置段组成。先阅读[物品结构](../../basics/item-structure.md)，再按需选择 XBuilder、Sertraline、Minecraft 数据组件或自定义数据。各段写在物品 ID 下方，与 `xbuilder` 对齐。

## 限制物品的用途

例如，活动券使用纸作为材质。你可以禁止玩家把它当作普通纸合成。把以下配置保存为 `plugins/Sertraline/workspace/event_only_token.yml`。

```yaml
event_only_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动专用券"
  common:
    disable-crafting: true
```

重载后，用 `/sertralineitem give event_only_token` 领取活动券。尝试用它代替合成书所需的任意一张纸，预览栏不会产出书。换回普通纸后，原版配方仍可正常使用。

支持的限制如下。每项默认关闭，按物品单独设置。合成站的材料消耗由该站点自己配置。

| `common` 下的字段 | 生效范围 |
| --- | --- |
| `disable-crafting` | `true` 禁止用作工作台和背包合成材料。用 `vanilla` 可仅针对原版配方 |
| `disable-smelting` | `true` 禁止用作熔炼输入 |
| `disable-smithing` | `true` 禁止用作锻造输入，需要现代构建 |
| `disable-village-trade` | `true` 阻止选择明确要求该物品的村民交易，需要现代构建 |
