---
icon: banjo
---

# Section: Sertraline

Overview Structure:

```yaml
sertraline:
  tier: C                    # Tier ID
  type: weapon               # Type ID
  lore-format: defaultloreformat  # Lore format ID
  vals:                      # Static values (determined at compile time)
    some_constant: 100
  vars:                      # Variable data (can be persisted to NBT)
    count: 0
    player_name: "%player_name%"
  dynamics:                  # Dynamic data (re-evaluated at runtime)
    random_damage:
      - "random 10 to 20"
  context:                   # Runtime context (not persisted)
    some_var: "runtime_value"
  keep-context: [key1, key2] # Context keys that should be persisted as variables
  actions:                   # Event actions
    onAttack:
      - async: false
        js: |
          event.damage = 100
  revision-id: 1             # Manual revision ID
  no-sertraline-id: false    # Prevent writing sertraline_id to NBT
```

vars placeholder usage (used with the tag handler; see below):

```yaml
sertraline:
  vars:
    pills: 14
    total: 14
xbuilder:
  lore: |
    <!i><gray>Left pills: <green>${var:pillsAsIn}$</green><gray> /${var:totalAsIn}$
```

Example:

```yaml
sword_of_counter:
  sertraline:
    tier: C
    vars:
      count: 0
    actions:
      onAttack:
        - js: |
            var tag = ItemStackUtil.getItemTag(bItem);
            var added = count + event.damage;
            tag.putDeep("sertraline_data.count", added);
            ItemStackUtil.setItemTagDirect(bItem, tag);
            PlayerUtil.sendMessage(player, mmUtil.deserialize("Damage: " + added))
            ItemStackUtil.rebuildLore(bItem, player);
  xbuilder:
    name: "<!i>&7Sword Of &6Counter"
    lore: |
      <!i><white>Total Damage: <yellow>${var:countAsIn}$
    material: IRON_SWORD
```

Please see the pages below
