---
icon: sensor-cloud
---

# Item Text Tags

Use `${tag:parameter}$` to insert data into an item name, lore, or another field that handles tags. This example displays the number of rounds, the rewards, and a contact. Save it to `plugins/Sertraline/workspace/event_pass.yml`, reload, and run `/sertralineitem give event_pass`.

```yaml
event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Event Pass"
    lore:
      - "<!i><gray>Rounds ${val:roundsAsIn}$"
      - "<!i><gray>Rewards ${val:rewardsAsLiSplitBy / }$"
      - "<!i><gray>Contact ${val:contact?:—}$"
  sertraline:
    vals:
      rounds: 2.7
      rewards: ["Gold", "Paper"]
```

The result displays `3` rounds and `Gold / Paper` as rewards. Since `contact` is missing, the contact line shows `—`. Add `contact: "Steve"` under `vals`, then reload and get a fresh pass to see the new value.

## Choose and format a value

- `val` reads `sertraline.vals`. Use `var` for data stored on the item and `dynamic` for values computed on demand. See [Item Data](item-structures/section-sertraline/item-data.md).
- Add `?:fallback` after the key to supply fallback text, or use `!!` to render an empty string in place of `null`. If the corresponding data section is entirely empty, the tag can remain as written. The example defines other entries in `vals`.
- Append `AsIn` to round to a whole number, `AsSt` for text, `AsDo` or `AsFl` for floating-point numbers, `AsBy` for a byte, `AsLo` for a long integer, `AsBo` for a Boolean, `AsBd` for a high-precision decimal, `AsLi` to join a list, or `AsMa` to join key-value pairs.

## Change list and map separators

Lists are comma-separated by default. `AsLiSplitBy / ` separates entries with a slash and surrounding spaces. Use `AsLiSplitByBLANK` to join with an empty separator. To join with a single space, write that space after `SplitBy`.

