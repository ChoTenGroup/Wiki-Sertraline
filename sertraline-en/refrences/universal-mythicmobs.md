---
icon: square-github
---

# Universal Mythicmobs

{% hint style="info" %}
This feature required `Mythicmobs` and `Sertraline 3.8.4+`
{% endhint %}

{% hint style="warning" %}
This feature is `Experiential` ! If you have any issues please open issue or contact me at discord/QQ.
{% endhint %}

Use _`UMAPI`_ for `Mythic.API`.&#x20;

From Sertraline 3.8.19.3 You can use this to hook mythicmobs, let mythicmobs drop Sertraline item or equip Sertraline item:

```
MMMobName:
  Type: HUSK
  Display: '&7XXX'
  Health: 20
  Damage: 4
  Sertraline:
    drops: # or Equippments
      - ITEMID 1 1
```

(Item id is Case sensitive)

### Examples

```
UMAPI.getItem("example_sword").generateItemStack(5)
UMAPI.getItemStack("example_sword", player) // directly
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

### Item Examples

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
              skillName = "LightningProjectile", // SKILL NAME
              trigger = player,
              origin = player.location,
              et = emptySet,
              lt = emptySet,
              power = 1.5
              )
```

### More Info

{% embed url="https://taboolib.maplex.top/docs/expanding-technology/universal-mythic/" %}

{% embed url="https://github.com/TabooLib/universal-mythic" %}
