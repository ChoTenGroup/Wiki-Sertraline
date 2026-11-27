---
icon: binary-circle-check
---

# 物品数据

## 选择数据种类

| 配置字段 | 文本标签 | 用途 |
| --- | --- | --- |
| `vals` / `val` | `${val:键}$` | 同一物品定义共用的常量，支持文本、数字、列表和对象 |
| `vars` / `var` | `${var:键}$` | 每次生成时的初始值。存入实际物品，可以用动作在之后修改 |
| `dynamics` / `dynamic` | `${dynamic:键}$` | 渲染或读取时执行 Kether。可填一个字符串、YAML 多行文本或脚本行列表 |
| `keep-context` / `keepContext` | `${var:键}$` | 把生成上下文里指定的键保存到物品变量。通过 `${context:键}$` 也可读取当次生成过程的上下文 |

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

重载后用 `/sertralineitem give guide_roll_card` 领取。`uses` 从 3 开始。`roll` 是在求值时产生的 1 到 6 之间的数字，已生成的描述不会仅因玩家盯着它就自动刷新。如果想随机一次后保存结果，应把抽到的值写回变量。

`guide_drop_medal` 通过 [MythicMobs 掉落](../../../can-kao/universal-mythicmobs.md)生成且击杀者是玩家时，可以保存 `killer`。直接用发放命令领取时没有这个生成上下文，会显示缺省文本 `—`。

## 修改变量并刷新描述

变量保存在 `sertraline_data` 中。只修改脚本里的临时变量不会写回物品。下面给 `guide_roll_card` 增加右键动作。把 `actions` 放在原有的 `sertraline` 下，与 `vars` 对齐，避免写两个 `sertraline` 段。

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

重载、重新领取，然后手持右键。次数会从 3 降到 2，刷新描述时也会重新计算动态值。多行格式、转换和缺省文本见[物品文本标签](../../item-text-tags.md)，累计伤害的默认例子见 `workspace/default/default_weapon.yml` 中的 `sword_of_counter`。


## 源码核对

按插件修订版 `032ee71`（2026 年 11 月 26 日）核对：`vals` 是物品定义级的固定值，`vars` 是物品实例数据，`dynamics` 会在取值时执行。`keep-context` 用于指定生成物品时需要保留的上下文值。示例先更新物品存储的数据再重建描述，符合当前 API 调用方式。

<details>
<summary>旧版中文原文（原样保留）</summary>

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

</details>
