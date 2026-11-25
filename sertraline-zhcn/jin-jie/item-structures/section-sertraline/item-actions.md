---
icon: wave-pulse
---

# 物品动作

你可以让物品在右键、攻击、食用等操作发生时执行脚本。先在 `plugins/Sertraline/workspace/hello_compass.yml` 中创建一把指南针。

```yaml
hello_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><gold>问候指南针"
  sertraline:
    actions:
      onRightClick:
        - async: false
          kether: tell "你使用了问候指南针！"
```

保存后执行 `/sertraline reload`，再用 `/sertralineitem give hello_compass` 领取。把指南针放在主手并朝空气右键，聊天栏会显示消息。

## 选择触发器

| 触发器                                    | 触发时机                                 |
| -------------------------------------- | ------------------------------------ |
| `onClick`、`onRightClick`、`onLeftClick` | 点击交互。可以加 `@off` 指定副手的交互              |
| `onAttack`                             | 近战命中或可追溯到发射武器的投射物命中。近战读取主手武器         |
| `onShoot`、`onShootTrident`             | 发射弓类武器，或使用三叉戟触发激流                    |
| `onMine`、`onConsume`、`onBreak`         | 破坏方块、食用或饮用、物品耐久耗尽                    |
| `onDrop`、`onPickUp`、`onInventoryClick` | 丢弃、拾取或点击背包槽位中的物品。拾取触发器需要支持相应事件的现代服务端 |
| `onSwap`、`onSneak`                     | 交换主副手，或开始与结束潜行                       |
| `onLogin`                              | 登录时检查双手和四个盔甲槽                        |

`onMine`、`onSwap`、`onSneak` 和 `onLogin` 支持 `@main` 与 `@off`。在 `onSwap` 中，后缀表示**交换后的目标槽位**，例如 `onSwap@off` 读取即将移到副手的物品。登录还可用 `onLogin@helmet`、`onLogin@chestplate`、`onLogin@leggings` 和 `onLogin@boots`。基础触发器与对应的后缀触发器都配置时，两条动作都可能执行。要使用其他 Bukkit 事件，请参阅[自定义事件与全局监听](../../event-listeners.md)。

## 添加条件和脚本

一个触发器下面可以有多条动作，每条用 `-` 开头。把 `condition` 与脚本写在同一个列表项里，用 Kether 表达式决定是否执行该项。例如，把指南针原有的 `onRightClick` 条目替换为下面的写法，再手持指南针，分别测试普通右键和潜行右键。

```yaml
onRightClick:
  - async: false
    graaljs: |
      if (player.isSneaking()) {
        player.sendMessage("你在潜行时使用了指南针！");
      }
```

示例在 GraalJS 里判断玩家是否潜行。要改用其他脚本语言，可用 `kether`、`js`（也可写 `javascript`）、`jexl` 或 `fluxon`。JavaScript 可以读取 `player`（触发动作的玩家）、`event`（当前事件）、`bItem`（触发动作的物品）和[物品数据](item-data.md)。

动作默认异步执行。要访问背包或世界、修改伤害、取消事件，请设置 `async: false`。在可取消的事件中，可调用 `event.setCancelled(true)`。用 `config.yml` 的 `action.throttle-time` 修改默认触发间隔（毫秒），或在 `action.throttle` 下用触发器名称覆盖，例如 `onRightClick: 1000`。设为 `0` 可关闭该项节流。

可以在随附的 `workspace/default/default_weapon.yml` 试用 `sword_of_ultimate_damage`，或在 `default_consumable.yml` 试用食用后获得速度效果的 `speed_potion_cookie`。脚本可用的工具见[脚本工具](../../../can-kao/script-util/)。
