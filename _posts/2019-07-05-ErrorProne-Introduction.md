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

[Error Prone](https://errorprone.info/) 是一款开源的JAVA代码检查工具，它可以在项目编译过程中匹配可能的error pattern并给出相应的修改建议。

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

# SpotBugs

## 简介

[SpotBugs](https://spotbugs.github.io/) 继承自findbugs，它可以对代码进行静态分析，查找相关的漏洞。其中包括90余种Bad practice，155余种Correctness，9种Experimental， 2种 Internationalization，17种Malicious code vulnerability，46种Multithreaded correctness,4种 Bogus random noise，37种Performance，11种 Security,87种Dodgy。

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