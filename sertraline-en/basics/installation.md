---
icon: floppy-disks
---

# Installation

### THINGS YOU MUST BE KNOW

{% hint style="danger" %}
**!! DANGER !!**

Under existing JS engines, some engines may have full access to all methods, including **sensitive methods** such as `java.io.File`. Therefore, if you unconditionally trust all the items you import, it may have catastrophic consequences.

\
Before importing any Sertraline items, crafting stations, or other packages, **it is essential to check for any malicious code** and promptly report it to the management personnel at the resource acquisition location.
{% endhint %}

### Requirements

**Sertraline** requires <mark style="background-color:blue;">**PAPER**</mark>**.** It is compatible with most common forks, including <mark style="background-color:green;">**FOLIA**</mark>, Pufferfish, Purpur , Leaf and similar variants. You need **JDK 21** or above for your server.

| Section              | Requirements                                                                                                                           |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Supported Core       | <mark style="background-color:blue;">**PAPER**</mark> and them's fork. Include <mark style="background-color:green;">**FOLIA**</mark>. |
| Tested Version       | **1.21.4** +                                                                                                                           |
| Required JDK Version | ≥ 21                                                                                                                                   |

To install the plugin, Just drag and drop the latest builds of **Sertraline** to your plugins folder. Do not forget dependencies:

| Depencendy Name    | Type                                                                 | Description                                                                                                                                                                                                                                             |
| ------------------ | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PlaceHolderAPI     | <mark style="background-color:orange;">**Highly Recommended**</mark> | For PlaceholderAPI hook and use PAPI In your items.                                                                                                                                                                                                     |
| Packetevents       | <mark style="background-color:blue;">**Optional**</mark>             | For packet-lore and packet DataComponents.                                                                                                                                                                                                              |
| MythicLib          | <mark style="background-color:blue;">**Optional**</mark>             | For MythicLib mmo Attributes.                                                                                                                                                                                                                           |
| Other Item Plugins | <mark style="background-color:blue;">**Optional**</mark>             | <p>Sertraline supports generate from other Item Plugins. You can find supported item plugin list</p><p><a href="https://github.com/ProjectUnified/UniItem/">HERE</a>(3.8.2-)</p><p><a href="https://github.com/jhqwqmc/ItemBridge">HERE</a>(3.8.3+)</p> |

If you want to request support other Attribute plugin or more, you can contact me at QQ or Discord.

Requests from paying users will be prioritized, but requests from free users will also be added within 1 week if there are no unexpected circumstances.

### Configuring the Plugin

If the plugin is installed successfully, you will see a directory named `Sertraline` in your server's plugins folder, with a structure as follows:

{% code fullWidth="false" %}
```
.../Sertraline
     ├── internal
     │   └── ...
     ├── lang
     │   ├── zh_CN.yml
     │   └── ...
     ├── crafting-stations
     │   ├── example.yml
     │   └── ...
     ├── workspace
     │   └── chotenpack
     │       └── test.yml
     ├── config.yml
     ├── kether.yml
     └── datasource.yml
```
{% endcode %}

Next, I will explain the purpose of each file and folder:

* `internal` - Internal files
* `lang` - Language files
* `crafting-stations` - Crafting station files
* `workspace` - Item storage / workspace
* `config.yml` - Main configuration file
* `kether.yml` - Kether-related settings
* `datasource.yml` - Datasource/database settings

