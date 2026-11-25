---
icon: hammer-war
---

# Section: XBuilder

## XBuilder Structure Usage

### Item Example

```yaml
cool_sword:
  xbuilder:
    material: DIAMOND_SWORD
    name: "&2Cool Sword"
    lore: |
      You can tell from the name
      that this sword is cool
      because that's exactly what it says!
    enchants:
      UNBREAKING: 3
      SHARPNESS: 5
```

{% hint style="info" %}
The library used by XBuilder originally could not handle the Minimessage format, so we convert legacy color codes in the configuration.&#x20;
{% endhint %}

This code provides an example of an XBuilder item. How does it work?

* `material` - the item material
* `name` - gives the item a name. The name uses the CUSTOM\_NAME component (not ITEM\_NAME)
* `lore` - gives the item a description. A string or a list can be written here.
* `enchants` - the item's enchantments&#x20;

{% hint style="success" %}
XBuilder is very useful when you are creating general item formats. To find more usages, please see [HERE](../../refrences/xbuilder.md).&#x20;
{% endhint %}

{% hint style="info" %}
When xbuilder cannot meet your needs, you can look at the minecraft section, that is, the [data component section](https://app.gitbook.com/o/4JBmDCemEmfaVwC313DV/sites/site_q33Da/s/JV8wrbRjJhmUENOaPUJm/~/edit/~/changes/41/advanced/item-structures/section-minecraft-components), which provides everything from the vanilla data components, such as maximum stack size and equippable items.
{% endhint %}
