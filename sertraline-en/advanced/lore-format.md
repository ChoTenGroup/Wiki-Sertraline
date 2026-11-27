---
icon: tent-double-peak
---

# Lore Formats

Use one lore format to display type, tier, and level consistently across items. Save this configuration as `plugins/Sertraline/lore-formats/practice.yml`.

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

In the [item tier example](item-tier.md), change `classified_sword` from `lore-format: loreformat` to `lore-format: practice_lore`. Reload and get a fresh sword. Its original lore and the three classification lines appear in the order defined in `elements`.

## Change the displayed content

- Set `content` by itself to display fixed text. When you set a `key`, use `{value}` for its value. Lists produce one line per entry.
- `key: "sertraline:vals.note"` reads the item's `vals.note`. The `*` in `key: "*sertraline.vals.note"` tells the format to read directly from the item configuration along the dotted path. Use the normalized uppercase key for MMO stats, for example `mmo:ATTACK_DAMAGE`.
- Use `{val:note}` in a line to display the fixed value. To reuse one format across items, set the same `lore-format` ID under each item's `sertraline` section.

| Setting | Default | Effect |
| --- | --- | --- |
| `overwrite` | `true` | Replace the item's original lore. Set it to `false` to append |
| `visual` | `false` | Enable packet-based display. Requires PacketEvents and packet lore to be enabled |
| `auto-skip-blank` | `true` | Collapse consecutive blank lines |

## Show a line only when data exists

Add this entry to the end of `practice_lore`'s `elements` list. Add `vals: { note: "Event reward" }` under the sword's `sertraline` section. After reloading, the note line appears on items that supply a `note`.

```yaml
- content: "<!i><yellow>Note {val:note}"
  lineMode: ALL
  lineRequire:
    - "sertraline:vals.note"
```

When `lineRequire` lists multiple keys, `ANY` means at least one exists, `ALL` means all exist, `NOT` means none exist, and `NOT_ALL` means at least one is missing.

## Format numbers

| Syntax | Display |
| --- | --- |
| `{value}` | The original text of the value |
| `{auto}` | Removes a trailing `.0`, so `15.0` displays as `15` |
| `{plus}{auto}` | Adds a plus sign to positive numbers. Negative numbers keep their own minus sign |
| `{minus}` | Emits a standalone minus sign for negative numbers |
| `{round:2}` | Displays two decimal places in this version, so `15.567` displays as `15.57`. The number after the colon controls the field width |

Set `placeholders.plus` and `placeholders.minus` in `config.yml` to change the separate symbols. With player context, `content` can use PlaceholderAPI placeholders and inline Kether inside `{{ ... }}`. The bundled `lore-formats/compare_example.yml` and `lore-formats/switchable_example.yml` show equipment comparisons and lore switching.


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): `{round:N}` uses `N` as the format width and always formats to two decimal places. `lineRequire` checks item data or config paths according to the `*` prefix, with visibility controlled by `ANY`, `ALL`, `NOT`, or `NOT_ALL`.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

---
icon: tent-double-peak
---

# Lore Format

#### LoreSetting Properties <a href="#loresetting-properties" id="loresetting-properties"></a>

| Property    | Type      | Default | Description                                                                           |
| ----------- | --------- | ------- | ------------------------------------------------------------------------------------- |
| `overwrite` | `Boolean` | `true`  | If true, replaces existing lore; if false, appends to it                              |
| `visual`    | `Boolean` | `true`  | If true, applies during s2c (visual) transform; if false, applies to server-side item |
| `skipBlank` | `Boolean` | `true`  | If true, merges consecutive empty lines into single blank line                        |

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


</details>
