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
