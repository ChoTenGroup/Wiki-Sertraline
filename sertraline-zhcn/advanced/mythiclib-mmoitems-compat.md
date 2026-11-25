---
icon: infinity
---

# MythicLib/MMOItems Compat

### Use MMOItems Attributes

{% hint style="warning" %}
Due to the loading mechanism of MMO plugins,\
the attributes still do not work when only MythicLib is installed.\
You need install MMOItems (or) MMOCore to let attributes work.
{% endhint %}

Just Use Attribute Name under mmo section.

```
  mmo:
    attack-damage: 2020
    attack-speed: -5
    max-health: 5%
```

### Special Configuration Keys <a href="#configuration-keys" id="configuration-keys"></a>

```
mmo:
  id: "custom_identifier"
  source: "OTHER"
  slot: "MAIN_HAND"
  allowed:
    - "hand"
    - "helmet"
  
  # Regular attributes follow...
```

<table><thead><tr><th>Key</th><th>Type</th><th>Description</th><th>Default</th></tr></thead><tbody><tr><td><code>id</code></td><td>String</td><td>Unique identifier for this item's modifiers. Used to generate UUID for StatModifier registration.</td><td>Item's key (e.g., <code>example_sword</code>)</td></tr><tr><td><code>source</code></td><td>String</td><td><p>MythicLib <code>ModifierSource</code> enum value:</p><pre><code>MELEE_WEAPON,
RANGED_WEAPON,
OFFHAND_ITEM,
MAINHAND_ITEM,
HAND_ITEM,
ARMOR,
ACCESSORY,
ORNAMENT,
OTHER,
VOID;
</code></pre></td><td><code>OTHER</code> (from defSource parameter)</td></tr><tr><td><code>slot</code></td><td>String</td><td>MythicLib <code>EquipmentSlot</code> enum value: <code>MAIN_HAND</code>, <code>OFF_HAND</code>, <code>HELMET</code>, <code>CHESTPLATE</code>, <code>LEGGINGS</code>, <code>BOOTS</code></td><td>Auto-detected from item material or <code>MAIN_HAND</code></td></tr><tr><td><code>allowed</code></td><td>List</td><td>Which equipment slots allow this item to apply attributes. See <a href="https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/5e320724/Equipment%20Slot%20Filtering">Equipment Slot Filtering</a></td><td>Auto-detected from item material suffix</td></tr></tbody></table>

### Fix Attack speed

Have issues with rebound attack-speed?&#x20;

<pre class="language-yaml"><code class="lang-yaml">  mmo:
<strong>    fix-attack-speed: true
</strong></code></pre>
