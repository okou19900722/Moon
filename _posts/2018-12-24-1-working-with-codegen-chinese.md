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

# vertx-codegen 里的一些概念

上面的例子，我们用到了 `@ProxyGen` 注解，那么这个注解是干什么用的呢？下面我们来说说 `vertx-codegen` 里的一些概念

## 概念

`Model`, `ModelProvider`, `Generator`, `GeneratorLoader`, `TypeInfo`

### Model 

`Model` 类是用来描述某个功能类的结构的，比如 `DataObjectModel` 是根据 `Getter`, `Setter`， `Adder` 方法来生成字段信息 `PropertyInfo` 。

`vertx-codegen` 内置了5种Model(3.6.0 之前是6种)

* ClassModel : 在接口上使用 `@VertxGen` 注解之后，会生成对应的ClassModel对象，ClassModel类包含接口的方法和静态变量信息。通常是用来转换api用的，比如vertx-lang-*模块，和vertx-rx模块
* DataObjectModel : 在接口或者类上使用 `@DataObject` 注解之后，会生成对应的DataObjectModel对象，DataObjectModel 包含数据类的字段信息
* EnumModel : 在枚举上使用 `@VertxGen` 注解之后，会生成对应的EnumModel对象，包含枚举的values信息（只包括枚举的name，而不包括方法等信息）
* ModuleModel : 只有添加了 `package-info.java` 文件，并在该文件的包定义上使用 `@ModuleGen` 注解之后，才会执行 `vertx-codegen` 的AnnotationProcessor任务， 
* PackageModel  : 所有按 `vertx-codegen` 规则生成的Model所在的包都会生成对应的PackageModel

- ProxyModel : ProxyModel继承自ClassModel，在接口上使用 `@ProxyGen` 注解之后，会生成对应的ProxyModel对象。ProxyModel类包含

### ModelProvider

`vertx-codegen` 通过java的 `ServiceLoader` 类加载用户自定义的 `ModelProvider`, 并调用 `getModel` 方法来获取自定义的 `Model`，如果返回null，则忽略。
 
### Generator

用来生成需要的代码的 `生成器` 。3.6.0之前使用mvel模版引擎，但mvel与jdk10有兼容问题，所以在3.6.0中尝试将mvel移除（保留了兼容版本）。

### GeneratorLoader

`vertx-codegen` 通过java的 `ServiceLoader` 类加载用户自定义的 `GeneratorLoader`, 并调用 `loadGenerators` 方法来获取自定义的 `Generator`

### TypeInfo

TypeInfo与Model有点相似，但Model按不同功能而处理不同的数据，而TypeInfo只包含类的字义（包括泛型定义，是否可空等）。且TypeInfo不支持自定义。

## 解剖 vertx-service-proxy

因为 `vertx-codegen` 3.6.0之前的版本对自定义Generator和Model不是很友好，因此我们主要讲3.6.0之后的版本。

