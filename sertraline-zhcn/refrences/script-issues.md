---
icon: square-minus
---

# Script Issues

### Cannot use return in NashornJS

This is Nashorn's known limitation. In Nashorn, the top-level scope cannot use return - return can only appear inside the function body. Both GraalJS and standard JavaScript engines allow top-level returns during eval().\
Nashorn's eval() top-level does not allow returns, while GraalJS allows them.

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
