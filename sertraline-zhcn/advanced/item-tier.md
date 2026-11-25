# 物品品级

品级可以用来给物品分类。默认定义位于 `plugins/Sertraline/tiers/default.yml`，其中包含 E、D、C、B、A、S 等品级。你可以在物品的 `sertraline` 段引用已有品级：

```yaml
tiered_sword:
  sertraline:
    tier: C
  xbuilder:
    material: IRON_SWORD
    name: "<!i><gray>普通剑"
```

保存并执行 `/sertraline reload`，再用 `/sertraline item give tiered_sword` 获取。品级本身如何显示取决于物品及描述格式的配置；设置品级不等于自动增加攻击力。需要自定义品级时，可先参照 `tiers/default.yml` 和原项目的 `docs/TierTypeLevel-zh.md`。
