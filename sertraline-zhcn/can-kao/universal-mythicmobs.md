---
icon: square-github
---

# MythicMobs 掉落与装备

安装 MythicMobs 和 Sertraline 后，可以让 MythicMobs 的怪物装备或掉落 Sertraline 物品。

## 让尸壳装备并掉落新手之剑

先按[新手教程](../ji-chu/my-first-item.md)创建并重载 `beginner_sword`。将下面的配置保存为 `plugins/MythicMobs/Mobs/wiki_practice.yml`。

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

* `equipments` 按槽位填写物品 ID。可用槽位包括 `mainhand`、`offhand`、`helmet`、`chestplate`、`leggings`、`boots`。填 `air` 可清空一个槽位。
* `drops` 的每一行依次是物品 ID、数量、掉落概率。例如 `"blue_token 1-3 0.25"` 代表有 25% 的概率掉落 1 到 3 个蓝色代币。概率范围是 `0` 到 `1`，省略数量或概率时各自默认为 `1`。
* 物品 ID 区分大小写。希望怪物既佩戴物品又把它作为奖励时，请同时填写两段。通过 `equipments` 设置的装备本身不会作为常规装备掉落。

## 脚本 API

MythicMobs 挂钩成功后，支持的 Sertraline 脚本上下文可以使用 `UMAPI`。例如，`UMAPI.getItemStack("mythic_item_id", player)` 按 ID 获取 MythicMobs 的物品。这里传入的是 MythicMobs 物品 ID。接口详情见 [Universal Mythic](https://github.com/TabooLib/universal-mythic)。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：旧文中的 Sertraline 版本门槛属于历史信息，不是当前要求。MythicMobs 已加载时，挂钩会把其 API 作为 `UMAPI` 提供给支持的 Sertraline 脚本上下文。当前掉落解析支持物品 ID、可选的固定或区间数量，以及可选概率。

<details>
<summary>旧版中文原文（原样保留）</summary>

---
icon: square-github
---

# Universal Mythicmobs

{% hint style="info" %}
此功能需要 `MythicMobs` 和 `Sertraline 3.8.4+`。
{% endhint %}

{% hint style="warning" %}
此功能属于实验性功能；遇到问题可提交 issue，或通过 Discord / QQ 联系项目方。
{% endhint %}

使用 `UMAPI` 访问 Mythic API。

从 Sertraline 3.8.19.3 起，可以在 MythicMobs 配置中让怪物掉落或装备 Sertraline 物品：

```
MMMobName:
  Type: HUSK
  Display: '&7XXX'
  Health: 20
  Damage: 4
  Sertraline:
    drops: # 也可使用 Equippments
      - ITEMID 1 1
```

物品 ID 区分大小写。

### 方法示例

```
UMAPI.getItem("example_sword").generateItemStack(5)
UMAPI.getItemStack("example_sword", player) // 直接获取物品
var mob = UMAPI.getMob(entity)
if (mob != null) {
    player.sendMessage("MONSTER ID: " + mob.id)
    player.sendMessage("NAME: " + mob.displayName)
    player.sendMessage("LV: " + mob.level)
    player.sendMessage("STANCE: " + mob.stance)
    player.sendMessage("FACTION: " + mob.faction)
}
UMAPI.getSkillMechanic("message{m=\"Hello World\"}").execute(
        trigger = UMAPI.getDefaultSkillTrigger(),
        entity = player,
        target = entity
    )
```

### 物品示例

```
lightningbottle_blue:
  xbuilder:
    name: "<!i><#0000ff><bold>Lighting in a bottle"
    material: PAPER
  sertraline:
    actions:
      onRightClick:
        - async: false
          js: |
            bItem.amount = bItem.amount - 1
            var HashSet = Java.type('java.util.HashSet')
            var emptySet = new HashSet();
            UMAPI.castSkill(
              caster = player,
              skillName = "LightningProjectile", // 技能名称
              trigger = player,
              origin = player.location,
              et = emptySet,
              lt = emptySet,
              power = 1.5
              )
```

### 更多信息

{% embed url="https://taboolib.maplex.top/docs/expanding-technology/universal-mythic/" %}

{% embed url="https://github.com/TabooLib/universal-mythic" %}

</details>
