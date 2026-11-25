---
icon: file-zipper
---

# Section: Minecraft / Components

## Minecraft/Data Component Structure Usage

{% hint style="warning" %}
The data component system is only available in **1.20.5+**.

Some newer data components may not have been added on your server version, so they will have no effect.

**For 1.20.4- NBT, as well as a convenient way to use the Custom\_Data data component, please see** [Custom Data](section-custom-data.md).
{% endhint %}

## What Are Data Components

**Data Components**, or simply **Components**, are structured data used to define and store various data properties.For more information about data components, including how to use them, please refer to the Minecraft Wiki.

Below are links to the English data component wikis: [https://minecraft.wiki/w/Data\_component\_format](https://minecraft.wiki/w/Data_component_format)

{% hint style="info" %}
Available components depend on the server version and the plugin build. Check the console if a component cannot be applied.
{% endhint %}

## Usage

### Adding Data Components

Use the minecraft handler to add data components. For example, add a maximum stack size limit and an item model.

This will make the item use the texture of an iron ingot and allow it to stack only one.

```yaml
model_item:
  xbuilder:
    material: DIAMOND
    name: "An item disguised as an iron ingot that can only stack one but is actually a diamond"
  minecraft:
    item_model: "minecraft:iron_ingot"
    max_stack_size: 1
```

### Removing Data Components

Use null to remove a data component.

This will make this armor unable to be worn:

```yaml
Unequippable:
  xbuilder:
    material: DIAMOND_CHESTPLATE
    name: "Unequippable Armor"
  minecraft:
    equippable: null
```

Tip You can use the following commands to view the components of the item in your hand.

With all or true, it displays all components that come with the item.

`/paper dumpitem`

`/paper dumpitem all`

`/sertraline debug dumpComponent` (Some versions may return empty; if it returns empty, you can try the command below)

`/sertraline debug dumpComponent true`
