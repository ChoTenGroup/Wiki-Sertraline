---
icon: square-minus
---

# Script Issues

### Cannot use return in NashornJS

Nashorn evaluates inline JavaScript at the top level, where `return` must be inside a function. Other script contexts can wrap the snippet in a function body.

To solve that, just change:&#x20;

```
return 111
```

to:

```
sampleFunction(111)
function sampleFunction(result) {
    return result
}
```



### GraalJS Errors

When running GraalJS scripts on a higher version of GraalVM, the following error may occur.

```
LibGraal.initializeLibgraal(LibGraal.java:204)
→ java.lang.reflect.InvocationTargetException
This is a compatibility issue caused by the JVMCI interface change in JDK 25.
```

Add the following to the JVM startup parameters:\
`-XX:-EnableJVMCI`\
This will disable the JVMCI compiler backend, and GraalJS will revert back to pure interpreted execution mode (Truffle's BFS/partial evaluation still works, just without libgraal's AOT compilation acceleration), without affecting functionality.

If you have `-XX:+EagerJVMCI` you need to remove it
