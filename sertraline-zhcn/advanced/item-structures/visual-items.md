---
icon: network-wired
---

# Section: Visual Items

{% hint style="info" %}
This feature required `packetevents`.
{% endhint %}

{% hint style="info" %}
This feature has been fixed on **3.6.2 or above**.
{% endhint %}

Virtual items are Client-Side.

It allows Client data only exists on the client side and does not involve server-side components.&#x20;

Through the client-side item component, you can easily update item descriptions in real-time, including properties such as item\_name and custom\_model\_data.&#x20;

{% hint style="success" %}
In addition, like CraftEngine, items will also function normally in creative mode.&#x20;
{% endhint %}

{% hint style="warning" %}
However, as the client component is modifing packets, if a container is filled with Sertraline items, handling these items may result in an empty container "A Empty Container Error" of less than 1 second. This is extremely evident on low-frequency machines.

&#x20;In the future, it may be possible to rewrite this feature at the netty / buffer level to seek near zero lag like CraftEngine, but this feature is extremely difficult for me and may never be implemented.
{% endhint %}

### Example: Client-Bound Material

Use `material` section for client bound material.

for example, this item in server-side is `PAPER`, but in client-side it becomes a `DIAMOND`!

<pre class="language-yml"><code class="lang-yml">test_visual:
  xbuilder:
    name: "real Paper in server"
<strong>    material: PAPER
</strong>    lore: |
      &#x3C;!i>&#x3C;#66ccff>a paper, looks like a DIAMOND.
<strong>  visual:
</strong>    material: DIAMOND
</code></pre>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

### Example: Per-player name item

<pre><code>test_visual:
  xbuilder:
    name: "REAL NAME IN SERVER!"
    material: PAPER
    lore: |
      &#x3C;!i>&#x3C;#66ccff>a Per-player name item.
  visual:
<strong>    custom_name: '{"text":"VISUAL NAME IN CLIENT - HELLO %player_name%!"}'
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Alias

Have issue with about raw component? Don't worry, we provide 4 alias for your visual item:

<pre class="language-yaml"><code class="lang-yaml">test_visual:
  xbuilder:
    name: "REAL NAME IN SERVER!"
    material: PAPER
    lore: |
      &#x3C;!i>&#x3C;#66ccff>a Per-player name item.
  visual:
<strong>    auto_name: "&#x3C;yellow>%player_name% &#x3C;white>Love &#x3C;#ff66ff>ChoTenChan&#x3C;white>!"
</strong><strong>    auto_lore: |
</strong>      &#x3C;yellow>Hello,
      &#x3C;#66ffff>ChoTenGroup!
<strong>    auto_custom_model_data: 114514
</strong><strong>    auto_model: "minecraft:iron_ingot"
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
