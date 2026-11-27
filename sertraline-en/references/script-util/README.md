---
icon: subscript
---

# Script Util

### Utility Reference Table <a href="#utility-reference-table" id="utility-reference-table"></a>

When a script context injects Sertraline’s default script variables, the following built-in names are available. Integrations can add more variables:

| Variable Name           | Type                       | Purpose                                              |
| ----------------------- | -------------------------- | ---------------------------------------------------- |
| `mmUtil`                | MiniMessage Utility        | Deserialize MiniMessage format strings to Components |
| `mmJsonUtil`            | MiniMessage JSON Utility   | Handle JSON-format MiniMessage                       |
| `mmLegacySectionUtil`   | Legacy Section Utility     | Convert legacy § color codes to Components           |
| `mmLegacyAmpersandUtil` | Legacy Ampersand Utility   | Convert legacy & color codes to Components           |
| `jsonUtils`             | Gson JSON Utility          | Serialize/deserialize JSON data                      |
| `ItemStackUtil`         | ItemStack Helper           | Utility methods for ItemStack manipulation           |
| `EventUtil`             | Event Helper               | Utility methods for event handling                   |
| `ThreadUtil`            | Thread Helper              | Thread management utilities                          |
| `PlayerUtil`            | Player Helper              | Player-specific utility methods                      |
| `SertralineAPI`         | SertralineAPI instance     | Access to Sertraline's public API                    |
| `DataUtil`              | DataUtil Object            | Player data storage and cooldown management          |
| `Math`                  | java.lang.Math Class       | Mathematical operations                              |
| `System`                | java.lang.System Class     | System-level operations                              |
| `Bukkit`                | org.bukkit.Bukkit Class    | Bukkit server API                                    |
| `Gson`                  | `Class<Gson>`              | Gson class reference (not a preconfigured Gson instance) |
| `ExternalItemHelper`     | External item helper       | Resolve or build supported third-party item references |
| `AttributeAPI`           | Attribute API object       | Access Sertraline attribute operations                 |
| `bukkitPlugin`           | Bukkit plugin instance     | Access the running plugin instance                     |
| `scheduler`              | Bukkit scheduler           | Schedule server tasks                                   |
| `SertralineObj`          | Sertraline object          | Access the plugin singleton                            |
| `FluxonShell`            | Fluxon shell helper        | Access Fluxon scripting operations                     |
| `UUIDUtil`               | UUID utility               | UUID-related helpers                                   |
| `LoreFormatUtil`         | Lore format utility        | Lore-format operations                                 |
| `TierUtil`               | Tier utility               | Query tier data                                        |
| `TypeUtil`               | Type utility               | Query type data                                        |

{% hint style="warning" %}
Some script contexts expose Java/server classes such as `System` and `Bukkit`. Treat scripts as trusted code, and perform player or world operations on the server thread.
{% endhint %}
