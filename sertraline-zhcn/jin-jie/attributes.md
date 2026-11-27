# 自定义属性

属性可以为装备增加数值。例如，你可以定义“专注”，玩家主手拿着法杖时增加 8 点。确认 `config.yml` 中的 `attribute.enable` 为 `true`。

## 创建属性与装备

把属性定义保存到 `plugins/Sertraline/attributes/practice.yml`：

```yaml
attributes:
  tutorial:focus:
    name: "专注"
    base: 10
    constraint:
      min: 0
      max: 100
```

把下面的物品保存到 `plugins/Sertraline/workspace/focus_wand.yml`：

```yaml
focus_wand:
  attribute-modifiers:
    - type: tutorial:focus
      id: tutorial:wand/focus
      amount: 8
      operation: minecraft:add_value
      slot: mainhand
  xbuilder:
    material: BLAZE_ROD
    name: "<!i><aqua>专注法杖"
```

执行 `/sertraline reload`，再用 `/sertralineitem give focus_wand` 领取。该属性的基础值是 10，法杖在主手时增加到 18。把法杖移出主手后会回到 10。自定义数值需要由描述、脚本或其他使用该属性的玩法读取。

## 调整加成

| 字段 | 用途 |
| --- | --- |
| `type` | 要修改的属性 ID |
| `id` | 这条加成的独立 ID。同一属性下应为需要同时生效的加成使用不同 ID |
| `amount` | 加成数值。乘法操作用小数表示比例，例如 `0.2` 表示 20% |
| `operation` | `minecraft:add_value` 加固定值，`minecraft:add_multiplied_base` 按该计算阶段开始时的值增加比例，`minecraft:add_multiplied_total` 乘以 `1 + amount` |
| `slot` | 生效位置，如 `mainhand`、`offhand`、`head`、`chest`、`legs` 或 `feet` |

更多配置见 `attributes/default.yml` 和 `workspace/default/default_attribute.yml`。

## 派生值、自定义运算阶段与基础值提供器

属性系统可以用其他属性计算派生值、定义自定义修饰阶段，也可以从实体的原版属性读取基础值：

```yaml
attributes:
  practice:armor_ward:
    name: 护甲屏障
    base:
      type: vanilla
      attribute: minecraft:armor
      fallback: 0
      transform: "value * 0.5"
  practice:zombie_armor:
    name: 僵尸护甲
    base:
      type: by_entity_type
      attribute: minecraft:armor
      fallback: 0
    entities: [minecraft:zombie]
  practice:power:
    name: 力量
    base: 10
    operations:
      - minecraft:add_value
      - practice:add_capped
  practice:combat_power:
    name: 战斗力
    derived: "practice:power * 2"
  practice:vitality:
    name: 活力
    base: 0
    entities: [minecraft:player]
    sync:
      - target: minecraft:max_health
        operation: add_value
        value: value

attribute_operations:
  practice:add_capped:
    expression: "MIN(current + amount, base * 2)"
```

`vanilla` 读取实体当前的原版属性基础值（修饰符生效前）；`by_entity_type` 读取 Paper 为该实体类型提供的默认值。若目标属性或平台不支持这些来源，则使用 `fallback`；`transform` 中的 `value` 是提供器读出的基础值。`entities` 可将定义限制到实体 ID 或 Bukkit 实体标签。`operations` 中的阶段按列表顺序运行；自定义表达式可使用 `base`（该阶段开始时的值）、`current` 和 `amount`。表达式支持 `MIN` 和 `MAX`。`sync` 则把计算结果作为原版属性修饰符应用到 `target`；它与读取原版基础值的 `base` 方向相反。

{% hint style="success" %}
`derived` 会根据引用的属性 ID 计算结果，不使用自身的基础值、物品修饰符、运算阶段或 `sync` 规则。若装备需要修改某个值，请使用普通属性。
{% endhint %}

## 试用默认战斗属性

在配有所附战斗监听器的现代服务端上，执行 `/sertralineitem give combat_feather_boots` 并穿上轻羽长靴。它提供 100 点摔落防御。默认战斗规则按 `伤害 × 100 / (100 + 防御)` 计算该分量，把摔落基础伤害减半。原版附魔等仍可参与最终扣血。

默认战斗包需要启用属性，且 `config.yml` 中的 `script.graaljs.allow-java-type` 必须为 `true`。样例物品在 `workspace/default/default_combat_weapons.yml` 和 `default_combat_armor.yml`，对应规则在 `global-listeners/default-combat.yml`。升级已有服务器时，检查这些文件是否已随当前版本安装。

### 在游戏里核对数值

在支持 GraalJS 的测试服中，将下面的动作加到 `focus_wand` 下，与 `xbuilder` 同级。重载并领取法杖后，切换到它所在的快捷栏槽位，等约一秒再右键。聊天栏应显示 18。

```yaml
sertraline:
  actions:
    onRightClick:
      - async: false
        graaljs: |
          player.sendMessage("专注 " + AttributeAPI.getValue(player, "tutorial:focus"));
```
