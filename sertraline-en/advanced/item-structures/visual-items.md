---
icon: network-wired
---

# Visual Items

Use the `visual` section to change what players see. It needs a server with data components, PacketEvents, and `packet.register: true` and `packet.packet-component: true` in `config.yml`.

## Make paper look like a diamond

Save this as `plugins/Sertraline/workspace/test_visual.yml`.

```yaml
test_visual:
  xbuilder:
    material: PAPER
    name: "<!i><white>Server-side paper"
    lore:
      - "<!i><gray>The client sees a diamond."
  visual:
    material: DIAMOND
```

Run `/sertraline reload` and `/sertralineitem give test_visual`. The inventory shows a diamond, while server-side recipes and mechanics use its paper material.

<figure><img src="../../.gitbook/assets/image (5).png" alt="Paper displayed with a diamond appearance"><figcaption></figcaption></figure>

## Show a player-specific name

This example needs PlaceholderAPI and its Player expansion. Put it at the top level of the same file. Reload and use `/sertralineitem give visual_player_name`. The displayed name includes the viewer's name.

```yaml
visual_player_name:
  xbuilder:
    material: PAPER
    name: "Server-side name"
  visual:
    auto_name: "<yellow>Hello, %player_name%!"
    auto_lore: |
      <gray>Each viewer sees their own name.
```

Use `auto_custom_model_data` for model data and `auto_model` for an item model ID. The model depends on the server and client versions and the resource pack. To supply a raw JSON text component, use `custom_name: '{"text":"Hello, %player_name%!"}'` inside `visual`.

<figure><img src="../../.gitbook/assets/image (1).png" alt="Visual name containing the viewer’s name"><figcaption></figcaption></figure>
<figure><img src="../../.gitbook/assets/image (2).png" alt="Visual item name and lore"><figcaption></figcaption></figure>
