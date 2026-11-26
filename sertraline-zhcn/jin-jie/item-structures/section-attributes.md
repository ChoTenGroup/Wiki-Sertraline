# 物品属性配置

用 `xbuilder.attributes` 添加原版装备属性，用 `attribute-modifiers` 添加 Sertraline 属性，用 `mmo` 对接 MythicLib。下面使用默认 `demo:might` 和 `demo:vitality`，可在 `attributes/default.yml` 查看它们的定义。请启用 `config.yml` 的 `attribute.enable`。

## 固定值、比例和动态值

```yaml
guide_might_blade:
  xbuilder:
    material: IRON_SWORD
  attribute-modifiers:
    - type: demo:might
      id: guide:blade/might
      amount: 8
      operation: minecraft:add_value
      slot: mainhand
    - type: demo:might
      id: guide:blade/sneak_bonus
      amount: 0.2
      operation: minecraft:add_multiplied_base
      slot: mainhand
      scope: entity
      conditions:
        - "player.isSneaking()"
      update_interval: 10

guide_level_helmet:
  xbuilder:
    material: IRON_HELMET
  settings:
    attribute_modifiers:
      - type: demo:vitality
        id: guide:helmet/vitality
        amount:
          type: js
          script: "return 4 + player.getLevel() * 0.2;"
        operation: minecraft:add_value
        slot: head
```

两处列表的格式相同。同一物品如果同时填写 `attribute-modifiers` 和 `settings.attribute_modifiers`，使用前者。重载后分别领取剑和头盔。取剑到主手、切换潜行，或改变等级后戴上头盔，即可测试加成。如何查看数值见[自定义属性](../attributes.md)。

| 字段 | 怎么填 |
| --- | --- |
| `type` | 已经定义的属性 ID。可计算的派生属性不能直接添加修饰器 |
| `id` | 这条修饰器的唯一 ID，例如 `guide:blade/might`。同一属性需要叠加的修饰器用不同 ID |
| `amount` | 可填数字、表达式字符串，或填含 `type` / `engine` 和 `expression` / `script` 的对象。表达式用到的玩家为 `player` |
| `operation` | `minecraft:add_value` 加固定值，`minecraft:add_multiplied_base` 按该阶段开始时的基础值增加比例，`minecraft:add_multiplied_total` 乘以 `1 + amount`。也可用已定义的自定义运算 |
| `slot` | `mainhand` / `main_hand`、`offhand` / `off_hand`、`head` / `helmet`、`chest` / `chestplate`、`legs` / `leggings`、`feet` / `boots`。`hand` 匹配双手，`armor` 匹配护甲槽，`any` 或 `all` 匹配全部 |
| `scope` | 默认 `entity`，是装备者的属性。`weapon` 用于按武器读取的战斗属性，例如默认战斗监听器 |
| `conditions` / `condition` | 单个条件或列表，全部返回真才生效。支持表达式字符串，也支持 `{type: expression, expression: "player.isSneaking()"}` |
| `update_interval` | 动态值和条件的重新计算间隔，默认 20 tick，最小为 1 |

## 兼容旧版属性写法

`attribute` 可直接写属性名与数值。`4` 是固定值，`"15%"` 是百分比。`allowed` 指定生效槽位。省略或留空时按材质推断护甲槽，其他材质默认主手。`slot`、`source` 用于修饰器的槽位和来源信息。新配置可用上面的列表指定每一项的槽位、ID 和运算。

```yaml
guide_legacy_attribute:
  xbuilder:
    material: IRON_SWORD
  attribute:
    allowed: [mainhand, offhand]
    attack_damage: 4
    crit_chance: "15%"
```

<a id="chotenattributes"></a>

## ChoTenAttributes 数据

已安装读取此数据的插件时，可用 `chotenatb` 写入一组属性，也可用 `chotenatb_complex` 写入多组。两种写法都写入物品的 `chotenatb` 数据键，同一物品选择一种。简单写法省略 `allowed` 时会按护甲材质推断槽位，其他材质默认主手。复杂写法请逐组填写 `allowed`。

```yaml
guide_choten_sword:
  xbuilder:
    material: DIAMOND_SWORD
  chotenatb:
    attack_damage: 30
    max_health: "10%"
    allowed: [mainhand]

guide_choten_armor:
  xbuilder:
    material: DIAMOND_CHESTPLATE
  chotenatb_complex:
    - max_health: 10
      allowed: [chestplate]
    - movement_speed: "5%"
      allowed: [chestplate]
```

使用 MythicLib、MMOItems 或 MMOCore 的服务器请查阅 [MMO 属性示例](../mythiclib-mmoitems-compat.md)。

随附物品示例使用 `chotenatb` 或 `chotenatb_complex`。当前构建对旧 `choten` 段的支持仅限读取配置。
