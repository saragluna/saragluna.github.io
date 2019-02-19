---
layout: post
title:  "SpringEL ClassLoader"
subtitle: "SpringEL Fails the Third Time"
date:   2019-02-18 15:46:32
author: "Sarag"
header-img: "img/post-bg-2019.jpg"
tags: 
  - Spring
  - Java
---
### The First Look

Our project use springel to evaluate expressions and one strange error happens. It seems like that the first two evaluations are always successful, however when it comes to the third time the evaluation begins to fail. After tracing down the source code we find the reason.

```java
/**
 * Compile the expression if it has been evaluated more than the threshold number
 * of times to trigger compilation.
 * @param expressionState the expression state used to determine compilation mode
 */
private void checkCompile(ExpressionState expressionState) {
	this.interpretedCount++;
	SpelCompilerMode compilerMode = expressionState.getConfiguration().getCompilerMode();
	if (compilerMode != SpelCompilerMode.OFF) {
		if (compilerMode == SpelCompilerMode.IMMEDIATE) {
			if (this.interpretedCount > 1) {
				compileExpression();
			}
		}
		else {
			// compilerMode = SpelCompilerMode.MIXED
			if (this.interpretedCount > INTERPRETED_COUNT_THRESHOLD) {
				compileExpression();
			}
		}
	}
}
```

Each to `SpringExpress.getValue()` will call this check compile function and when `interpretedCount `exceeds 1 the compileExpression() is called which explains why evaluation succeeds first two times while fails the third.

### The Real Cause


The exception looks like this:

```java
Caused by: java.lang.ClassCastException: com.A.bean.User cannot be cast to com.A.bean.User
	at spel.Ex3.getValue(Unknown Source) ~[na:na]
	at org.springframework.expression.spel.standard.SpelExpression.getValue(SpelExpression.java:254) ~[spring-expression-5.0.8.RELEASE.jar:5.0.8.RELEASE]
```

The exception seems strange `com.A.bean.User cannot be cast to com.A.bean.User`, an object of a class can't be cast to the class of exactly the same name. So we suspect the root cause is about classLoader.

After some debugging we find that there are two `com.A.bean.User` class

```java
// ASM BYTECODE FOR "com.A.bean.User" @ClassLoader:org.springframework.boot.devtools.restart.classloader.RestartClassLoader@1aeb256e

// ASM BYTECODE FOR "com.A.bean.User" @ClassLoader:sun.misc.Launcher$AppClassLoader@18b4aac2
```

And the `RestartClassLoader` is because of our usage of `spring-boot-devtools` which is used for hot deploy.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

### References

[https://github.com/AxonFramework/AxonFramework/issues/344](https://github.com/AxonFramework/AxonFramework/issues/344)

[https://stackoverflow.com/questions/31282985/classcastexception-on-custom-class-loading](https://stackoverflow.com/questions/31282985/classcastexception-on-custom-class-loading)