首先是 `ModelProvider` ，`vertx-service-proxy` 提供了一个类[**ProxyModelProvider**](https://github.com/vert-x3/vertx-service-proxy/blob/9d2a5641085a81d17adb67663fa4eefed760fef8/src/main/java/io/vertx/serviceproxy/generator/model/ProxyModelProvider.java)
用来提供自定义的Model对象[**ProxyModel**](https://github.com/vert-x3/vertx-service-proxy/blob/9d2a5641085a81d17adb67663fa4eefed760fef8/src/main/java/io/vertx/serviceproxy/generator/model/ProxyModel.java)。
ProxyModelProvider的实现很简单，使用了@ProxyGen注解的类，会生成ProxyModel对象。

接着是 `GeneratorLoader`，`vertx-service-proxy` 提供的类是[**ServiceProxyGenLoader**](https://github.com/vert-x3/vertx-service-proxy/blob/9d2a5641085a81d17adb67663fa4eefed760fef8/src/main/java/io/vertx/serviceproxy/generator/ServiceProxyGenLoader.java) ，
他的 `loadGenerators` 方法返回了两个Generator：[**ServiceProxyHandlerGen**](https://github.com/vert-x3/vertx-service-proxy/blob/9d2a5641085a81d17adb67663fa4eefed760fef8/src/main/java/io/vertx/serviceproxy/generator/ServiceProxyHandlerGen.java) 
和 [**ServiceProxyGen**](https://github.com/vert-x3/vertx-service-proxy/blob/9d2a5641085a81d17adb67663fa4eefed760fef8/src/main/java/io/vertx/serviceproxy/generator/ServiceProxyGen.java)。
ServiceProxyHandlerGen类用来生成 `VertxProxyHandler` 后缀的类，而ServiceProxyGen类用来生成 `VertxEBProxy` 后缀的类。

因为这些类之前都是 `vertx-codegen` 包的，所以 `ApiTypeInfo` 类里保留了proxyGen 值来标记是否收使用 `@ProxyGen` 注解的类

>在 `vertx-codegen` **3.6.0** 版本之前，与 `vertx-codegen` 相关，但为 `vertx-service-proxy` 提供服务的类都是属于 `vertx-codegen` 包的，在3.6.0版本之后，除 `@ProxyGen` 这个类，其他类都移到了 `vertx-service-proxy` 包。其中 `ServiceProxyHandlerGen` 和 `ServiceProxyGen` 类是3.6.0版本移除了mvel之后新加的类 

3.6.0版本可以算是 `vertx-codegen` 的一个里程碑：他更好的支持自定义的功能。随着3.6.0版本孵化的[**项目**](https://github.com/vert-x3/vertx-web/tree/master/vertx-web-api-service)
提供了更好的说明。他使用了自定义的 `@WebApiServiceGen` 注解。

## `vertx-codegen` 实战

目前 vert.x 的代码风格大部分是基于回调的，而基于回调的代码有个最大的问题： `回调地狱` 。

![callback-hell](assets/img/codegen/callback-hell.jpg)

当然，我们也可以通过 `Future` 来解决回调地狱的问题（其他方式还有rx/kotlin coroutine, vertx-sync等）

![Future examples](assets/img/codegen/future.jpg)
(图片来自QQ群消息，如果侵权请联系我)

虽然 vert.x 给我们提供了 `Future` 类来解决回调问题，但官方的api依然还是回调风格（Future的代码根据官方的 
[路线图](https://github.com/vert-x3/wiki/wiki/Vert.x-Roadmap#completionstage-support) 
来说要等4.0）

所以本节我们通过将现有api的回调风格改为返回Future来体验如何使用vertx-codegen来生成我们需要的代码

### 准备

本例子，我们将会有两个模块：`vertx-future-wrapper` 和 `vertx-future-wrapper-gen` 。在 `vertx-future-wrapper-gen` 模块中，我们来实现 `vertx-codegen` 提供的接口，
然后在 `vertx-future-wrapper` 模块中使用 `vertx-codegen` 和 `vertx-future-wrapper-gen` 来生成我们需要的代码。

> 本例使用maven来构建（gradle没有研究过，官方通过codegen生成代码的项目目前都是使用maven来构建的，所以本例也使用maven）

#### 分析

首先我们需要处理的是通过 `@VertxGen` 注解的类，因此我们不需要额外的注解，同时也不需要生成自己的Model，因此我们也不需要 `ModelProvider` 和自定义的 `Model` 类。

综上所述，我们只需要自定义 `Generator` 和加载自定义Generator的GeneratorLoader 。

同时因为我们是转换的类，所以我们需要一个注解 `FutureGen` 来标记wrapper之前的类（[vertx-rx](https://github.com/vert-x3/vertx-rx/blob/master/rx-gen/src/main/java/io/vertx/lang/rx/RxGen.java)也是这么干的）

所以我们生成的类应该是这样：

回调风格的api

```java
package io.vertx.core;
@VertxGen
public interface Api {
    void close(Handler<AsyncResult<Void>> handler);
}
```

生成的类

```java
@FutureGen(io.vertx.core.Api.class)
public class Api {
    public io.vertx.core.Future<Void> fClose(){
        return io.vertx.core.Future.future(r -> close(r));
    }
}
```

#### 解包源码

因为执行codegen需要源码，而我们需要执行的源码即为 `vert.x` 各模块的源码，因此第一次事情是将我们需要的源码打包。

我们使用maven的插件 `maven-dependency-plugin` 来解包 `vertx-core` 源码（需要生成其他包的类只需要配置此插件添加相应的包即可）

```xml
<plugin>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
      <includeGroupIds>io.vertx</includeGroupIds>
      <includeArtifactIds>
        vertx-core,
      </includeArtifactIds>
      <classifier>sources</classifier>
      <includeTypes>jar</includeTypes>
    </configuration>
    <executions>
      <execution>
        <id>unpack-java</id>
        <phase>generate-sources</phase>
        <goals>
          <goal>unpack-dependencies</goal>
        </goals>
        <configuration>
            <includes>io/vertx/**/*.java</includes>
            <excludes>**/impl/**/*.java,**/logging/**/*.java,io/vertx/groovy/**,io/vertx/reactivex/**,io/vertx/rxjava/**</excludes>
            <outputDirectory>${project.build.directory}/sources/java</outputDirectory>
        </configuration>
      </execution>
    </executions>
</plugin>
```

此配置只解包io.vertx包及子包的文件，并排除impl包中的实现类（实现类与codegen无关），logging包中的类（日志相关的类也与codegen无关）。

通过上述配置，即可将vertx-core 中我们需要的源码解压到 `vertx-future-wrapper` 模块的 `target/sources/java` 目录。

### 自定义Generator和GeneratorLoader

新建 `FutureWrapperGeneratorLoader` 和 `FutureWrapperGenerator` 类，

FutureWrapperGeneratorLoader 比较简单，我们直接放代码在这里。

```java
import io.vertx.codegen.Generator;
import io.vertx.codegen.GeneratorLoader;
import javax.annotation.processing.ProcessingEnvironment;
import java.util.stream.Stream;

public class FutureWrapperGeneratorLoader implements GeneratorLoader {
  @Override
  public Stream<Generator<?>> loadGenerators(ProcessingEnvironment processingEnv) {
    return Stream.of(new FutureWrapperGenerator());
  }
}
```

接下来我们具体分析 `FutureWrapperGenerator` 类

`FutureWrapperGenerator` 类必须继承自 `io.vertx.codegen.Generator` 类。 `Generator` 类有个泛型 `M` ，`M` 必须是 `Model` 的子类，这是我们在实现时需要关注的类型。因为我们只关注@VertxGen标识的类，因为我们只需要关注 `ClassModel`。
即这里的泛型使用ClassModel。

然后我们需要在构造方法里设置Generator的name和需要处理的kind（Model接口有个getKind方法，是Model类类型的唯一标识）。 `ClassModel` 的kind为 `class` 。

所以FutureWrapperGenerator的构造方法应该为

```java
class FutureWrapperGenerator extends Generator<ClassModel> {
  FutureWrapperGenerator() {
    this.name = "FutureWrapper";
    this.kinds = Collections.singleton("class");
  }
}
```

接着是 `filename` 方法，他用来决定生成的class的名字。这里有3种选择：

1. 类的完整









