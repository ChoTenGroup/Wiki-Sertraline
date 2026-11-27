---
icon: star
---

# Item Tiers

Tiers are named item-quality definitions loaded from `plugins/Sertraline/tiers/`. The default tier file is generated on first startup when this folder is missing. Each top-level YAML key is a tier ID; items reference that ID with `sertraline.tier`.

## Define a Tier

```yaml
# plugins/Sertraline/tiers/custom.yml
mythic:
  name: "<gold>Mythic</gold>"
  description: "A rare weapon."
  color: "<gold>"
  weight: 10
  meta:
    source: "quest"
```

`name` defaults to the tier ID, `description` to an empty string, `color` to `<white>`, and `weight` to `1`. `extra` and `meta` are optional maps.

## Assign a Tier to an Item

```yaml
my_sword:
  sertraline:
    tier: mythic
```

The value must match a tier ID that is loaded from the `tiers/` directory. The tier data can be used by lore formats through `{tier}`, `{tier:name}`, `{tier:id}`, `{tier:description}`, `{tier:color}`, and `{tier:weight}`. See [Lore Format](lore-format.md) for placeholder usage.

{% hint style="success" %}
Use the tier ID (`mythic`) in item data; `name` is the display text and may contain MiniMessage formatting.
{% endhint %}

**Implementation:** [tier file loader](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/config/loadTierFiles.kt) · [tier data model](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/data/Tier.kt) · [tier feature handler](https://github.com/zzzyyylllty/Sertraline-Hydrochloride/blob/032ee71ebd935552337e02371ba39e19294d65d1/project/common/src/main/kotlin/io/github/zzzyyylllty/sertraline/listener/sertraline/featureloader/getFeatureTier.kt)
