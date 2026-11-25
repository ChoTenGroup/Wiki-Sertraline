# 自定义事件与全局监听

你可以用 `custom-events.yml` 让装备响应额外事件，也可以用 `global-listeners/` 给整台服务器添加事件规则。

## 让指南针响应等级变化

在 `plugins/Sertraline/custom-events.yml` 顶层添加下面的触发器。已有其他事件时保留原条目，并给新条目使用唯一名称。

```yaml
onPracticeLevelChange:
  class: PlayerLevelChangeEvent
  item-source:
    - main-hand
```

新建 `workspace/practice_compass.yml`。动作名称必须与上面的 `onPracticeLevelChange` 一致。

```yaml
practice_compass:
  xbuilder:
    material: COMPASS
    name: "<!i><aqua>练习指南针"
  sertraline:
    actions:
      onPracticeLevelChange:
        - async: false
          graaljs: |
            player.sendMessage("你的新等级是 " + event.getNewLevel());
```

保存并重载后，用 `/sertralineitem give practice_compass` 领取并放进主手。让经验等级跨到下一级，或在现代版测试服上执行 `/experience add @s 1 levels`。聊天栏会显示变化后的等级。换下指南针后再改变等级，物品动作便不会触发。

`class` 指定事件，`item-source` 指定从哪个位置查找触发动作的物品。你也可以填 `off-hand`、`both-hands`、`armor` 或 `inventory`。更复杂的物品来源可参考自带 `custom-events.yml` 中的 `custom-source` 示例。内置动作名已由插件占用，给自定义触发器起一个新名字。

## 给所有进服玩家发一条消息

将下面的配置保存到 `global-listeners/practice_welcome.yml`。示例使用现代构建中的 GraalJS。

```yaml
practice_welcome:
  bind: PlayerJoinEvent
  async: false
  eval:
    graaljs: |
      player.sendMessage("欢迎来到服务器！");
```

执行 `/sertraline reload`，重新登录即可看到消息。全局监听器绑定指定事件，玩家无需领取物品。默认忽略已取消的事件。涉及 Bukkit 对象或修改事件的脚本使用 `async: false`。
