---
icon: square-minus
---

# 脚本常见问题

## Nashorn 顶层 return

Nashorn 的顶层脚本不能直接使用 `return`，应将它放进函数体。例如：

```javascript
sampleFunction(111)
function sampleFunction(result) {
    return result
}
```

不同引擎对顶层返回值的处理可能不同，切换引擎时请检查脚本的执行结果。

## GraalJS 和 JVMCI 错误

在某些高版本 GraalVM / JDK 25 环境中可能出现 `LibGraal.initializeLibgraal` 和 `InvocationTargetException`。遇到该问题时，可检查启动参数：移除 `-XX:+EagerJVMCI`，并尝试添加 `-XX:-EnableJVMCI` 后重启。具体效果取决于使用的 JVM 和版本，修改前请保存原启动参数。

### 识别 JVMCI 报错

如果控制台出现类似下面的内容，可用上一节的启动参数排查。禁用 JVMCI 后，GraalJS 可能改用解释执行，脚本仍可运行，执行性能可能下降。

```text
LibGraal.initializeLibgraal(LibGraal.java:204)
java.lang.reflect.InvocationTargetException
```
