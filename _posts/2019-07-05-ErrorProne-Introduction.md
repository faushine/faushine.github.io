---
layout: post
title: "Error Prone VS SpotBugs"
date:   2019-07-05
author: Faushine
tags: 
- ErrorProne
- SpotBugs
- Java
---

# Error Prone

## 简介

[Error Prone](https://errorprone.info/) 是一款开源的JAVA代码检查工具，它将常见编程错误作为运行时错误报告，并给出相应的修改建议。

## Installation in Maven

```xml
 <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
        <configuration>
          <source>8</source>
          <target>8</target>
          <!-- 编译参数，所有新增的参数应该用空格隔开，写入-Xplugin:ErrorProne之后-->
          <compilerArgs>
            <arg>-XDcompilePolicy=simple</arg>
            <arg>-Xplugin:ErrorProne</arg>
          </compilerArgs>
          <annotationProcessorPaths>
            <path>
              <groupId>com.google.errorprone</groupId>
              <artifactId>error_prone_core</artifactId>
              <version>2.3.3</version>
            </path>
            <!-- 2.2.0之后的版本开始支持lombok，lombok的依赖需要添加到这里-->
          </annotationProcessorPaths>
        </configuration>
      </plugin>
    </plugins>
  </build>
```

在JDK 8下需要添加：

```xml
 <properties>
    <javac.version>9+181-r4173-1</javac.version>
  </properties>

  <!-- using github.com/google/error-prone-javac is required when running on JDK 8 -->
  <profiles>
    <profile>
      <id>jdk8</id>
      <activation>
        <jdk>1.8</jdk>
      </activation>
      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
              <fork>true</fork>
              <compilerArgs combine.children="append">
                <arg>-J-Xbootclasspath/p:${settings.localRepository}/com/google/errorprone/javac/${javac.version}/javac-${javac.version}.jar</arg>
              </compilerArgs>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
```

```bash
mvn compile
```

## Severity

通过设置编译参数，用户可以决定某个check是否生效，或者重写`sevsrity等级`(Warning vs Error)：

```
-Xep:<checkName>[:severity]
```
`checkName`是[Bug Pattern](https://errorprone.info/bugpatterns)中列举的一系列名称，`severity`即 Warning 或者 Error
比如：

```
-Xep:ReferenceEquality  [打开 ReferenceEquality check]
-Xep:ReferenceEquality:OFF  [关闭 ReferenceEquality check]
-Xep:ReferenceEquality:WARN  [打开 ReferenceEquality check，检查内容以warning显示]
-Xep:ReferenceEquality:ERROR  [打开 ReferenceEquality，检查内容以error显示]
-Xep:ReferenceEquality:OFF -Xep:ReferenceEquality  [打开 ReferenceEquality check]
```

此外你还可以定义某个目录下的文件不希望被检查：

```
-XepExcludedPaths:.*/build/generated/.*
```

由于ErrorProne在项目编译过程中检查程序，一旦发现Error就会停止编译，此时如果想持续编译需要加入以下编译参数，它会把Error当成Warning输出，这样就不会中断编译：

```
-XepAllErrorsAsWarnings
```

## Patching 




# SpotBugs

## 简介

[SpotBugs](https://spotbugs.github.io/) 继承自`FindBugs`，它通过对字节码进行静态分析，查找相关的漏洞。其中包括90余种Bad practice，155余种Correctness，9种Experimental， 2种 Internationalization，17种Malicious code vulnerability，46种Multithreaded correctness,4种 Bogus random noise，37种Performance，11种 Security,87种Dodgy。

## Installation in Maven

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <reporting>
    <plugins>
      <plugin>
        <groupId>com.github.spotbugs</groupId>
        <artifactId>spotbugs-maven-plugin</artifactId>
        <version>3.1.13-SNAPSHOT</version>
        <configuration>
          <!-- Optional directory to put spotbugs xml report -->
        </configuration>
      </plugin>
    </plugins>
  </reporting>
  ...
</project>
```

执行以下命令运行spotbugs ui

```bash
mvn spotbugs:gui
```