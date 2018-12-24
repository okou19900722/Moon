---
layout: post
title:  "如何使用vertx-codegen工作"
date:   2018-12-24
excerpt: "`vertx-codegen` 广泛应用于vert.x 3的项目中，比如 `Polyglot` 的实现或者 `RxJava` 的支持。未来 **可能**（个人猜测，也可能不会） 在支持 `CompletionStage` 的项目中使用。那么在工作中，我们在哪些地方需要使用到 `vertx-codegen` 呢？"
project: vert.x 3
tag: 
- vert.x 3
- java
- codegen
comments: true
---

`vertx-codegen` 广泛应用于vert.x 3的项目中，比如 `Polyglot` 的实现或者 `RxJava` 的支持。未来 **可能**（个人猜测，也可能不会） 在支持 `CompletionStage` 的项目中使用。那么在工作中，我们在哪些地方需要使用到 `vertx-codegen` 呢？

# service proxy的使用

最早接触到 `vertx-codegen` 应该是使用 `vertx-service-proxy` 这个模块了。我们先来看看在 `vertx-service-proxy` 中，我们是如何配置的。

## java 代码

首先，你需要定义你的 `service` 接口 `SomeDatabaseService.java` ，并添加 `@ProxyGen` 注解：

```java
@ProxyGen
public interface FooService {
  void foo(String foo, Handler<AsyncResult<Void>> result);
}
```

然后你需要在包路径上定义一个 `package-info.java` 文件

```java
@ModuleGen(name = "examples", groupPackage = "examples")
package examples;
```

## build配置

无论你是maven或者gradle项目，那么你还需要添加依赖，并配置 `AnnotationProcessor` 处理程序:

如果使用 `maven` ，你需要添加以下依赖

```xml
<dependencies>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-core</artifactId>
        <version>3.6.2</version>
    </dependency>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-service-proxy</artifactId>
        <version>3.6.2</version>
    </dependency>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-codegen</artifactId>
        <version>3.6.2</version>
    </dependency>
</dependencies>
```

使用 `gradle`，你需要添加以下依赖
```kotlin
dependencies {
    compile("io.vertx:vertx-core:3.6.2")
    compile("io.vertx:vertx-codegen:3.6.2")
    compile("io.vertx:vertx-service-proxy:3.6.2")
}
```
### maven 配置

使用maven，通常有两种方式来配置AnnotationProcessor

一种是直接配置编译器
([**示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-maven-compiler-args))，但这种配置需要给依赖加上classifier为processor的配置

```xml
<dependencies>
    <dependency>
        <groupId>io.vertx</groupId>
        <artifactId>vertx-codegen</artifactId>
        <version>3.6.2</version>
        <classifier>processor</classifier>
    </dependency>
</dependencies>
```

插件配置(默认好像是不用配置的，但为了说明一下，避免其他环境配置导致AnnotationProcessor不能正常工作。)

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <executions>
        <execution>
            <id>default-compile</id>
            <configuration>
                <compilerArgs>
                    <arg>-proc:only</arg>
                </compilerArgs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

别一种是通过processor插件来配置
([**示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-maven-processor-plugin))

```xml
<plugin>
    <groupId>org.bsc.maven</groupId>
    <artifactId>maven-processor-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
      <systemProperties>
        <java.util.logging.SimpleFormatter.format>%4$s: %3$s - %5$s %6$s%n</java.util.logging.SimpleFormatter.format>
      </systemProperties>
    </configuration>
    <executions>
      <execution>
        <id>generate-sources</id>
        <goals>
          <goal>process</goal>
        </goals>
        <phase>process-sources</phase>
        <configuration>
          <processors>
            <processor>io.vertx.codegen.CodeGenProcessor</processor>
          </processors>
          <optionMap>
            <codegen.output>src/main/</codegen.output>
          </optionMap>
        </configuration>
      </execution>
    </executions>
</plugin>
```

### gradle 配置(本示例使用kotlin dsl)

gradle与maven一样，也有两种配置，一种是配置编译器，一种是自定义task。

直接配置编译器也需要classifier为processor的配置，这里与maven一样

```kotlin
dependencies {
    annotationProcessor("io.vertx:vertx-codegen:3.6.2:processor")
}
```
>注意：gradle 5的annotationProcessor不再使用compile配置的classpath，因此需要使用annotationProcessor来配置，而gradle 4则可以通过compile配置

第一种配置编译器的方案：
([**gradle4示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-gradle4-compiler-args))
([**gradle5示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-gradle5-compiler-args))

```kotlin
tasks.getByName("compileJava") {
    this as JavaCompile
    options.compilerArgs = listOf(
            "-Acodegen.output=src/main"
    )
}
```

>注意，这种配置有个问题就是生成的java代码在build/classes/java/main里，打包的时候会包含进去，需要再配置Jar来把*.java给排除

第二种通过自定义task配置的方案：
([**gradle4示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-gradle4-custom-task))
([**gradle5示例**](https://github.com/okou19900722/blog-source/tree/master/working-with-vertx-codegen/service-proxy-gradle5-custom-task))
```kotlin
task<JavaCompile>("annotationProcessing") {
    source = sourceSets["main"].java
    classpath = configurations.compile.get() + configurations.compileOnly.get()
    destinationDir = project.file("build/generated")
    //如果是gradle 4则不需要设置annotationProcessorPath，所有相关的依赖配置只需要使用compile即可
    options.annotationProcessorPath = classpath
    options.compilerArgs = listOf(
            "-proc:only",
            "-processor", "io.vertx.codegen.CodeGenProcessor",
            "-Acodegen.output=src/main"
    )
}
```
>注意，如果使用这种方案，最好将compileJava依赖自定义的task

```kotlin
tasks.getByName("compileJava").dependsOn("annotationProcessing")
```

这里推荐第二种：自定义task的配置。原因如下，如果有错误可以指出

1. 第二种粒度更小，在只需要执行AnnotationProcessor的时候可以只执行它
2. 第一种生成的代码在 `build/classes/java/main` 目录下，而这个目录同时也是编译之后的 `.class` 文件，这导致源码与编译代码混在了一起，打包还需要额外配置将他排除在外
3. 假设第二条通过配置 `codegen.output.service_proxy=generated` 来配置 `service_proxy` 的输出目录，哪怕将generated目录添加到sourceSet，也无法编译。



