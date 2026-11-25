# Gem System

The gem system gives Sertraline items **sockets**. Socketing a gem item applies **data alterations** to the host item (variables, name, lore, attributes, …). It is built on top of `sertraline:alteration`, so gem effects survive every `rebuild`.

## Try the bundled sword and ruby

The bundled examples are in `workspace/default/default_gem.yml`. Give yourself the sword and ruby:

```text
/sertralineitem give gem_example_sword
/sertralineitem give gem_example_ruby
```

Put the sword in your main hand. Keep the ruby in your inventory or off hand. Run `/sertralinegem socket slot_red gem_example_ruby` to consume the ruby and fill the slot. Check the result with `/sertralinegem list` and use `/sertralinegem unsocket slot_red` to return the ruby. The attack number in the lore changes from 100 to 115 and back. This number is the example's custom variable. To use it in combat, configure an attribute or action that reads it.

---

## 1. Declaring sockets on an item

Sockets live under `sertraline:gem-slots` of the host item; each key is a socket id:

```yaml
gem_example_sword:
  sertraline:
    vars:
      atk: 100
    gem-slots:
      slot_red:
        name: "<red>Red Socket"            # socket name, used in messages and lore
        desc: "<gray>Can apply a red gem."  # shown while the socket is empty
        gem-color: "red"                   # regex; omit / blank = any color
      slot_any:
        name: "<light_purple>Prismatic Socket"
        desc: "<gray>Accepts any gem color."
  xbuilder:
    name: "<!i>&cGem Example Sword"
    material: DIAMOND_SWORD
    lore: |
      <!i><gray>Attack: <red>${var:atkAsIn}$
      <!i>${gem-slot:slot_red}$
      <!i>${gem-slot:slot_any}$
```

- `gem-color` is a **regex** matched against the gem color (`red|blue` works); leave it out to accept every color.
- A socket id holds exactly one gem.

## 2. Declaring gem behaviour

```yaml
gem_example_ruby:
  sertraline:
    gem:
      color: red
      meta:                    # optional, free-form
        note: "A red gem."
      modify-data:             # applied to the host item on socket
        sertraline:
          vars:
            atk: "+15"
  xbuilder:
    name: "<!i><red>Ruby Gem"
    material: RED_DYE
```

Two equivalent spellings of `modify-data`:

```yaml
modify-data:
  sertraline:                  # nested — flattened to sertraline:vars.atk
    vars:
      atk: "+15"
```

```yaml
modify-data:
  "sertraline:vars.atk": "+15" # flat — already an item data key
```

Leaf value semantics:

| Value | Meaning |
| --- | --- |
| `"+15"` / `"-5"` / `"+2.5"` | Add to / subtract from the current number (missing target counts as 0) |
| `"set:value"` | Set the value |
| `"remove"` | Remove the key |
| `"list_add:v"` / `"list_add_last:v"` / `"list_remove:i"` / `"list_remove_value:v"` / `"list_remove_all:v"` / `"list_clear"` | List operations |
| `"map_set:k=v"` / `"map_remove:k"` / `"map_clear"` | Map operations |
| anything else (`100`, `"hello"`, `true`, …) | Set (SET) |

Operation names are case-insensitive and treat `-` like `_` (`list-add:v` = `list_add:v`).

> Keys must resolve to `namespace:path`. Only namespaces the item builder understands (`sertraline`, `xbuilder`, `minecraft`, …) take effect; a path without a namespace is written as a literal top-level data key and ignored at build time.

## 3. Socketing and removing

### Interaction (configurable)

Click-to-socket is driven by `gem.interaction.click` in `config.yml`:

```yaml
gem:
  interaction:
    click:
      enable: true        # master switch
      types:              # gestures that socket a gem
        - RIGHT           # LEFT / RIGHT / SHIFT_LEFT / SHIFT_RIGHT / DOUBLE_CLICK
      permission: ""      # required permission, blank = none
```

On top of that, click socketing can be opted out per **item / socket / gem** (commands and the API are unaffected):

```yaml
# 1. the whole item rejects click socketing
some_item:
  sertraline:
    gem-click: false
    gem-slots:
      # 2. a single socket rejects click socketing
      slot_locked:
        name: "<gray>Locked Socket"
        click: false
      slot_red:
        name: "<red>Red Socket"
        gem-color: "red"

# 3. the gem itself rejects click socketing
gem_example_plain:
  sertraline:
    gem:
      color: white
      click: false
```

`click` accepts `true/false`, `yes/no`, `on/off`, `1/0`, `enable/disable`, `allow/deny` (alias `click-socket`; item level uses `gem-click` / `gem-click-socket`). Any `false` refuses the click route and shows `gem-socket-denied`.

### Developer reference

> New interaction routes (a GUI, a crafting station, a kether function, …) only need a new `GemSocketMethod` entry plus its rule in `GemSocketPolicy.denialOf`; `socketGem(..., method)` applies the definition-level flags per route automatically.

### Commands

| Command | Description |
| --- | --- |
| `/sertralinegem list` | List sockets and socketed gems of the held item |
| `/sertralinegem socket <slotId> [gemId]` | Socket a gem with the given ID from your inventory or off hand. Omit the ID to use the off-hand gem |
| `/sertralinegem unsocket <slotId>` | Extract the gem back to the player |

The alias is `/sgem`. The permission is `sertraline.command.gem`, granted to operators by default. Command socketing works independently of the click settings.

### Where the effects go

- Alterations targeting `sertraline:vars.*` are **folded into** `sertraline_data` (permanent) and reversed on unsocket using the recorded previous value / the negated delta.
- Everything else (`xbuilder:name`, lore, attributes, …) is replayed on every rebuild, so simply stopping the replay restores the original item.

## 4. NBT layout

```yaml
sertraline_gem:
  used: ["slot_red"]        # occupied sockets — the single source of truth
  slots:
    slot_red:
      gem: "gem_example_ruby"  # Sertraline id of the socketed gem
      timestamp: 1791764589    # unix millis
```

The default `rebuild.keep-data` list includes `sertraline_gem`. Keep this entry when customizing the list.

## 5. Lore placeholder

`${gem-slot:<slotId>}$` renders at build time:

- empty socket → `◇ <name> — <desc>`
- occupied socket → `◆ <name>: <gemId>`

Use one line per socket, for example `<!i>${gem-slot:slot_red}$`. The `<!i>` prefix removes default italics.

## 6. FAQ

**The gem has no effect.** Check that the `modify-data` key is a namespaced path such as `sertraline:vars.xxx`; nested maps are now flattened to exactly that form.

**`+15` does nothing.** A missing target variable now counts as 0, so re-check the key spelling.

**Do private (per-player) items work?** Yes — socket and gem definitions resolve through `itemTemplateForStack`, which understands private templates.

**Can I socket twice into one socket?** No: an occupied socket returns `SLOT_OCCUPIED`; unsocket first.
