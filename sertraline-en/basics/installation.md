---
icon: floppy-disks
---

# Installation and Configuration

The Standard and Spigot builds target the Minecraft 1.21.4 API on Java 21. The Legacy build targets Minecraft 1.12.2 on Java 8 and omits modern recipes and data components. Use the build for your server platform and check the release notes for the version you plan to install. [Sertraline releases](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/releases)

PlaceholderAPI enables external placeholders. PacketEvents 2.10 or newer is required for packet-based lore and visual components. A basic custom item works without either plugin. MythicLib integrates MMO attributes when used with MMOItems or MMOCore. ItemBridge provides references to items from other plugins. [ItemBridge supported providers](https://github.com/jhqwqmc/ItemBridge)

1. Place the chosen Sertraline JAR in the server's `plugins/` folder.
2. Start the server and check its console for Sertraline's load messages. The plugin creates `plugins/Sertraline/` on first use.
3. To change command messages to Simplified Chinese, set `lang: zh_CN` in `plugins/Sertraline/config.yml` and run `/sertraline reload`.

| Path under `plugins/Sertraline/` | Use |
| --- | --- |
| `workspace/` | Item definitions and bundled examples under `workspace/default/` |
| `templates/`, `lore-formats/` | Reusable configuration and item lore layouts |
| `recipes/`, `crafting-stations/` | Vanilla-interface recipes and custom crafting menus |
| `tiers/`, `types/`, `levels/` | Item classification |
| `attributes/` | Custom attribute definitions |
| `custom-events.yml`, `global-listeners/`, `scripts/` | Custom item triggers, server-wide listeners, and reusable scripts |
| `lang/`, `internal/` | Language files and internal resources |
| `config.yml`, `manager.yml`, `experimental.yml` | Plugin options, item manager defaults, and experimental options |

{% hint style="success" %}
Item files are found recursively under `workspace/` (up to 10 directory levels). The default `file-load.item` regex in `config.yml` matches the **filename** and accepts `.yaml`, `.yml`, `.toml`, `.tml`, `.json`, and `.conf`. If a file is not loaded, check its extension and this filter, then review the reload log.
{% endhint %}

Run `/sertraline reload` after changing runtime settings. Changes to database or metrics settings require a server restart. Packet handlers that were disabled at startup also require a restart to enable. Review scripts before importing third-party item packs because scripts can access server resources.

Next, [create your first item](my-first-item.md) or look up the [common commands](commands.md).

## Optional dependencies

| Plugin | When to install it |
| --- | --- |
| PlaceholderAPI | Resolve supported placeholders, such as player values in item lore |
| PacketEvents 2.10+ | Display packet-based lore, materials, and components |
| MythicLib with MMOItems or MMOCore | Use MMO attributes |
| An ItemBridge-supported item plugin | Reference its items in recipes and crafting stations |

Use the community to request integration with an additional item or attribute plugin. [Services](../description/services.md) · [Discord community](https://discord.gg/VHs958jJXj)

## Change common settings

Edit the existing settings in `config.yml`. Keep the other keys in the same section and give each YAML key only one entry.

| Setting | Effect |
| --- | --- |
| `debug: true` | Show more diagnostics for reload and script issues |
| `metrics.enabled: false` | Turn off the plugin's FastStats reporting after a restart |
| `item-update.enabled: false` | Turn off automatic updates to existing items |
| `packet.packet-lore: false` | Turn off packet-based lore. Ordinary lore stored on items remains available |
| `database.enable: false` | Use SQLite. To use MySQL, set it to `true`, provide the connection details, and restart |

To skip scanning online players' items on every reload, set `item-update.triggers.on-reload` to `false`. Login, held-item changes, and other update checks have their own switches.
