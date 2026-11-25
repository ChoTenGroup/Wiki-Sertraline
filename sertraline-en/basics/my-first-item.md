---
icon: gem
---

# My First Item

## First Item

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
* `model` - The item's Custom Model Data.
* `lore` - The item's description text. Using a string defaults the list to having only one entry. Similar to Ratziel, you can use `\n` within a single line, and we will automatically handle the line breaks.

Next, run `/sertraline reload` and use `/sertraline item give Embian` to Enjoy!

![image](https://github.com/user-attachments/assets/51175153-2a45-4820-b96a-d4f45cbb8c0d) ![image](https://github.com/user-attachments/assets/819bb686-796a-449b-8f81-bae757d5ba7c)

Looking for more configurations, such as custom model paths? Please refer to the subsequent reference section. This concludes the tutorial on creating your first item.

Wishing you success with your server!
