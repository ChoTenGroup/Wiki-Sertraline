---
icon: square-github
---

# MythicMobs 掉落与装备

安装 MythicMobs 和 Sertraline 后，可以让 MythicMobs 的怪物装备或掉落 Sertraline 物品。

## 让尸壳装备并掉落新手之剑

先按[新手教程](../basics/my-first-item.md)创建并重载 `beginner_sword`。将下面的配置保存为 `plugins/MythicMobs/Mobs/wiki_practice.yml`。

```yaml
WikiPracticeHusk:
  Type: HUSK
  Display: "&e练习尸壳"
  Health: 20
  Sertraline:
    equipments:
      mainhand: beginner_sword
    drops:
      - "beginner_sword 1 1"
```

执行 `/mm reload`，再用 `/mm mobs spawn WikiPracticeHusk` 生成怪物。它会手持新手之剑。击败后，`drops` 配置会掉落一把同 ID 的剑。 [MythicMobs 命令参考](https://wiki.mythiccraft.io/mythicmobs/Commands-and-Permissions)

- `equipments` 按槽位填写物品 ID。可用槽位包括 `mainhand`、`offhand`、`helmet`、`chestplate`、`leggings`、`boots`。填 `air` 可清空一个槽位。
- `drops` 的每一行依次是物品 ID、数量、掉落概率。例如 `"blue_token 1-3 0.25"` 代表有 25% 的概率掉落 1 到 3 个蓝色代币。概率范围是 `0` 到 `1`，省略数量或概率时各自默认为 `1`。
- 物品 ID 区分大小写。希望怪物既佩戴物品又把它作为奖励时，请同时填写两段。通过 `equipments` 设置的装备本身不会作为常规装备掉落。

## 脚本 API

MythicMobs 挂钩成功后，支持的 Sertraline 脚本上下文可以使用 `UMAPI`。例如，`UMAPI.getItemStack("mythic_item_id", player)` 按 ID 获取 MythicMobs 的物品。这里传入的是 MythicMobs 物品 ID。接口详情见 [Universal Mythic](https://github.com/TabooLib/universal-mythic)。
