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

[Error Prone](https://errorprone.info/) 是一款由Google开发的开源JAVA代码检查工具，它通过加强编译器的类型检查来匹配bug。起初Goole是想把FindBugs整合到他们的代码库里面，后来实验之后发现FindBugs是编译后生成检查报告，它不会中断编译过程，这就导致很多程序员根本不去修复检查出的bug。所以他们根据ClangMR的经验，也就是把检查过程放到编译中，按照FindBugs的风格写了EP。EP所涵盖的bug不像FindBugs那么多，毕竟它是编译时检查，要是随便碰到个warning就终止编译也太烦了。据官方说上线的每个bug都要vetting in Google's codebase，所以像useless code这种bug就不会报告了。至于性能方面，我在IDEA上对比过 java compiler 和 ep compiler 的编译时间，采用EP来编译代码会多花一倍的时间，其实还能接受。

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
          <!-- 在这里可以添加自定义checker或者lombok作为dependency -->
          <annotationProcessorPaths>
            <path>
              <groupId>com.google.errorprone</groupId>
              <artifactId>error_prone_core</artifactId>
              <version>2.3.3</version>
            </path>
            <path>
              <groupId>com.example</groupId>
              <artifactId>sample-plugin</artifactId>
              <version>1.0-SNAPSHOT</version>
            </path>
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

## patching the code

通过配置编译器参数可以让ep在检查中生成patch文件，这个文件是用来自动的按照fix suggesttion去修改源文件。比如

```java
public class EqualsHashCodeTestPositiveCases {
public static class EqualsAndHashCode { public boolean equals(Object o) {
return false; }
public int hashCode () { return 42;
} }
public static class HashCodeOnly { public int hashCode () {
return 42; }
}
public static class Neither {} }
```

编译之后EP提示@override忘了，那么如果你在pom文件里面写了这个

```xml
  <compilerArgs>
            <arg>-XDcompilePolicy=simple</arg>
            <arg>-Xplugin:ErrorProne -XepPatchChecks:MissingOverride -XepPatchLocation:/Users/faushine/Documents/CaseStudy </arg>
          </compilerArgs>
```

那么你就会得到这样一个patch文件

![alt text](/img/in-post/2018-07-30/patchfile.png)

只要用patch去运行这个文件你就能得到一段健康的代码了。

## Write a custom checker

Error-Prone通过`@AutoService(BugChecker.class)`加载新的checker，官方提供了一个[示例](https://github.com/google/error-prone/tree/master/examples/plugin/maven)可以看看，或者你也可以参考[我的repo](https://github.com/faushine/custom-checker)，我把wiki里提到的DoNotReturnNull实现了，以及两个新的pattern，NoSynchronizedMethodCheck 和 NoSynchronizedThisCheck。



# SpotBugs

## 简介

[SpotBugs](https://spotbugs.github.io/) （以下简称SB）继承自`FindBugs`，它通过对字节码进行静态分析，查找相关的漏洞。其中包括90余种Bad practice，155余种Correctness，9种Experimental， 2种 Internationalization，17种Malicious code vulnerability，46种Multithreaded correctness,4种 Bogus random noise，37种Performance，11种 Security,87种Dodgy。

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

   <plugin>
          <groupId>com.github.spotbugs</groupId>
          <artifactId>spotbugs-maven-plugin</artifactId>
          <version>3.1.11</version>
          <dependencies>
            <dependency>
              <groupId>com.github.spotbugs</groupId>
              <artifactId>spotbugs</artifactId>
              <version>4.0.0-beta2</version>
            </dependency>
          </dependencies>
        </plugin>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-site-plugin</artifactId>
          <version>3.7.1</version>
        </plugin>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-project-info-reports-plugin</artifactId>
          <version>3.0.0</version>
   </plugin>
  ...
</project>
```

因为SB是基于字节码分析，所以你得先编译过才行。

执行以下命令生成检查报告：

```bash
mvn site
```

运行SB的图形界面
```
mvn spotbugs:gui
```

或者直接去target里面找xml文件


