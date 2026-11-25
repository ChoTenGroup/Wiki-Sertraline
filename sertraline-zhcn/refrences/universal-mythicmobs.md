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
