---
icon: cards-blank
---

# Templates

Use a template when several items share settings such as material, lore, or attributes. After editing the template and reloading, get a fresh copy of an item to see the change.

## Create two swords from one template

Define a template in `plugins/Sertraline/templates/practice.yml`. Its top-level key, `practice:blade`, is the template ID.

```yaml
practice:blade:
  xbuilder:
    material: "%{material?:IRON_SWORD}"
    name: "%{name?:<!i><green>Practice Sword}"
    unbreakable: true
    lore:
      - "<!i><gray>Practice equipment"
```

Create two items in `plugins/Sertraline/workspace/practice_blades.yml`.

```yaml
practice_iron_blade:
  $template: practice:blade

practice_gold_blade:
  $template:
    use: practice:blade
    arguments:
      material: GOLDEN_SWORD
      name: "<!i><gold>Gold Practice Sword"
  xbuilder:
    lore:
      - "<!i><gray>Event reward"
```

Run `/sertraline reload`. Use `/sertralineitem give practice_iron_blade` and `/sertralineitem give practice_gold_blade` to get the swords. The iron one uses the default name and material from the template. The gold one uses the values from `arguments` and adds a line after the shared lore. Both swords are unbreakable.

## Change arguments and overrides

- `%{material}` uses the `material` in `arguments`. `%{material?:IRON_SWORD}` supplies an iron sword as the fallback. You can substitute text, numbers, and Boolean values.
- `%{_ID}` uses the current file's top-level entry ID. For the gold sword its value is `practice_gold_blade`.
- To replace the shared lore for an item, put `$list-mode: replace` alongside `$template`. The item's own fields take precedence for matching keys. Lists append by default.
- Templates can use `$template` to build on other templates. Give each template a unique ID and include its namespace when referencing it. Templates also work in supported folders such as `recipes/` and `crafting-stations/`.

## Process an argument before inserting it

Use `$t` in `arguments` to transform an argument. Replace the gold sword's current `arguments` with the following and reload to see its name in uppercase.

```yaml
arguments:
  name:
    $t:
      type: upper
      value: "gold practice blade"
  material: GOLDEN_SWORD
```

Other `$t` options include `lower`, `when`, `condition`, and `self_increase`. You can also compute a value with `kether`, `js`, `graaljs`, `fluxon`, or `jexl`. Transforms are evaluated in declaration order, so place the arguments they depend on first. `self_increase` assigns a value for each template expansion. Multiple references in that expansion share the value. A reload resets the sequence.

For a template with a `%{damage}` argument, add this under `arguments`. Here `ceil` rounds `10.2` up to `11`. The integer converter also supports `floor` and `round`.

```yaml
  damage:
    $c:
      type: int
      value: "10.2"
      round: ceil
```

A `$c` cast uses the result of a `$t` on the same argument, then `$c.value`, then the same named argument from an outer template. Set `fallback` for a failed cast. Types include `int`, `double`, and `string`. Each configuration field still applies its own parsing rules after substitution.

See the bundled `templates/basic_gear.yml` and `workspace/default/example_templated_items.yml` for more examples. Check the reload log for duplicate template names, unused arguments, and unresolved placeholders.

## Developer API

#### Template Access (via `SertralineAPI`)

```kotlin
val api = Sertraline.api()

// Query
val template: Map<String, Any?>? = api.getTemplate("my_weapon_template")
val names: Set<String> = api.getTemplateNames()
val all: Map<String, Map<String, Any?>> = api.getAllTemplates()
val count: Int = api.getTemplateCount()

// Resolve
val resolved = api.resolveTemplate("my_weapon_template", mapOf("name" to "Test"))
```

#### Custom Processor Registration

Register custom `$t` transformer types:

```kotlin
api.registerTransformer("my_type") { type, config, resolveCtx ->
    // type: "my_type"
    // config: the full $t map
    // resolveCtx: current args including _ID, _ORIGIN
    "transformed value"
}

api.unregisterTransformer("my_type")
```

Register custom `$c` converter types:

```kotlin
api.registerConverter("my_format") { type, config, value, resolveCtx ->
    // value: the input string to convert
    "converted value"
}

api.unregisterConverter("my_format")
```

Register custom argument directives (same level as `$t`/`$c`):

```kotlin
api.registerDirective("\$myDirective") { name, config, key, currentArgs, dynamicArgs, resolveCtx ->
    // Process like $t/$c but with full control over currentArgs/dynamicArgs
}

api.unregisterDirective("\$myDirective")
```

The functional interfaces (defined in `TemplateManager`):

```kotlin
fun interface TransformerProvider {
    fun process(type: String, config: Map<String, Any?>, resolveCtx: Map<String, String>): String
}

fun interface ConverterProvider {
    fun process(type: String, config: Map<String, Any?>, value: String, resolveCtx: Map<String, String>): String
}

fun interface DirectiveProvider {
    fun process(name: String, config: Map<String, Any?>, key: String, currentArgs: MutableMap<String, String>, dynamicArgs: MutableMap<String, () -> String>, resolveCtx: Map<String, String>)
}
```
