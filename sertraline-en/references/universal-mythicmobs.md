---
icon: square-github
---

# MythicMobs Drops and Equipment

With MythicMobs and Sertraline installed, you can equip MythicMobs creatures with Sertraline items or add those items to their drops.

## Equip a husk and make it drop a sword

Create and reload `beginner_sword` using [My First Item](../basics/my-first-item.md). Save the following as `plugins/MythicMobs/Mobs/wiki_practice.yml`.

```yaml
WikiPracticeHusk:
  Type: HUSK
  Display: "&ePractice Husk"
  Health: 20
  Sertraline:
    equipments:
      mainhand: beginner_sword
    drops:
      - "beginner_sword 1 1"
```

Run `/mm reload` and then `/mm mobs spawn WikiPracticeHusk`. The husk holds the Beginner's Sword. Defeat it to receive one sword from the `drops` entry. [MythicMobs command reference](https://wiki.mythiccraft.io/mythicmobs/Commands-and-Permissions)

- Under `equipments`, set an item ID for each slot. Slots include `mainhand`, `offhand`, `helmet`, `chestplate`, `leggings`, and `boots`. Use `air` to clear a slot.
- Each entry in `drops` takes an item ID, amount, and drop chance, in that order. For example, `"blue_token 1-3 0.25"` gives a 25% chance to drop one to three blue tokens. Chances range from `0` to `1`. The amount and chance each default to `1`.
- Item IDs are case-sensitive. Add an item to both sections to display it as equipment and offer it as a reward. The equipment added by `equipments` has its normal equipment drop chance set to zero.

## Scripting API

When the MythicMobs integration loads, supported Sertraline script contexts can access `UMAPI`. For example, `UMAPI.getItemStack("mythic_item_id", player)` builds an item using a MythicMobs item ID. See the [Universal Mythic API](https://github.com/TabooLib/universal-mythic) for details.


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): the old Sertraline version threshold is historical, not a current requirement. When MythicMobs is loaded, the integration exposes its API as `UMAPI` to supported Sertraline script contexts. The current drop parser accepts an item ID, an optional fixed or ranged amount, and an optional chance.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

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

</details>
