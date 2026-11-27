---
icon: binary-circle-check
---

# Item Data

## Pick a data type

| Configuration field | Text tag | Use |
| --- | --- | --- |
| `vals` / `val` | `${val:key}$` | A fixed value shared by all copies, such as text, a number, a list or a map |
| `vars` / `var` | `${var:key}$` | An initial value saved on each generated item. An action can change it later |
| `dynamics` / `dynamic` | `${dynamic:key}$` | Evaluate Kether when rendering or reading the value. Use a string, YAML block or list of script lines |
| `keep-context` / `keepContext` | `${var:key}$` | Save selected build context keys as item variables. Use `${context:key}$` to read context during that build |

```yaml
guide_roll_card:
  xbuilder:
    material: PAPER
    lore:
      - "<!i><gray>${val:guide_title}$"
      - "<!i><gray>Uses: ${var:usesAsIn}$"
      - "<!i><gray>Preview: ${dynamic:rollAsIn}$"
  sertraline:
    vals:
      guide_title: "Practice card"
      rewards: [Gold, Paper]
      stats: {points: 10, bonus: 2}
    vars:
      uses: 3
    dynamics:
      roll: "random 1 to 6"

guide_drop_medal:
  xbuilder:
    material: GOLD_NUGGET
    lore:
      - "<!i><gray>Killer: ${var:killer?:—}$"
  sertraline:
    keep-context: [killer]
```

Reload and run `/sertralineitem give guide_roll_card`. `uses` starts at 3. `roll` evaluates to a number from 1 to 6. The displayed text stays the same until the lore is rendered again. To keep a single roll, save the result as a variable.

When `guide_drop_medal` is generated as a [MythicMobs drop](../../../references/universal-mythicmobs.md) with a player as killer, it can retain `killer` in the item. A plain give command has no such context and uses the fallback `—`.

## Save a variable and refresh the lore

Variables are stored under `sertraline_data`. Assigning a temporary script variable alone leaves the saved item unchanged. Add the following `actions` under the card's existing `sertraline` section, aligned with `vars`.

```yaml
actions:
  onRightClick:
    - async: false
      js: |
        if (Number(uses) > 0) {
          var tag = ItemStackUtil.getItemTag(bItem);
          tag.putDeep("sertraline_data.uses", Number(uses) - 1);
          ItemStackUtil.setItemTagDirect(bItem, tag);
          ItemStackUtil.rebuildLore(bItem, player);
        }
```

Reload, give yourself a fresh copy and right-click with it in your hand. The number of uses drops from 3 to 2 and rebuilding the lore evaluates the roll again. See [Item Text Tags](../../item-text-tags.md) for formatting and fallbacks. The bundled `sword_of_counter` in `workspace/default/default_weapon.yml` shows how to accumulate damage instead.


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): `vals` are definition-level values, `vars` are item-instance data, and `dynamics` are evaluated when the value is requested. `keep-context` selects build-context values to retain on the generated item. The example updates saved item data before rebuilding lore, matching the current API flow.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

---
icon: binary-circle-check
---

# Item Data

### Requirements

**Sertraline** have 3 types of item data.

| Data Type | Stored Type                                                         | Description                                               |
| --------- | ------------------------------------------------------------------- | --------------------------------------------------------- |
| Val       | <mark style="background-color:yellow;">**Memory**</mark>            | This data in unmodifible.                                 |
| Var       | <mark style="background-color:purple;">**In Item NBT**</mark>       | This data will stored in item NBT node `sertraline_data`. |
| Dynamic   | <mark style="background-color:blue;">**Instantly generated**</mark> | Instant generation on get.                                |

### What's the difference between them?

For example:

* Val is a fixed value that is suitable for use with Lore generators. For example, define a val under vals and then specify this val in the lore generator.
* Var is a variable value stored persistently in item NBT, which is very suitable for making variant items, such as a sword that randomizes damage during generation.
* Dynamic is an executable Kether list that executes when data is needed and returns the script value as the result. _Not recommended for use, as this may result in some performance overhead._

_NOTE: Var data will be keep on rebuild by default. We will talk rebuild later._

### Good Examples

#### Random Damage Sword (VAR)

A sword. When generation it will randomly generate damage.

<pre class="language-yaml" data-expandable="true"><code class="lang-yaml">sword_of_lucky:
  sertraline:
    tier: S
<strong>    vars:
</strong><strong>      damage: "%sertraline_random:40to80%"
</strong>  xbuilder:
    name: "&#x3C;!i>&#x3C;b>&#x3C;u>&#x3C;#9999ff>Mr. BlueICE's&#x3C;/u> &#x3C;yellow>&#x3C;i>Lucky Sword"
    lore: |
      &#x3C;!i>&#x3C;#66ccff>a sword contains random damage in generation.
    material: DIAMOND_SWORD
    attributes:
      ATTACK_DAMAGE:
<strong>        amount: "${var:damage}$"
</strong>        operation: ADD_NUMBER
        slot: HAND
</code></pre>

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

#### Auto Lore & Attribute Sword

Item File:

<pre class="language-yaml" data-title="ITEM SECTION" data-expandable="true"><code class="lang-yaml">sword_of_autolore:
  mmo:
    attack-damage: 10
  sertraline:
    tier: D
    lore-format: example
    vals:
<strong>      attack-damage: 10
</strong><strong>      lore: |
</strong><strong>        A desc.
</strong><strong>      graylore: |
</strong><strong>        A gray desc.
</strong>  xbuilder:
    name: "&#x3C;!i>&#x3C;#FF6699>Auto Lore Sword"
    material: NETHERITE_SWORD
  # flags: [HIDE_ATTRIBUTES]
    attributes:
      ATTACK_DAMAGE:
<strong>        amount: "${val:attack-damage}$"
</strong>        operation: ADD_NUMBER
        slot: HAND
</code></pre>

Lore Format:

{% code title="LORE FORMAT SECTION" expandable="true" %}
```yaml
example:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - content: ""
    - key: "sertraline:vals.attack-damage"
      content: "<!i><#777777>| <#ff6936>Attack Damage <#aabbcc>{plus}{minus}{auto}"
    - content: ""
    - key: "sertraline:vals.lore"
      content: "<!i><#cccccc>| {value}"
    - key: "sertraline:vals.graylore"
      content: "<!i><#777777>| {value}"
```
{% endcode %}

ShowCase:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

</details>
