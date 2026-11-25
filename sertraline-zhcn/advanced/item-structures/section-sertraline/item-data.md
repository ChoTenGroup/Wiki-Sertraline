---
icon: binary-circle-check
---

# 物品数据

### 数据类型

**Sertraline** 提供三种物品数据。

| 数据类型 | 存储位置                                                         | 说明                                               |
| --------- | ------------------------------------------------------------------- | --------------------------------------------------------- |
| Val       | <mark style="background-color:yellow;">**内存**</mark>            | 固定数据。                                 |
| Var       | <mark style="background-color:purple;">**物品 NBT**</mark>       | 存储在物品的 `sertraline_data` 节点中。 |
| Dynamic   | <mark style="background-color:blue;">**即时生成**</mark> | 读取时生成。                                |

### 它们有什么区别？

例如：

* Val 是固定值，适合用于描述生成器：可以在 `vals` 中定义，再在描述格式中引用。
* Var 是持久存储在物品中的变量，适合制作生成时具有随机数值的物品。
* Dynamic 是在需要数据时执行的 Kether 列表，以脚本结果作为值；使用时需考虑性能开销。

默认重建配置会保留 Var 数据。

### 示例

#### 随机伤害剑（Var）

生成这把剑时会随机产生伤害值；此示例使用 PlaceholderAPI。

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

#### 自动描述和属性剑

物品文件：

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

描述格式：

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

效果展示：

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
