---
icon: wave-pulse
---

# 物品动作

`actions` 下每个触发器都填一个列表。把示例保存到 `workspace/guide_signal.yml`，重载并用 `/sertralineitem give guide_signal` 领取，然后分别用主手和副手右键。条件为真的主手动作会发送消息，潜行时还会显示第二条。

```yaml
guide_signal:
  xbuilder:
    material: COMPASS
  sertraline:
    actions:
      onRightClick:
        - async: false
          condition:
            - "check 1 == 1"
          kether: 'tell "Signal received."'
        - async: false
          graaljs: |
            if (player.isSneaking()) {
              player.sendMessage("Sneaking signal.");
            }
      "onRightClick@off":
        - async: false
          kether:
            - 'tell "Off-hand signal."'
```

## 内置触发器

| 触发器 | 什么时候执行，读取哪件物品 |
| --- | --- |
| `onClick`、`onRightClick`、`onLeftClick` | 与空气或方块交互。用 `@off` 指定副手，主手使用无后缀版本 |
| `onAttack` | 近战读取主手武器，投射物命中可读取发射时记录的武器 |
| `onShoot` | 发射弓类武器时读取发射武器 |
| `onShootTrident` | 三叉戟触发激流时读取三叉戟，需要支持该事件的构建 |
| `onMine` | 破坏方块时检查双手，支持 `@main` 与 `@off` |
| `onConsume` | 食用、饮用等消耗动作，读取被消耗的物品 |
| `onBreak` | 物品耐久耗尽，读取损坏的物品 |
| `onDrop` | 丢弃物品，读取丢出的物品 |
| `onPickUp` | 拾取掉落物，读取地面物品，需要支持该事件的构建 |
| `onInventoryClick` | 点击容器或背包槽位，读取被点击槽内的物品 |
| `onSwap` | 交换双手，`@main` 或 `@off` 指的是物品即将移动到的槽位 |
| `onSneak` | 开始或结束潜行时检查双手，支持 `@main` 与 `@off` |
| `onLogin` | 登录时检查装备。可用 `@main`、`@off`、`@helmet`、`@chestplate`、`@leggings` 和 `@boots` |

支持后缀的事件可以同时满足基础触发器和某个后缀触发器。无后缀的双手或多槽位动作也可能对多件物品执行。需要监听其他 Bukkit 事件时，见[自定义事件与全局监听](../../event-listeners.md)。

## 条件、脚本和执行方式

| 写在每个列表项里 | 用法 |
| --- | --- |
| `condition`、`require`、`requirement`、`requirements` | Kether 条件，可写单个或列表。列表全部通过才执行该项。省略时直接执行 |
| `async` | 默认为 `true`。读取或修改背包、世界、事件伤害或取消事件时设为 `false` |
| `kether` / `ke` | 一段 Kether，或一个脚本行列表 |
| `javascript` / `js` | JavaScript 文本。可使用 `\|` 写多行 |
| `jexl` / `je` | JEXL 文本 |
| `fluxon` / `fl` | Fluxon 文本 |
| `graaljs` / `gjs` | GraalJS 文本 |
| `mythic` / `mm` | 安装 MythicMobs 后填技能机制字符串，可以写列表，也可在技能后用 `~触发器` 指定 MythicMobs 触发器 |

同一个列表项可以写多种脚本。条件通过后按 Kether、JavaScript、JEXL、Fluxon、GraalJS、MythicMobs 的顺序执行。以下片段写在 `sertraline.actions` 下面，替换同名触发器后可试验几种语法。

```yaml
onRightClick:
  - async: false
    kether: 'tell "Kether"'
    js: 'player.sendMessage("JavaScript");'
    jexl: 'player.sendMessage("JEXL");'
    fluxon: |
      import 'sertraline:bukkit'
      &player::sendMessage("Fluxon")
    graaljs: 'player.sendMessage("GraalJS");'
```

脚本可用 `player`、`event`、`bItem`（实际物品）、`sItem`（物品定义）和物品数据里的键。可取消的事件还能用 `cancellableEvent`，例如在 `onDrop` 中设置同步并用 `js: 'cancellableEvent.setCancelled(true);'` 阻止丢弃。

`config.yml` 的 `action.throttle-time` 设定默认触发间隔，单位毫秒。可在 `action.throttle` 下按基础名覆盖，例如 `onRightClick: 1000`，设为 `0` 关闭该项节流。更多实例见 `workspace/default/default_weapon.yml` 和 `default_consumable.yml`，写入变量的方法见[物品数据](item-data.md)。
