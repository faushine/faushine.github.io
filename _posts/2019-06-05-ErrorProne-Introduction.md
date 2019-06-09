---
layout: post
title: "Error Prone入门"
date:   2019-06-05
author: Faushine
tags: 
- ErrorProne
- Java
---
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

To be continued...