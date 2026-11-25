---
icon: cards-blank
---

# 模板

多件物品共用材质、描述或属性时，可以把重复的配置放进模板。更新模板并重载后，重新领取物品就能看到变化。

## 用一份模板创建两把剑

先在 `plugins/Sertraline/templates/practice.yml` 定义一个模板。顶层名称 `practice:blade` 是模板 ID。

```yaml
practice:blade:
  xbuilder:
    material: "%{material?:IRON_SWORD}"
    name: "%{name?:<!i><green>练习剑}"
    unbreakable: true
    lore:
      - "<!i><gray>练习装备"
```

再在 `plugins/Sertraline/workspace/practice_blades.yml` 创建两件物品。

```yaml
practice_iron_blade:
  $template: practice:blade

practice_gold_blade:
  $template:
    use: practice:blade
    arguments:
      material: GOLDEN_SWORD
      name: "<!i><gold>黄金练习剑"
  xbuilder:
    lore:
      - "<!i><gray>活动奖励"
```

执行 `/sertraline reload`，再分别用 `/sertralineitem give practice_iron_blade` 和 `/sertralineitem give practice_gold_blade` 领取。铁剑使用模板的默认名称与材质，金剑使用 `arguments` 提供的名称与材质。两把剑都无法损坏。金剑的描述还会在模板原有描述后多出一行。

## 调整参数和覆盖方式

- `%{material}` 会读取 `arguments` 中的 `material`，`%{material?:IRON_SWORD}` 在参数缺失时使用 `IRON_SWORD`。文本、数字和布尔参数都可用于替换。
- `%{_ID}` 是当前文件的顶层条目 ID。例如金剑使用模板时，该值为 `practice_gold_blade`。
- 如果只想替换这个条目的描述，把 `$list-mode: replace` 放在与 `$template` 同级的位置。条目自己的同名字段优先，列表默认追加。
- 模板可通过 `$template` 引用其他模板。每个模板使用唯一名称，引用带命名空间的模板时写完整 ID。你也可以在支持模板的目录中复用它，例如 `recipes/` 或 `crafting-stations/`。

## 在填入模板前处理参数

可以在 `arguments` 里使用 `$t`。例如，把金剑原有的 `arguments` 替换为下面的配置，重载后它的名称会变成大写。

```yaml
arguments:
  name:
    $t:
      type: upper
      value: "gold practice blade"
  material: GOLDEN_SWORD
```

`$t` 还支持 `lower`、`when`、`condition` 和 `self_increase`，也可用 `kether`、`js`、`graaljs`、`fluxon` 或 `jexl` 计算结果。转换器按声明顺序求值，先写它需要引用的参数。`self_increase` 在每次模板展开时分配一个数值，同一次展开引用多次时值相同，重载会重置序列。

对于带 `%{damage}` 参数的模板，可在 `arguments` 下写成这样。这里 `ceil` 把 `10.2` 向上取整为 `11`，另有 `floor` 和 `round`。

```yaml
  damage:
    $c:
      type: int
      value: "10.2"
      round: ceil
```

`$c` 的输入依次取自同条参数上的 `$t` 运算结果、`$c.value` 或外层同名参数，转换失败可用 `fallback`。可用类型包括 `int`、`double` 和 `string`。在模板值的引号内使用这些参数时，下游配置仍会按各字段的规则解析。

完整示例见随附的 `templates/basic_gear.yml` 和 `workspace/default/example_templated_items.yml`。加载时报模板名重复、参数未使用或占位符未解析时，可从重载日志定位具体文件。开发者可以查阅 [SertralineAPI 的模板接口](../refrences/api/SertralineAPI.md)。
