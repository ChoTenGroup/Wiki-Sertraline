---
icon: tent-double-peak
---

# Lore Format

#### LoreSetting Properties <a href="#loresetting-properties" id="loresetting-properties"></a>

| YAML setting        | Type      | Default | Description                                                                 |
| ------------------- | --------- | ------- | --------------------------------------------------------------------------- |
| `overwrite`         | `Boolean` | `true`  | Replace existing lore; when `false`, append generated lines                 |
| `visual`            | `Boolean` | `false` | Apply the format to the packet/client-side view instead of server-side lore |
| `auto-skip-blank`   | `Boolean` | `true`  | Collapse consecutive empty lines into a single blank line                   |

{% hint style="warning" %}
`visual: true` requires PacketEvents and `packet.packet-lore: true` in Sertraline's main configuration. Without both, the format is treated as non-visual and the plugin logs a warning.
{% endhint %}

#### Key Iteration Examples <a href="#key-iteration-examples" id="key-iteration-examples"></a>

**Without Key (Static Line):**

```
content: "<gray>═══════════════════"
```

Produces a single static line.

**With Key (Dynamic Lines):**

```
key: "sertraline:vals.test"
content: "<green>{plus}{value} {name}"
```

If `sertraline:attributes` contains `[10, 20, 30]`, produces three lines with placeholder substitution for each value.

{% hint style="warning" %}
Use the key path and spelling expected by its data source; for example, the MMO attribute example below uses `mmo:ATTACK_DAMAGE`.
{% endhint %}

**With Config Key:**

```
key: "*xbuilder.lore"
content: "{value}"
```

Retrieves lore from the item's `config section` instead of runtime data.

### Examples

```yaml
default:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - content: ""
    - key: "mmo:ATTACK_DAMAGE"
      content: "<!i><#777777>| <#ff6936>Attack Damage <#aabbcc>{plus}{minus}{auto}"
    - key: "mmo:ATTACK_SPEED"
      content: "<!i><#777777>| <#26cc79>Attack Speed <#aabbcc>{plus}{minus}{auto}"
    - content: ""
    - key: "sertraline:vals.lore"
      content: "<!i><#777777>| <#cccccc>{value}"
    - key: "sertraline:vals.graylore"
      content: "<!i><#777777>| {value}"
```

```yaml
wooden_sword:
  sertraline:
    tier: C
    lore-format: default
    vals:
      lore: |
        Common wood sword.
      graylore: |
        Just another description
  xbuilder:
    name: "<!i><#bb9900>Wooden Sword"
    material: WOODEN_SWORD
    flags: [HIDE_ATTRIBUTES]
    item-model: "kangelitem:wooden_sword"
  mmo:
    fix-attack-speed: true
    slot: "MAIN_HAND"
    allowed:
      - "hand"
    attack-damage: 5
    attack-speed: 1.6
```

### Advanced

### Conditional Line Rendering <a href="#conditional-line-rendering" id="conditional-line-rendering"></a>

The `LineMode` enum determines how multiple required keys are evaluated. The system uses `handleLoreExists()` to check if a `LoreElement` should be displayed based on its `lineRequire` list.

#### LineMode Behavior Table <a href="#linemode-behavior-table" id="linemode-behavior-table"></a>

| LineMode  | Condition                                        | Result       |
| --------- | ------------------------------------------------ | ------------ |
| `ANY`     | At least one key in `lineRequire` exists         | Display line |
| `ALL`     | All keys in `lineRequire` exist                  | Display line |
| `NOT`     | No keys in `lineRequire` exist                   | Display line |
| `NOT_ALL` | At least one key in `lineRequire` does not exist | Display line |

The `handleExistLore()` function checks key existence. If the key starts with `*`, it queries the item's config section; otherwise, it queries the item's data via `getDeepData()`.

{% hint style="info" %}
When `lineRequire` is empty or omitted, a line is shown if `lineMode` is omitted, `ALL`, or `NOT`. With an empty requirement list, `ANY` and `NOT_ALL` do not show the line.
{% endhint %}

### Placeholder Processing System <a href="#placeholder-processing-system" id="placeholder-processing-system"></a>

For a keyed element, Sertraline processes placeholders in this order:

1. **Value and item paths:** `{plus}`, `{minus}`, `{value}`, `{auto}`, `{round:N}`, `{data:path}`, and `{config:path}`. Value placeholders use each resolved value; `data:` reads item data and `config:` reads the item's configuration tree.
2. **General Sertraline tags:** tier, type, level, `val`, `var`, and `context` placeholders.
3. **PlaceholderAPI:** `%...%` placeholders are expanded when a player context is available.
4. **Inline Kether:** text inside `{{...}}` is evaluated last.

Static elements have no keyed value, but still pass through the general tag, PlaceholderAPI, and Kether processing stages.

**Implementation:** [lore-format YAML parser](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/config/loadLoreFormatFiles.kt#L141-L159) · [placeholder handling](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt#L619-L751) · [line requirements](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt#L758-L861)

#### Built-in Placeholder Types <a href="#built-in-placeholder-types" id="built-in-placeholder-types"></a>

The `performNormalPlaceholders()` function processes special placeholders based on the iterated value:

| Placeholder | Behavior                                | Example                           |
| ----------- | --------------------------------------- | --------------------------------- |
| `{plus}`    | Shows `+` if value > 0, empty otherwise | `15` → `+`                        |
| `{minus}`   | Shows `-` if value < 0, empty otherwise | `-5` → `-`                        |
| `{value}`   | Raw string value                        | `15.5` → `15.5`                   |
| `{auto}`    | Removes `.0` suffix from integers       | `15.0` → `15`                     |
| `{round:N}` | Formats to N decimal places             | `{round:2}` on `15.567` → `15.57` |

The `plus` and `minus` symbols can be customized in `config.yml` under `placeholders.plus` and `placeholders.minus`.

#### PlaceholderAPI Integration <a href="#placeholderapi-integration" id="placeholderapi-integration"></a>

After built-in placeholders, the `performPlaceholders()` function applies PlaceholderAPI replacements if a player context is available.

This allows standard PAPI placeholders like `%player_name%`, `%player_health%`, etc.

#### Inline Kether Evaluation <a href="#inline-kether-evaluation" id="inline-kether-evaluation"></a>

If the content contains `{{`, the system treats text within `{{...}}` as inline Kether scripts.

Like:  `{{ player name }}`&#x20;

This allows dynamic computation in lore templates.&#x20;

