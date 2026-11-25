---
icon: wrench
---

# 自定义配方

在 `plugins/Sertraline/recipes/` 里创建配方，就能让玩家用工作台、熔炉等原版设施制作物品。该功能适用于现代构建。需要独立菜单、背包扣料或自定义制作计时，可以创建[合成站](crafting-stations.md)。

## 试用自带配方

打开 `recipes/sample.yml`，找到 `star_from_vanilla`。它使用 `workspace/default/default_misc.yml` 中的 `star_of_recall` 作为产物。下面省略了使用默认值的 `provider`。

```yaml
star_from_vanilla:
  format: SHAPED
  type: CRAFTING
  pattern:
    - "PPP"
    - "PGP"
    - "PPP"
  ingredients:
    P: "minecraft:paper"
    G: "minecraft:gold_ingot"
  result:
    id: "sertraline:star_of_recall"
    count: 1
```

在工作台中央放一块金锭，周围八格各放一张纸，即可制作回忆之星。`pattern` 的每行对应网格的一行，`ingredients` 定义每个字母代表的材料，空格表示空槽。要改这条随附配方，可直接编辑该条目。

## 添加无序配方

按[新手教程](../ji-chu/my-first-item.md)创建 `beginner_sword`，再将下面的配置保存为 `plugins/Sertraline/recipes/practice.yml`。

```yaml
practice_sword_recipe:
  type: CRAFTING
  format: SHAPELESS
  ingredients:
    - "minecraft:iron_ingot"
    - "minecraft:iron_ingot"
    - "minecraft:stick"
  result:
    id: "sertraline:beginner_sword"
    count: 1
```

执行 `/sertraline reload`。把两块铁锭和一根木棍分别放入三个格子，顺序任意，即可得到新手之剑。原版设施的每个配方槽消耗一件物品。`amount` 可省略，写成大于 1 的值会导致注册失败。无序配方要重复列出相同材料，有序配方则在图案里重复对应字母。

## 添加熔炉配方

把下面的条目加到 `recipes/practice.yml` 的顶层。重载后将泥巴和燃料放入熔炉，等待烧成红砖。正常速度下 200 游戏刻约为 10 秒，取出产物时由熔炉结算经验。

```yaml
bake_practice_brick:
  type: FURNACE
  ingredient: "minecraft:mud"
  experience: 0.5
  cookingTime: 200
  result:
    id: "minecraft:brick"
    count: 1
```

## 选择材料

| 写法                                           | 含义                                 |
| -------------------------------------------- | ---------------------------------- |
| `"minecraft:paper"`                          | 任意以纸为基础材质的物品                       |
| `"sertraline:beginner_sword"`                | 名称、附魔等数据与配方要求匹配的新手之剑               |
| `"itemsadder:ruby"`                          | 由相应物品插件提供的自定义物品                    |
| `"#minecraft:planks"` 或 `"tag:planks"`       | Minecraft 木板标签中的任一材质。带 `#` 的写法要加引号 |
| `{ itemId: "minecraft:paper", exact: true }` | 只接受和普通纸数据一致的物品                     |
| `{ tagId: "#minecraft:planks" }`             | 标签的对象写法                            |

省略物品前缀时，插件先匹配原版材质，再查找 Sertraline 物品。引用自定义材料时，建议使用每次生成结果一致的物品。随机属性、玩家专属名称和物品更新都可能改变匹配所需的数据。

例如，你可以把下面的 `ingredients` 用在一条无序配方里。第一格接受橡木或白桦木板，第二格使用普通纸。

```yaml
ingredients:
  - options:
      - "minecraft:oak_planks"
      - "minecraft:birch_planks"
  - itemId: "minecraft:paper"
    exact: true
```

## 选择设施和产物

| `type`                                    | 设施           | 配置                                              |
| ----------------------------------------- | ------------ | ----------------------------------------------- |
| `CRAFTING`                                | 工作台          | `SHAPED` 使用 `pattern` 和字符映射。 `SHAPELESS` 使用材料列表 |
| `FURNACE`、`BLASTING`、`SMOKING`、`CAMPFIRE` | 熔炉、高炉、烟熏炉、营火 | `ingredient`、`experience`、`cookingTime`         |
| `STONECUTTING`                            | 切石机          | `ingredient`                                    |
| `SMITHING_TRANSFORM`                      | 锻造台升级        | `template`、`base`、`addition`                    |
| `SMITHING_TRIM`                           | 锻造台盔甲纹饰      | `template`、`base`、`addition`。选择支持此类型的服务端与注册方式   |

例如，可用切石机把一个黑曜石切成两块黑石台阶。

```yaml
cut_practice_slabs:
  type: STONECUTTING
  ingredient: "minecraft:obsidian"
  result:
    id: "minecraft:blackstone_slab"
    count: 2
```

产物由 `result.id` 和 `result.count` 指定，数量省略时为 1。`SMITHING_TRIM` 也要求填写有效的 `result.id`，实际产物由模板和盔甲决定。工作台预览会用当前玩家的上下文生成 Sertraline 产物。工作台、烧炼和切石配方可用 `group` 设置分组。工作台配方可设置 `showNotification: false`，在支持该选项的平台上隐藏配方书通知。

## 合成后显示消息

把下面的 `function` 添加到一条工作台配方的 `result` 下，与 `id` 同级。重载后取出产物，聊天栏会显示消息。

```yaml
function:
  - type: kether
    kether: tell "合成成功！"
```

也可以使用 `type: command` 和 `command: "say %player% 完成了合成！"`。命令以控制台身份执行，`%player%` 代表玩家名。脚本类型 `javascript` 或 `js` 使用 `script` 字段，当前版本的配方 JavaScript 不注入 `player`。

## 注册方式和排查

* 省略 `provider` 时使用 `SIMPLE`，按上面的匹配规则处理材料。`COMPLEX` 属于实验性方式，按基础材质或标签匹配，例如以纸为底材的卡牌可被普通纸替代。它拒绝 `exact: true`。
* `input-options` 中的数据过滤和组件过滤是预留项。要按完整数据匹配，请使用 `SIMPLE` 和自定义物品 ID。
* 配方 ID 在目录内保持唯一。建议只用小写字母、数字和下划线。重载后检查控制台的注册结果。如果可手工制作但配方书尚未刷新，请重新登录。

更多完整示例见随附的 `recipes/sample.yml`。开发插件时可查阅[配方 API](../can-kao/recipe-api.md)。
