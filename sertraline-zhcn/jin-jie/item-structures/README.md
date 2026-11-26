# 物品配置段

在 `plugins/Sertraline/workspace/` 下新建物品文件。每个顶层键是一个物品 ID，下面各配置段和 `xbuilder` 同级。可以按用途建子目录，一个文件也可以写多件物品。请使用唯一 ID，发放时保留大小写。以 `__` 开头的 ID 留给[私有物品](../item-manager-system.md)。

## 从完整示例开始

把下面的例子保存为 `workspace/guide_items.yml`，再执行 `/sertraline reload`。在游戏内用 `/sertralineitem give guide_event_pass` 领取通行证。悬停查看活动名称，主手右键查看聊天提示，然后把它放入合成格测试限制。

```yaml
guide_event_pass:
  xbuilder:
    material: PAPER
    name: "<!i><gold>活动通行证"
    lore:
      - "<!i><gray>活动 ${val:event_name}$"
  sertraline:
    vals:
      event_name: "夏日庆典"
    actions:
      onRightClick:
        - async: false
          kether: 'tell "这是夏日庆典通行证。"'
  common:
    disable-crafting: true
  custom_data:
    event_category: summer

guide_blank_card:
  xbuilder:
    material: PAPER
```

只需要普通材质时，像 `guide_blank_card` 那样写一个 `material` 即可。多件物品要复用同一套名称、描述或属性时，用[模板](../templates.md)里的 `$template`、`arguments` 和 `$list-mode`。

## 按用途选择配置段

| 写在物品 ID 下 | 可以做什么 |
| --- | --- |
| [`xbuilder`](section-xbuilder.md) | 指定材质、显示名称、描述、耐久、附魔、原版属性和特殊材质的属性 |
| [`sertraline`](section-sertraline/README.md) | 设置品级、类型、等级、描述、宝石、动作和物品数据 |
| [`minecraft`](section-minecraft-components.md) | 写入或移除服务器物品的原版数据组件 |
| [`visual`](visual-items.md) | 按客户端修改显示，例如纸在客户端显示为钻石 |
| [`custom_data`](section-custom-data.md) | 附加供脚本或其他插件读取的自定义数据 |
| [`common`](#限制物品用途) | 限制在原版操作中使用物品 |
| [`attribute-modifiers`、`settings.attribute_modifiers`、`attribute`](section-attributes.md) | 给使用或装备物品的实体应用 Sertraline 属性 |
| [`mmo`](../mythiclib-mmoitems-compat.md) | 通过 MythicLib 应用 MMO 属性 |
| [`chotenatb`、`chotenatb_complex`](section-attributes.md#chotenattributes) | 按 ChoTenAttributes 的格式保存属性数据 |

## 选择基础物品

通常把 `material: PAPER` 写在 `xbuilder` 下。原版材质也可带命名空间，例如 `"minecraft:paper"`。`minecraft.material` 和 `itemdef.material` 也能提供基础材质，查找顺序是 `xbuilder`、`minecraft`、`itemdef`。`itemdef` 的其他字段由提供该配置段的插件决定。

如果需要从现有物品取得材质，可以写 `"sertraline:guide_blank_card"`，这只使用目标物品的原版材质。要复用完整定义，使用模板。若已安装并启用了外部物品提供者，`xbuilder.material` 也可以填写 `"提供者:物品ID"`，例如已配置 MythicMobs 物品 `MythicBlade` 时写 `"mythicmobs:MythicBlade"`。这时生成会从外部物品开始。

## 文件格式

以下 JSON 和 TOML 都定义一个纸质物品。文件应为 UTF-8。`config.yml` 的 `file-load.item` 默认接受 `.yml`、`.yaml`、`.conf`（YAML）、`.json`、`.toml` 和 `.tml`，并跳过以 `#` 或 `!` 开头的文件名。

```json
{
  "guide_json_card": {
    "xbuilder": {
      "material": "PAPER",
      "name": "<!i><gold>JSON 卡片"
    }
  }
}
```

```toml
[guide_toml_card.xbuilder]
material = "PAPER"
name = "<!i><gold>TOML 卡片"
```

## 限制物品用途

| `common` 字段 | 设置后的效果 |
| --- | --- |
| `disable-crafting` | `true` 阻止工作台和背包合成使用此材料，`vanilla` 仅拦截 `minecraft` 命名空间的配方 |
| `disable-smelting` | 阻止作为熔炼输入，`true` 与 `vanilla` 效果相同 |
| `disable-smithing` | 在支持锻造事件的构建中阻止取出锻造结果，`true` 与 `vanilla` 效果相同 |
| `disable-village-trade` | 在支持交易事件的构建中，阻止选中明确要求此物品的村民交易，`true` 与 `vanilla` 效果相同 |

| `disable-enchanting` | 阻止在附魔台为该物品附魔 |
| `disable-anvil` | 任一输入物品受限时，隐藏铁砧结果并阻止取出 |
| `disable-brewing` | 酿造材料或三个瓶位中的任一物品受限时，取消整次酿造 |
| `disable-stonecutting` | 在有切石机的服务端上阻止取出切石结果，界面仍可能预览结果 |

各项默认关闭，可写 `false`。还可以用带引号的 `"yes"` 或 `"no"` 表达开启或关闭。合成站的材料消耗由[站点配置](../crafting-stations.md)控制。限制通过物品的 Sertraline ID 查找公共定义，所以使用 `sertraline.no-sertraline-id: true` 的物品无法应用这些限制。附魔台、铁砧、酿造台和切石机中，`true` 与 `vanilla` 效果相同。示例中的剑可用附魔台和铁砧测试，地狱疣可配水瓶测试酿造，石头可用于测试切石。旧版插件需要更新到包含这些限制的构建。

```yaml
guide_locked_sword:
  xbuilder:
    material: IRON_SWORD
  common:
    disable-enchanting: true
    disable-anvil: true

guide_locked_wart:
  xbuilder:
    material: NETHER_WART
  common:
    disable-brewing: true

guide_locked_stone:
  xbuilder:
    material: STONE
  common:
    disable-stonecutting: true
```

可在 `config.yml` 的 `messages.common` 中设置 `enchanting-blocked`、`anvil-blocked` 和 `stonecutting-blocked` 的提示文本，设为空字符串则不提示。酿造台在后台处理，不向玩家发送提示。

## 逐项排查

1. 重载后用 `/sertraline manager use public` 和 `/sertraline manager info <物品ID>` 检查定义是否已加载。
2. 重新发一件物品。已发出的旧物品可能保留耐久、附魔或变量，也受服务器的物品自动更新设置影响。
3. 如果领取失败，检查控制台、物品 ID、文件扩展名、缩进和材质。组件、资源包和第三方插件相关字段还要核对各自的版本及依赖。