For a map, use a backtick between the two separators after `AsMaSplitBy`. The first separates a key from its value and the second separates entries. For example, ``${val:statsAsMaSplitBy=` / }$`` displays `{ hp: 10, speed: 2 }` as `hp=10 / speed=2`.


## Source check

Checked against plugin revision `032ee71` (November 26, 2026): the `AsIn` conversion rounds to the nearest integer; list and map separators are applied by their `AsLi` and `AsMa` conversions. Tags can remain literal when their entire source section is empty, so the examples define at least one value in each referenced section.

<details>
<summary>Archived original English text (preserved verbatim)</summary>

---
icon: sensor-cloud
---

# Item Text Tags

Item Text Tags allows you use Tag in item's anywhere.

### Usage

just use `${tag:param}$` in your item.

For example:

<pre><code>sword_of_lucky:
  sertraline:
    tier: S
    vars:
<strong>      damage: "%sertraline_random:40to80%"
</strong>  xbuilder:
    name: "&#x3C;!i>&#x3C;b>&#x3C;u>&#x3C;#9999ff>Mr. BlueICE's&#x3C;/u> &#x3C;yellow>&#x3C;i>Lucky Sword"
    lore: |
      &#x3C;!i>&#x3C;#66ccff>a sword contains random damage in generation.
    material: DIAMOND_SWORD
    attributes:
      ATTACK_DAMAGE:
<strong>        amount: "${var:damage}$"
</strong>        operation: ADD_NUMBER
        slot: HAND
</code></pre>

Then when item building, the `${var:damage}$` will be replaced.

### Advanced

#### Non-Null & Default

Use non-null suffix to avoid "null". Use `?:` to get default value. Example:

```
vars:
  test: null
```

* `"${var:test}$"` - `null`&#x20;
* `"${var:test!!}$"` -   (Empty String)
* `"${var:test?:DEF}$"` - `DEF` (Default Value)            &#x20;

#### As\<Type>

You can use As\<Type> Suffix to change a tag's type:

**Available Type Conversions:**

| Modifier | Code | Description                  | Example                                             |
| -------- | ---- | ---------------------------- | --------------------------------------------------- |
| `AsSt`   | `St` | Convert to String            | `${val:numberAsSt}$` → `"42"`                       |
| `AsIn`   | `In` | Convert to Integer (rounded) | `${val:priceAsIn}$` → `100`                         |
| `AsDo`   | `Do` | Convert to Double            | `${val:multiplierAsDo}$` → `1.5`                    |
| `AsFl`   | `Fl` | Convert to Float             | `${val:ratioAsFl}$` → `0.75f`                       |
| `AsBy`   | `By` | Convert to Byte              | `${val:flagAsByByte}$` → `1`                        |
| `AsLo`   | `Lo` | Convert to Long              | `${val:timestampAsLo}$` → `1234567890`              |
| `AsBo`   | `Bo` | Convert to Boolean           | `${val:enabledAsBo}$` → `true`                      |
| `AsBd`   | `Bd` | Convert to BigDecimal        | `${val:preciseAsBd}$` → `123.456789`                |
| `AsLi`   | `Li` | Convert List to String       | `${val:enchantmentsAsLi}$` → `"Fire,Ice,Lightning"` |
| `AsMa`   | `Ma` | Convert Map to String        | `${val:statsAsMa}$` → `"STR,50,DEX,30"`             |

```
vars:
 test: 1.99
```

* `"${var:testAsIn}$"` - `2`

### SplitBy Modifiers for Collections <a href="#splitby-modifiers-for-collections" id="splitby-modifiers-for-collections"></a>

The `SplitBy` modifier controls how collections (Lists and Maps) are joined into strings. It must be used together with `AsLi` or `AsMa` modifiers.

**Syntax:**

```
${prefix:keyAsLiSplitBySeparator}$     # For lists
${prefix:keyAsMaSplitByKV`entry}$      # For maps
```

**List Examples:**

| Tag                              | Input List                 | Output                           |
| -------------------------------- | -------------------------- | -------------------------------- |
| `${val:colorsAsLi}$`             | `["Red", "Blue", "Green"]` | `"Red,Blue,Green"` (default `,`) |
| `${val:colorsAsLiSplitBy }$`     | `["Red", "Blue", "Green"]` | `"Red Blue Green"` (space)       |
| `${val:colorsAsLiSplitBy-}$`     | `["Red", "Blue", "Green"]` | `"Red-Blue-Green"` (dash)        |
| `${val:colorsAsLiSplitByBLANK}$` | `["Red", "Blue", "Green"]` | `"RedBlueGreen"` (no separator)  |

**Map Examples:**

Maps use a two-part separator split by `` ` ``(3.8.2 below is  `|)`:

* First part: separates key from value
* Second part: separates entries

| Tag                              | Input Map            | Output                                   |
| -------------------------------- | -------------------- | ---------------------------------------- |
| `${val:statsAsMa}$`              | `{STR: 50, DEX: 30}` | `"STR,50,DEX,30"` (default `,` for both) |
| ``${val:statsAsMaSplitBy=`}$``   | `{STR: 50, DEX: 30}` | ``"STR=50`DEX=30"``                      |
| ``${val:statsAsMaSplitBy:`\n}$`` | `{STR: 50, DEX: 30}` | `"STR:50\nDEX:30"`                       |

**Special Values:**

* `SplitByBLANK` - Use empty string (no separator)
* `SplitBy` - Use single space

```
                "St" -> input.toString()
                "In" -> input.toString().toDoubleOrNull()?.roundToInt()
                "Do" -> input.toString().toDoubleOrNull()
                "Fl" -> input.toString().toFloatOrNull()
                "By" -> input.toString().toByteOrNull()
                "Lo" -> input.toString().toLongOrNull()
                "Bo" -> input.toBooleanTolerance()
                "Bd" -> input.toString().toBigDecimal()
                "Li" -> (input as? List<*>?)?.joinToString(sep)
                "Ma" -> {
                    val split = sep.split("`")
                    (input as? Map<*, *>?)?.joinToStringExtended(split[0], if (split.size > 1) split[1] else ",")
                }
```


</details>
