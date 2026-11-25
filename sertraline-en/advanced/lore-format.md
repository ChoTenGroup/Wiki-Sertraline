---
icon: tent-double-peak
---

# Lore Formats

Reuse a lore format to show consistent type, tier, and level lines across several items. Save this as `plugins/Sertraline/lore-formats/practice.yml`:

```yaml
practice_lore:
  settings:
    overwrite: true
    visual: false
    auto-skip-blank: true
  elements:
    - key: "xbuilder:lore"
      content: "{value}"
    - key: "sertraline:type"
      content: "<!i><gray>Type {type:name}"
    - key: "sertraline:tier"
      content: "<!i><gray>Tier {tier:format}"
    - key: "sertraline:level"
      content: "<!i><gray>Level {level:name}"
```

In the [classification example](item-tier.md), change `classified_sword` from `lore-format: loreformat` to `lore-format: practice_lore`. Reload and give yourself another sword. Its lore should show the original description followed by the three labels.

## Customize individual lines

- Set `content` alone for static text. Add `key` to read item data. When that value is a list, the format renders one line per entry using `{value}`.
- `key: "sertraline:vals.note"` reads `note` from the resolved item data. `key: "*sertraline.vals.note"` reads it from the configuration using a dotted path.
- List required keys under `lineRequire` and use `lineMode` to choose `ANY` (at least one exists), `ALL` (all exist), `NOT` (none exist), or `NOT_ALL` (at least one is missing).
- For numeric values, use `{plus}` and `{minus}` for signs, `{auto}` for the unsigned number, or `{round:2}` for rounding. `{value}` retains the original value. Player contexts also support PlaceholderAPI and inline Kether.

| `settings` field | Default | Use |
| --- | --- | --- |
| `overwrite` | `true` | Replace existing lore. Set to `false` to append |
| `visual` | `false` | Writes lore onto the item by default. Packet-based display requires PacketEvents and `packet.packet-lore` |
| `auto-skip-blank` | `true` | Collapse consecutive blank lines |

The bundled `lore-formats/compare_example.yml` and `lore-formats/switchable_example.yml` show how to configure comparison and switchable lore.

#### LoreSetting Properties <a href="#loresetting-properties" id="loresetting-properties"></a>

| Property    | Type      | Default | Description                                                                           |
| ----------- | --------- | ------- | ------------------------------------------------------------------------------------- |
| `overwrite` | `Boolean` | `true`  | If true, replaces existing lore; if false, appends to it                              |
| `visual`    | `Boolean` | `false`  | If true, applies during s2c (visual) transform; if false, applies to server-side item |
| `auto-skip-blank` | `Boolean` | `true`  | If true, merges consecutive empty lines into single blank line                        |

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
Some key is different with config's section. for example is MMO Attributes: `attack-damage` -> `ATTACK_DAMAGE`
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

### Placeholder Processing System <a href="#placeholder-processing-system" id="placeholder-processing-system"></a>

The lore formatting system supports three layers of placeholder processing applied in sequence.

```
```

**Sources:** [project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt54-101](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/7fbdc299/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/util/loreformat/handleLoreFormat.kt#L54-L101)

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
