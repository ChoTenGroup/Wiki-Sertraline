---
icon: cubes
---

# 物品结构

一个配置文件可以包含多个物品。每个顶层名称是一个物品 ID，其下的配置段分别描述物品的不同部分：

```yaml
red_token:
  xbuilder:
    material: PAPER
    name: "<!i><red>红色代币"

blue_token:
  xbuilder:
    material: PAPER
    name: "<!i><blue>蓝色代币"
```

保存到 `plugins/Sertraline/workspace/` 后执行 `/sertraline reload`，再用 `/sertralineitem give red_token` 领取红色代币。

常见配置段：

* [xbuilder](../jin-jie/item-structures/section-xbuilder.md)：材质、名称、描述及常用物品属性。
* [sertraline](../jin-jie/item-structures/section-sertraline/)：品级、物品数据和触发动作。
* [minecraft](../jin-jie/item-structures/section-minecraft-components.md)：受服务端版本支持的数据组件。
* [custom\_data](../jin-jie/item-structures/section-custom-data.md)：自定义数据。

{% hint style="success" %}
所有配置段都是可选的；简单物品可从 `xbuilder` 开始，只添加实际需要的部分。`minecraft` 段取决于服务端版本；`visual` 覆盖需要 PacketEvents 2.10 或更高版本，Legacy 1.12.2 构建不提供这些功能。
{% endhint %}

配置中的 ID 应保持唯一；使用字母、数字和下划线有助于避免命令输入问题。

## 其他配置段与 ID

* [`mmo`](../jin-jie/mythiclib-mmoitems-compat.md) 用于配置由 MythicLib 提供的属性，需要启用对应模块。
* [`visual`](../jin-jie/item-structures/visual-items.md) 用于定制客户端看见的材质、名称或描述。
* 物品 ID 区分大小写，例如 `sword_of_skyline` 与 `Sword_Of_Skyline` 是不同的写法。以 `__` 开头的 ID 保留给私人物品管理器，工作区加载器会跳过这类定义。发放物品、创建配方和调用脚本时使用与配置一致的 ID。默认构建出的物品会用 `sertraline_id` 保存其 ID。

## 开发者参考

加载时，处理器读取配置段并把处理后的数据保存到物品定义。生成时，适配器使用这些数据设置实际物品的属性。例如，`xbuilder` 处理器读取木剑的材质和名称，生成时再解析占位符并写入显示名称。脚本侧可参阅 [SertralineAPI](../can-kao/sertralineapi.md)。
