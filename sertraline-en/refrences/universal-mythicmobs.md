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
