---
icon: plug-circle-plus
---

# 合成站（实验性）

合成站是插件提供的独立制作菜单。玩家点击配方后，插件从背包扣除材料，计时结束便自动发放产物。合成站属于实验性功能。

## 用纸制作蓝色代币

先按[物品结构](../ji-chu/item-structure.md)的示例创建 `blue_token`，再将下面的配置保存为 `plugins/Sertraline/crafting-stations/practice.yml`。

```yaml
practice_station:
  display:
    title: "<blue>练习合成站"
    layout:
      - "         "
      - "    I    "
      - "         "
    key: "I"
  recipes:
    press_paper:
      displayName: "蓝色代币"
      time: 3
      inputs:
        - item: "minecraft:paper"
          amount: 2
      outputs:
        - item: "sertraline:blue_token"
          amount: 1
```

1. 执行 `/sertraline reload`，在背包里放至少两张普通纸。
2. 执行 `/sertraline station sertraline:practice_station`。命令里的站点 ID 是配置最外层的 `practice_station`。
3. 点击菜单中央的蓝色代币。背包会扣除两张纸，约三秒后收到一个代币。

## 调整菜单

`display.layout` 每行写 9 个字符，行数决定箱子菜单的大小。用引号保留空格，`display.key` 对应的字符表示放置配方图标的槽位。配方默认用第一件产物作为图标。你也可以给配方加上 `display.item: "minecraft:book"`，让菜单显示一本书。

要添加背景，用 `#` 填充布局中的空白位置，并在 `display` 下、与 `layout` 同级的位置加入以下配置。

```yaml
elements:
  background:
    char: "#"
    material: GRAY_STAINED_GLASS_PANE
    name: "<gray>装饰"
```

配方多于一页时，可添加名为 `next_page` 和 `previous_page` 的元素，把各自的 `char` 放入布局。设置 `roll: true` 可在首尾页之间循环。元素名称和描述可使用 `{page}`、`{total_pages}`、`{next_page}` 和 `{previous_page}`。

## 调整配方

* 在 `recipes` 下为每条配方使用一个独立 ID。`time` 以秒为单位，也可写成 `"{{ random 1 to 10 }}"`。
* `inputs` 与 `outputs` 都是列表。每项用 `item` 写物品标识，例如 `minecraft:paper`、`sertraline:blue_token` 或 `itemsadder:ruby`。省略前缀时按原版材质读取。数量用 `amount`，省略时默认 1。自定义材料按完整物品数据匹配，建议先用生成结果固定的物品测试。
* 每个玩家同时只能进行一项制作。制作期间再次打开合成站，通常会取消当前制作并返还材料。给配方设置 `options.cancellable: false` 可禁止通过重新打开菜单来取消。

例如，你可以给 `press_paper` 添加权限限制。将下面这段加在该配方下，与 `inputs` 同级，再用没有该权限的账号点击测试。

```yaml
conditions:
  - name: "蓝色代币制作权限"
    type: permission
    condition: "example.craft.blue_token"
messages:
  ConditionNotMetPermission: "<red>你还没有{condition_name}。</red>"
```

`type: kether` 或 `boolean` 会检查 `condition` 脚本的结果。`type: Scaled` 则比较 `amount` 与 `required` 的数值，达到或超过要求时通过。`messages` 可按类型覆盖提示，例如 `ConditionNotMetScaled`。可用的提示变量有 `{condition_name}`、`{condition_amount}`、`{condition_required}` 和 `{condition_type}`。

## 制作完成时显示消息

把下面的配置加到 `press_paper`，与 `time` 同级。重载后再制作一次，领取代币时会收到消息。

```yaml
agents:
  onClaim: |-
    tell "蓝色代币制作完成！"
```

`onStart` 在开始计时时执行，`onClaim` 在发出产物后执行。条件也可在自己的 `agents` 里配置 `onSuccess` 和 `onFail`。你可以在站点或单条配方下添加制作脚本，两处都写时先执行配方级脚本。无前缀的脚本使用 Kether。使用 JavaScript 时，在脚本第一行加 `gjs:`（GraalJS）或 `js:`（JSR 223），例如 `gjs: player.sendMessage("制作完成！")`。

## 排查问题

* 找不到站点时，先核对顶层 ID，再用 `/sertraline debug getStations` 查看已加载的站点，并检查重载日志。
* 菜单没显示配方时，检查 `display.key` 是否出现在布局中，以及 `elements` 是否占用了同一槽位。
* 合成无法开始时，检查玩家权限、材料数量和材料数据。使用自定义材料时，先用配置里的同一物品 ID 重新领取材料。
* 离线期间完成计时，重新登录后会尝试补发产物。若登录时尚未完成，则会尝试退回材料。重载会取消在线玩家正在进行的合成并退还材料。

随附的 `crafting-stations/craftingStation.yml` 展示了更多布局和物品引用。`displayTime`、`plural` 和 `option.enable-tracker` 会被读取，但当前版本的菜单与制作流程未使用它们。
