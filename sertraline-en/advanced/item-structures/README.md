# Item Configuration Sections

An item can have several sections. Start with [Item Structure](../../basics/item-structure.md) and add XBuilder, Sertraline, Minecraft data components, or custom data as needed. Put each section under the item ID at the same indentation as `xbuilder`.

## Restrict how an item can be used

If an event token uses paper as its material, you can prevent players from spending it in crafting. Save this as `plugins/Sertraline/workspace/event_only_token.yml`.

```yaml
event_only_token:
  xbuilder:
    material: PAPER
    name: "<!i><gold>Event-Only Token"
  common:
    disable-crafting: true
```

Reload and run `/sertralineitem give event_only_token`. Try using the token in place of one piece of paper in a book recipe. The output slot stays empty. Replace it with plain paper to make the book.

Restrictions are off by default and apply per item. A custom crafting station uses its own ingredient configuration.

| Field under `common` | What it controls |
| --- | --- |
| `disable-crafting` | Set to `true` to block use as an ingredient in the crafting table and inventory grid. Use `vanilla` to block vanilla recipes only |
| `disable-smelting` | Set to `true` to block use as a smelting input |
| `disable-smithing` | Set to `true` to block use as a smithing input. Requires a modern build |
| `disable-village-trade` | Set to `true` to block selection of a villager trade that explicitly requires this item. Requires a modern build |
