---
icon: gem
---

# Create Your First Item

[Install Sertraline](installation.md) and use an operator account to try this example. Create `plugins/Sertraline/workspace/my_first_item.yml` and save the following:

```yaml
beginner_sword:
  xbuilder:
    material: IRON_SWORD
    name: "<!i><green>Beginner's Sword"
    lore: |
      <!i><gray>A sword for a new adventurer.
      <!i><yellow>Safe travels!
```

`beginner_sword` is the item ID used in commands. Use spaces for indentation and align fields at the same level. Here `material` selects an iron sword, `name` sets its display name, and `lore` supplies two description lines. `<!i>` removes the default italics.

In game, run:

```text
/sertraline reload
/sertraline item give beginner_sword
```

Check your inventory for the sword. To give two swords to an online player called Steve, use `/sertralineitem give beginner_sword Steve 2`. The [command reference](commands.md) lists the optional arguments. If the sword does not appear, check the reload output for your filename and verify its indentation, material, and item ID. The bundled examples are under `workspace/default/`.

## Customize an item

### Creating Files

Create your first item in the `plugins/Sertraline/workspace` directory. Currently, we support YAML, JSON and TOML format.

{% hint style="success" %}
You can use regex in `config.yml` to filter files.
{% endhint %}

{% hint style="success" %}
Siece `3.7.1`, Sertraline supports deep directory reading and nested folders.

You can even create files in deep directory structures like `plugins/Sertraline/workspace/chotentech/items/depazpills.yml`.
{% endhint %}

Now create a file in the `plugins/Sertraline/workspace` directory. Unlike other plugins such as `MMOItems`, the filename itself isn't critical. Let's name it `myfirstitem.yml`.

### Configuring the Item

Next, let's create a basic item. Write the following content in the configuration file:

```yml
Embian:
  # XBuilder Processor
  xbuilder:
    
    # Item material
    material: COOKED_BEEF
    
    # Item display name
    name: "<!i><#555555>Embian</#555555> &6Pills"
    
    # Custom Model Data
    custom-model-data: 10000
    
    # Item description
    lore: |
      <!i><gray>”though i guess it was designed to be like that… face without mouth emoji”
      <!i>&7-Ame
```

{% hint style="success" %}
Most text we use supports mixed Mini Message and Legacy formats.
{% endhint %}

{% hint style="success" %}
Use `<!i>` to remove default italic.
{% endhint %}

### Configuration Details

Although the configuration is quite simple, let's briefly analyze it:

* `material` - The base ItemStack used to construct the item.
* `name` - The display name of the item.
* `custom-model-data` - The item's Custom Model Data.
* `lore` - The item's description text. Using a string defaults the list to having only one entry. Similar to Ratziel, you can use `\n` within a single line, and we will automatically handle the line breaks.

Next, run `/sertraline reload` and use `/sertraline item give Embian` to Enjoy!

![image](https://github.com/user-attachments/assets/51175153-2a45-4820-b96a-d4f45cbb8c0d) ![image](https://github.com/user-attachments/assets/819bb686-796a-449b-8f81-bae757d5ba7c)

Looking for more configurations, such as custom model paths? Please refer to the subsequent reference section. This concludes the tutorial on creating your first item.

Wishing you success with your server!
