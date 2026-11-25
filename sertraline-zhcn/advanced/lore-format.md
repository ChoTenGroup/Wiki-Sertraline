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

