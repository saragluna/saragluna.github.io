---
layout:     post
title:      "Maven Dependency Management"
subtitle:   "Understand Maven Dependency Management"
date:       "2019-03-15 12:00:00"
author:     "Sarag"
header-img: "img/post-bg-2019.jpg"
tags:
    - Java
---



I got this problem of incorrect maven dependency jar version while developing and this leads me to have an deeper understanding of maven dependency management.



## The Clueless Version

The case is that I want to include google's gson of version `2.8.2` in project A, and project A depends on project B. In project B's pom.xml the `2.8.2` version of gson has been added explicitly. So I take it for granted that in project A I will have the gson dependency of version `2.8.2`. Contrary to that I get a version of `2.8.0` out of the blue. 

I use command like `mvn dependency:tree -X` to try to find out  where does this version come from, and got something like this:

```java
[DEBUG]    com.ctrip.framework.apollo:apollo-client:jar:1.0.0:compile
[DEBUG]       com.ctrip.framework.apollo:apollo-core:jar:1.0.0:compile
[DEBUG]          com.google.code.gson:gson:jar:2.8.0:compile
```

Maybe I have found the problem, so I add `<exclusion></exclusion>` in the apollo-client dependency. Considering that I have resolved the problem, but run `mvn dependency:tree` again I get this:

```java
[INFO]   +- com.squareup.retrofit2:retrofit:jar:2.2.0:compile
[INFO]   +- com.squareup.retrofit2:converter-gson:jar:2.2.0:compile
[INFO]   |  \- com.google.code.gson:gson:jar:2.8.0:compile
```

It's stated in the converter-gson.jar's pom.xml that the depended gson version is `2.7`, how come the result is `2.8.0`?

```xml
<!-- Converter Dependencies -->
<gson.version>2.7</gson.version>
```

I find this dependency management in my project, and the version is brought by import of dependency management. And dependency management will control transitive dependency.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- this import includes the 2.8.0 version-->
<gson.version>2.8.0</gson.version>
```



## Dependency Management

>**Dependency management** - this allows project authors to directly specify the versions of artifacts to be used when they are encountered in transitive dependencies or in dependencies where no version has been specified. In the example in the preceding section a dependency was directly added to A even though it is not directly used by A. Instead, A can include D as a dependency in its dependencyManagement section and directly control which version of D is used when, or if, it is ever referenced.

## Dependency Scope

- **import**
  This scope is only supported on a dependency of type `pom` in the `<dependencyManagement>` section. It indicates the dependency to be replaced with the effective list of dependencies in the specified POM's `<dependencyManagement>` section. Since they are replaced, dependencies with a scope of `import` do not actually participate in limiting the transitivity of a dependency.