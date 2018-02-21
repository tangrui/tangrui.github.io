---
title: 使用 Dubbo 对遗留单体系统进行微服务改造
author: 唐睿
date: 2016-11-21 10:51:00 +0800
categories: [技术]
tags: [微服务, Dubbo, Spring Boot, Spring Cloud, RPC, RESTful]
excerpt: >
  在 2016 年 11 月份的《技术雷达》中，ThoughtWorks 给予了微服务很高的评价。同时，也有越来越多的组织将实施微服务作为架构演进的一个必选方向。只不过在拥有众多遗留系统的组织内，将曾经的单体系统拆分为微服务并不是一件容易的事情。本文将从对遗留系统进行微服务改造的原则要求出发，探讨如何使用 Dubbo 框架实现单体系统向微服务的迁移。
layout: post
published: true
---

![巨石阵](/static/uploads/2017/dubbo/stone-henge.jpg)
<small>Credit: [Justin Kenneth Rowley](https://www.flickr.com/photos/110569055@N04/). You can find the original photo at [flickr](https://www.flickr.com/photos/110569055@N04/25341920334/in/photolist-EBnY1W-aguFHZ-D2DhnV-AC7Wp6-zTUqUV-hwq3HA-CJwnX7-CL4umw-VgEL9g-nuAybE-atpa2p-preFYp-V9rtZj-WaTqzf-ChJtWh-pvpTX1-8qsZbi-aD13TS-VNGiy7-V6v5fz-2PDDnT-RRiHZz-pcEG7c-XmYL5X-npMaVu-vHw5Q-SarYac-SarYti-8vsQ5G-pv9Phr-jYZUzi-az5qfo-Vakgmt-AKVTJm-E1sqMN-5FKt8R-oR4jnP-dmQ6Fu-oSSnLG-W8ZTHJ-f1ZRNS-qiTgmt-oazXai-QMScS7-cNrmy3-5FPJjS-fp84Ev-oBGJGq-4UR3cs-drvG8d).</small>

> The microservices style of architecture highlights rising abstractions in the developer world because of containerization and the emphasis on low coupling, offering a high level of operational isolation. Developers can think of a container as a self-contained process and the PaaS as the common deployment target, using the microservices architecture as the common style. Decoupling the architecture allows the same for teams, cutting down on coordination cost among silos. Its attractiveness to both developers and DevOps has made this the de facto standard for new development in many organizations.

在 2016 年 11 月份的《[技术雷达](https://assets.thoughtworks.com/assets/technology-radar-nov-2016-en.pdf)》中，ThoughtWorks 给予了微服务很高的评价。同时，也有越来越多的组织将实施微服务作为架构演进的一个必选方向。只不过在拥有众多遗留系统的组织内，将曾经的单体系统拆分为微服务并不是一件容易的事情。本文将从对遗留系统进行微服务改造的原则要求出发，探讨如何使用 Dubbo 框架实现单体系统向微服务的迁移。

# 一、原则要求

想要对标准三层架构的单体系统进行微服务改造——简言之——就是将曾经单一进程内服务之间的本地调用改造为跨进程的分布式调用。这虽然不是微服务改造的全部内容，但却直接决定了改造前后的系统能否保持相同的业务能力，以及改造成本的多少。

## 1.1、适合的框架

在微服务领域，虽然技术栈众多，但无非 RPC 与 RESTful 两个流派，这其中最具影响力的代表当属 [Dubbo](http://dubbo.io/) 与 [Spring Cloud](http://cloud.spring.io/) 了 。他们拥有相似的能力，却有着截然不同的实现方式——本文并不是想要对微服务框架的选型过程进行深入剖析，也不想对这两种框架的孰优孰劣进行全面比较——本章所提到的全部这些原则要求都是超越具体实现的，其之于任何微服务框架都应该是适用的。读者朋友们大可以把本文中的 Dubbo 全部替换为 Spring Cloud，而并不会对最终结果造成任何影响，唯一需要改变的仅仅是实现的细节过程而已。因此，无论最后抉择如何，都是无所谓对错的，关键在于：要选择符合组织当下现状的最适合的那一个。

## 1.2、方便的将服务暴露为远程接口

单体系统，服务之间的调用是在同一个进程内完成的；而微服务，是将独立的业务模块拆分到不同的应用系统中，每个应用系统可以作为独立的进程来部署和运行。因此进行微服务改造，就需要将进程内方法调用改造为进程间通信。进程间通信的实现方式有很多种，但显然基于网络调用的方式是最通用且易于实现的。那么能否方便的将本地服务暴露为网络服务，就决定了暴露过程能否被快速实施，同时暴露的过程越简单则暴露后的接口与之前存在不一致性的风险也就越低。

## 1.3、方便的生成远程服务调用代理

当服务被暴露为远程接口以后，进程内的本地实现将不复存在。简化调用方的使用——为远程服务生成相应的本地代理，将底层网络交互细节进行深层次的封装——就显得十分必要。另外远程服务代理在使用与功能上不应该与原有本地实现有任何差别。

## 1.4、保持原有接口不变或向后兼容

在微服务改造过程中，要确保接口不变或向后兼容，这样才不至于对调用方产生巨大影响。在实际操作过程中，我们有可能仅仅可以掌控被改造的系统，而无法访问或修改调用方系统。倘若接口发生重大变化，调用方系统的维护人员会难以接受：这会对他们的工作产生不可预估的风险和冲击，还会因为适配新接口而产生额外的工作量。

## 1.5、保持原有的依赖注入关系不变

基于 Spring 开发的遗留系统，服务之间通常是以依赖注入的方式彼此关联的。进行微服务改造后，原本注入的服务实现变成了本地代理，为了尽量减少代码变更，最好能够自动将注入的实现类切换为本地代理。

## 1.6、保持原有代码的作用或副作用效果不变

这一点看上去有些复杂，但却是必不可少的。改造后的系统跟原有系统保持相同的业务能力，当且仅当改造后的代码与原有代码保持相同的作用甚至是副作用。这里要额外提及的是副作用。我们在改造过程中可以很好的关注一般作用效果，却往往会忽视副作用的影响。举个例子，Java 内部进行方法调用的时候参数是以引用的方式传递的，这意味着在方法体中可以修改参数里的值，并将修改后的结果“返回”给被调用方。看下面的例子会更容易理解：

```java
public void innerMethod(Map<String, String> map) {
  map.put("key", "new");
}

public void outerMethod() {
  Map<String, String> map = new HashMap<>();
  map.put("key", "old");
  System.out.println(map); // {key=old}
  this.innerMethod(map);
  System.out.println(map); // {key=new}
}
```

这段代码在同一个进程中运行是没有问题的，因为两个方法共享同一片内存空间，`innerMethod` 对 `map` 的修改可以直接反映到 `outerMethod` 方法中。但是在微服务场景下事实就并非如此了，此时 `innerMethod` 和 `outerMethod` 运行在两个独立的进程中，进程间的内存相互隔离，`innerMethod` 修改的内容必须要主动回传才能被 `outerMethod` 接收到，仅仅修改参数里的值是无法达到回传数据的目的的。

此处副作用的概念是指在方法体中对传入参数的内容进行了修改，并由此对外部上下文产生了可察觉的影响。显然副作用是不友好且应该被避免的，但由于是遗留系统，我们不能保证其中不会存在诸如此类写法的代码，所以我们还是需要在微服务改造过程中，对副作用的影响效果进行保持，以获得更好的兼容性。

## 1.7、尽量少改动（最好不改动）遗留系统的内部代码

多数情况下，并非所有遗留系统的代码都是可以被平滑改造的：比如，上面提到的方法具有副作用的情况，以及传入和传出参数为不可序列化对象（未实现 `Serializable` 接口）的情况等。我们虽然不能百分之百保证不对遗留系统的代码进行修改，但至少应该保证这些改动被控制在最小范围内，尽量采取变通的方式——例如添加而不是修改代码——这种仅添加的改造方式至少可以保证代码是向后兼容的。

## 1.8、良好的容错能力

不同于进程内调用，跨进程的网络通信可靠性不高，可能由于各种原因而失败。因此在进行微服务改造的时候，远程方法调用需要更多考虑容错能力。当远程方法调用失败的时候，可以进行重试、恢复或者降级，否则不加处理的失败会沿着调用链向上传播（冒泡），从而导致整个系统的级联失败。

## 1.9、改造结果可插拔

针对遗留系统的微服务改造不可能保证一次性成功，需要不断尝试和改进，这就要求在一段时间内原有代码与改造后的代码并存，且可以通过一些简单的配置让系统在原有模式和微服务模式之间进行无缝切换。优先尝试微服务模式，一旦出现问题可以快速切换回原有模式（手动或自动），循序渐进，直到微服务模式变得稳定。

## 1.10、更多

当然微服务改造的要求远不止上面提到的这些点，还应该包括诸如：配置管理、服务注册与发现、负载均衡、网关、限流降级、扩缩容、监控和分布式事务等，然而这些需求大部分是要在微服务系统已经升级改造完毕，复杂度不断增加，流量上升到一定程度之后才会遇到和需要的，因此并不是本文关注的重点。但这并不意味着这些内容就不重要，没有他们微服务系统同样也是无法正常、平稳、高速运行的。

# 二、模拟一个单体系统

## 2.1、系统概述

我们需要构建一个具有三层架构的单体系统来模拟遗留系统，这是一个简单的 Spring Boot 应用，项目名叫做 `hello-dubbo`。本文涉及到的所有源代码均可以到 [Github](https://github.com/tangrui/hello-dubbo) 上查看和下载。

首先，系统存在一个模型 `User` 和对该模型进行管理的 DAO，并通过 `UserService` 向上层暴露访问 `User` 模型的接口；另外，还存在一个 `HelloService`，其调用 `UserService` 并返回一条问候信息；之后，由 `Controller` 对外暴露 RESTful 接口；最终再通过 Spring Boot 的 `Application` 整合成一个完整应用。

## 2.2、模块化拆分

通常来说，一个具有三层架构的单体系统，其 Controller、Service 和 DAO 是存在于一整个模块内的，如果要进行微服务改造，就要先对这个整体进行拆分。拆分的方法是以 Service 层为分界，将其分割为两个子模块：Service 层往上作为一个子模块（称为 `hello-web`），对外提供 RESTful 接口；Service 层往下作为另外一个子模块（称为 `hello-core`），包括 Service、DAO 以及模型。`hello-core` 被 `hello-web` 依赖。当然，为了更好的体现面向契约的编程精神，可以把 `hello-core` 再进一步拆分：所有的接口和模型都独立出来，形成 `hello-api`，而 `hello-core` 依赖 `hello-api`。最终，拆分后的模块关系如下：

```
hello-dubbo
|-- hello-web（包含 Application 和 Controller）
    |-- hello-core（包含 Service 和 DAO 的实现）
        |-- hello-api（包含 Service 和 DAO 的接口以及模型）
```

## 2.3、核心代码分析

### 2.3.1、User

```java
public class User implements Serializable {
  private String id;
  private String name;
  private Date createdTime;

  public String getId() {
    return this.id;
  }
  public void setId(String id) {
    this.id = id;
  }

  public String getName() {
    return this.name;
  }
  public void setName(String name) {
    this.name = name;
  }

  public Date getCreatedTime() {
    return this.createdTime;
  }
  public void setCreatedTime(Date createdTime) {
    this.createdTime = createdTime;
  }

  @Override
  public String toString() {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    if (this.getCreatedTime() != null) {
      return String.format("%s (%s)", this.getName(), sdf.format(this.getCreatedTime()));
    }

    return String.format("%s (N/A)", this.getName());
  }
}
```

`User` 模型是一个标准的 POJO，实现了 `Serializable` 接口（因为模型数据要在网络上传输，因此必须能够支持序列化和反序列化）。为了方便控制台输出，这里覆盖了默认的 `toString` 方法。

### 2.3.2、UserRepository

```java
public interface UserRepository {
  User getById(String id);

  void create(User user);
}
```

`UserRepository` 接口是访问 `User` 模型的 DAO，为了简单起见，该接口只包含两个方法：`getById` 和 `create`。

### 2.3.3、InMemoryUserRepository

```java
@Repository
public class InMemoryUserRepository implements UserRepository {
  private static final Map<String, User> STORE = new HashMap<>();

  static {
    User tom = new User();
    tom.setId("tom");
    tom.setName("Tom Sawyer");
    tom.setCreatedTime(new Date());
    STORE.put(tom.getId(), tom);
  }

  @Override
  public User getById(String id) {
    return STORE.get(id);
  }

  @Override
  public void create(User user) {
    STORE.put(user.getId(), user);
  }
}
```

`InMemoryUserRepository` 是 `UserRepository` 接口的实现类。该类型使用一个 `Map` 对象 `STORE` 来存储数据，并通过静态代码块向该对象内添加了一个默认用户。`getById` 方法根据 `id` 参数从 `STORE` 中获取用户数据，而 `create` 方法就是简单将传入的 `user` 对象存储到 `STORE` 中。由于所有这些操作都只是在内存中完成的，因此该类型被叫做 `InMemoryUserRepository`。

### 2.3.4、UserService

```java
public interface UserService {
  User getById(String id);

  void create(User user);
}
```

与 `UserRepository` 的方法一一对应，向更上层暴露访问接口。

### 2.3.5、DefaultUserService

```java
@Service("userService")
public class DefaultUserService implements UserService {
  private static final Logger LOGGER = LoggerFactory.getLogger(DefaultUserService.class);

  @Autowired
  private UserRepository userRepository;

  @Override
  public User getById(String id) {
    User user = this.userRepository.getById(id);
    LOGGER.info(user.toString());
    return user;
  }

  @Override
  public void create(User user) {
    user.setCreatedTime(new Date());
    this.userRepository.create(user);
    LOGGER.info(user.toString());
  }
}
```

`DefaultUserService` 是 `UserService` 接口的默认实现，并通过 `@Service` 注解声明为一个服务，服务 id 为 `userService`（该 id 在后面会需要用到）。该服务内部注入了一个 `UserRepository` 类型的对象 `userRepository`。`getUserById` 方法根据 `id` 从 `userRepository` 中获取数据，而 `createUser` 方法则将传入的 `user` 参数通过 `userRepository.create` 方法存入，并在存入之前设置了该对象的创建时间。很显然，根据 1.6 节关于副作用的描述，为 `user` 对象设置创建时间的操作就属于具有副作用的操作，需要在微服务改造之后加以保留。为了方便看到系统工作效果，这两个方法里面都打印了日志。

### 2.3.6、HelloService

```java
public interface HelloService {
  String sayHello(String userId);
}
```

`HelloService` 接口只提供一个方法 `sayHello`，就是根据传入的 `userId` 返回一条对该用户的问候信息。

### 2.3.7、DefaultHelloService

```java
@Service("helloService")
public class DefaultHelloService implements HelloService {
  @Autowired
  private UserService userService;

  @Override
  public String sayHello(String userId) {
    User user = this.userService.getById(userId);
    return String.format("Hello, %s.", user);
  }
}
```

`DefaultHelloService` 是 `HelloService` 接口的默认实现，并通过 `@Service` 注解声明为一个服务，服务 id 为 `helloService`（同样，该名称在后面的改造过程中会被用到）。该类型内部注入了一个 `UserService` 类型的对象 `userService`。`sayHello` 方法根据 `userId` 参数通过 `userService` 获取用户信息，并返回一条经过格式化后的消息。

### 2.3.8、Application

```java
@SpringBootApplication
public class Application {
  public static void main(String[] args) throws Exception {
    SpringApplication.run(Application.class, args);
  }
}
```

`Application` 类型是 Spring Boot 应用的入口，详细描述请参考 Spring Boot 的[官方文档](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/)，在此不详细展开。

### 2.3.9、Controller

```java
@RestController
public class Controller {
  @Autowired
  private HelloService helloService;

  @Autowired
  private UserService userService;

  @RequestMapping("/hello/{userId}")
  public String sayHello(@PathVariable("userId") String userId) {
    return this.helloService.sayHello(userId);
  }

  @RequestMapping(path = "/create", method = RequestMethod.POST)
  public String createUser(@RequestParam("userId") String userId, @RequestParam("name") String name) {
    User user = new User();
    user.setId(userId);
    user.setName(name);

    this.userService.createUser(user);
    return user.toString();
  }
}
```

`Controller` 类型是一个标准的 Spring MVC Controller，在此不详细展开讨论。仅仅需要说明的是这个类型注入了 `HelloService` 和 `UserService` 类型的对象，并在 `sayHello` 和 `createUser` 方法中调用了这两个对象中的有关方法。

## 2.4、打包运行

`hello-dubbo` 项目包含三个子模块：`hello-api`、`hello-core` 和 `hello-web`，是用 Maven 来管理的。到目前为止所涉及到的 POM 文件都比较简单，为了节约篇幅，就不在此一一列出了，感兴趣的朋友可以到项目的 Github 仓库上自行研究。

`hello-dubbo` 项目的打包和运行都非常直接：

```bash
# 编译、打包和安装
# 在项目根目录下执行命令
$ mvn clean install

# 运行
# 在 hello-web 目录下执行命令
$ mvn spring-boot:run
```

测试结果如下，注意每次输出括号里面的日期时间，它们都应该是有值的。
![样例系统测试结果](/static/uploads/2017/dubbo/test-hello-web.png)

再返回 `hello-web` 系统的控制台，查看一下日志输出，时间应该与上面是一样的。
![样例系统控制台](/static/uploads/2017/dubbo/hello-web-console.png)

# 三、动手改造

## 3.1、改造目标

上一章，我们已经成功构建了一个模拟系统，该系统是一个单体系统，对外提供了两个 RESTful 接口。本章要达到的目标是将该单体系统拆分为两个独立运行的微服务系统。如 2.2 节所述，进行模块化拆分是实施微服务改造的重要一步，因为在接下来的描述中会暗含一个约定：`hello-web`、`hello-core` 和 `hello-api` 这三个模块与上一章中所设定的能力是相同的。基于 1.7 节所提到的“尽量少改动（最好不改动）遗留系统的内部代码”的改造要求，这三个模块中的代码是不会被大面积修改的，只会有些许调整，以适应新的微服务环境。

具体将要实现的目标效果如下：

```
第一个微服务系统：

hello-web（包含 Application 和 Controller）
|-- hello-service-reference（包含 Dubbo 有关服务引用的配置）
    |-- hello-api（包含 Service 和 DAO 的接口以及模型）


第二个微服务系统：

hello-service-provider（包含 Dubbo 有关服务暴露的配置）
|-- hello-core（包含 Service 和 DAO 的实现）
    |-- hello-api（包含 Service 和 DAO 的接口以及模型）
```

`hello-web` 与原来一样，是一个面向最终用户提供 Web 服务的终端系统，其只包含 Application、Controller、Service 接口、 DAO 接口以及模型，因此它本身是不具备任何业务能力的，必须通过依赖 `hello-service-reference` 模块来远程调用 `hello-service-provider` 系统才能完成业务。而 `hello-service-provider` 系统则需要暴露可供 `hello-service-reference` 模块调用的远程接口，并实现 Service 及 DAO 接口定义的具体业务逻辑。

本章节就是要重点介绍 `hello-service-provider` 和 `hello-service-reference` 模块是如何构建的，以及它们在微服务改造过程中所起到的作用。

## 3.2、暴露远程服务

Spring Boot 和 Dubbo 的结合使用可以引入诸如 `spring-boot-starter-dubbo` 这样的[起始包](https://github.com/alibaba/dubbo-spring-boot-starter)，使用起来会更加方便。但是考虑到项目的单纯性和通用性，本文仍然延用 Spring 经典的方式进行配置。

首先，我们需要创建一个新的模块，叫做 `hello-service-provider`，这个模块的作用是用来暴露远程服务接口的。依托于 Dubbo 强大的服务暴露及整合能力，该模块不用编写任何代码，仅需添加一些配置即可完成。

**注：有关 Dubbo 的具体使用和配置说明并不是本文讨论的重点，请参考官方文档。**

### 3.2.1、添加 dubbo-services.xml 文件

`dubbo-services.xml` 配置是该模块的关键，Dubbo 就是根据这个文件，自动暴露远程服务的。这是一个标准 Spring 风格的配置文件，引入了 Dubbo 命名空间，需要将其摆放在 `src/main/resources/META-INF/spring` 目录下，这样 Maven 在打包的时候会自动将其添加到 classpath。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
  xsi:schemaLocation="
  http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
  http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

  <!-- 检索 net.tangrui.demo.dubbo.hello 包下的所有类型，并应用注解  -->
  <context:component-scan base-package="net.tangrui.demo.dubbo.hello" />

  <!-- 定义该 Dubbo 应用的名称 -->
  <dubbo:application name="hello-service-provider" />

  <!--
    - 使用广播机制进行服务注册与发现
    - 基于广播机制实现的服务注册与发现仅限于开发过程使用，因为其无需搭建和启动任何其他服务
    - 如果是生产环境请使用 Zookeeper 或 Redis
    - 详细内容请参考 Dubbo 文档
    -->
  <dubbo:registry address="multicast://224.5.6.7:1234" />

  <!-- 定义 RPC 传输协议 -->
  <dubbo:protocol name="dubbo" />

  <!--
    - 暴露 UserService 服务
    - 这里服务实现使用 ref 属性指定，属性值 userService 即为 DefaultUserService 上的注解 @Service 中指定的 id
    -->
  <dubbo:service interface="net.tangrui.demo.dubbo.hello.service.UserService"
    ref="userService" version="1.0" />

  <!-- 暴露 HelloService 服务 -->
  <dubbo:service interface="net.tangrui.demo.dubbo.hello.service.HelloService"
    ref="helloService" version="1.0" />
</beans>
```

### 3.2.2、添加 POM 文件

有关 Maven 的使用与配置也不是本文关注的重点，但是这个模块用到了一些 Maven 插件，在此对这些插件的功能和作用进行一下描述。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>net.tangrui</groupId>
    <artifactId>hello-dubbo</artifactId>
    <version>0.1.0-SNAPSHOT</version>
  </parent>

  <artifactId>hello-service-provider</artifactId>
  <packaging>jar</packaging>
  <name>Hello Service Provider</name>

  <properties>
    <slf4j.version>1.7.25</slf4j.version>
    <logback.version>1.2.3</logback.version>
    <dubbo.version>2.6.0</dubbo.version>
  </properties>

  <dependencies>
    <dependency>
      <!-- 需要依赖服务的具体实现 -->
      <groupId>net.tangrui</groupId>
      <artifactId>hello-core</artifactId>
      <version>${project.version}</version>
    </dependency>

    <!-- 引入日志组件 -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jul-to-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>log4j-over-slf4j</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>${logback.version}</version>
    </dependency>

    <!-- 引入 Dubbo -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>${dubbo.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- 从依赖包中提取文件 -->
      <plugin>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>3.0.2</version>
        <executions>
          <!-- 在 package 阶段对指定的依赖包进行解压缩 -->
          <execution>
            <id>unpack</id>
            <phase>package</phase>
            <goals>
              <goal>unpack</goal>
            </goals>
            <configuration>
              <artifactItems>
                <artifactItem>
                  <!-- 指定依赖包的 groupId, artifactId 和 version -->
                  <groupId>com.alibaba</groupId>
                  <artifactId>dubbo</artifactId>
                  <version>${dubbo.version}</version>
                  <!-- 提取依赖包中 META-INF/assembly 目录下的所有内容（主要是可执行脚本） -->
                  <includes>META-INF/assembly/**</includes>
                  <!-- 输出到指定目录 -->
                  <outputDirectory>${project.build.directory}/dubbo</outputDirectory>
                </artifactItem>
              </artifactItems>
            </configuration>
          </execution>
        </executions>
      </plugin>

      <!-- 根据 assembly.xml 文件的配置将此项目重新打包 -->
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.1.0</version>
        <configuration>
          <!-- 指定 assembly.xml 文件的路径 -->
          <descriptors>
            <descriptor>src/main/assembly/assembly.xml</descriptor>
          </descriptors>
        </configuration>
        <executions>
          <!-- 在 package 阶段进行打包 -->
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>

      <!-- 在 Maven 中直接启动应用 -->
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.6.0</version>
        <executions>
          <execution>
            <goals>
              <!-- 执行 java 命令 -->
              <goal>java</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <!-- 执行指定类型下的 main 方法 -->
          <mainClass>com.alibaba.dubbo.container.Main</mainClass>
          <!-- 向 classpath 添加额外路径，主要包含配置文件 -->
          <additionalClasspathElements>
            <additionalClasspathElement>src/main/assembly/conf</additionalClasspathElement>
          </additionalClasspathElements>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### 3.2.3、添加 assembly.xml 文件

Assembly 插件的主要功能是对项目重新打包，以便自定义打包方式和内容。对本项目而言，需要生成一个压缩包，里面包含所有运行该服务所需要的 jar 包、配置文件和启动脚本等。Assembly 插件需要 `assembly.xml` 文件来描述具体的打包过程，该文件需要摆放在 `src/main/assembly` 目录下。有关 `assembly.xml` 文件的具体配置方法，请参考[官方文档](http://maven.apache.org/plugins/maven-assembly-plugin/assembly.html)。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<assembly
  xmlns="http://maven.apache.org/ASSEMBLY/2.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">

  <id>assembly</id>

  <formats>
    <!-- 打包成 tar.gz 格式 -->
    <format>tar.gz</format>
  </formats>

  <!-- 让压缩包包含基础根目录 -->
  <includeBaseDirectory>true</includeBaseDirectory>

  <fileSets>
    <!--
      - 将 com.alibaba:dubbo 依赖包里面解压出来的文件重新打包
      - 从该依赖包里面解压文件的操作是在 POM 的 maven-dependency-plugin 插件中描述的
      -->
    <fileSet>
      <!-- 欲打包的目录 -->
      <directory>${project.build.directory}/dubbo/META-INF/assembly/bin</directory>
      <!-- 打包后的输出目录 -->
      <outputDirectory>bin</outputDirectory>
      <!-- 指定文件权限，主要是给予可执行权限 -->
      <fileMode>0755</fileMode>
    </fileSet>

    <!-- 打包配置文件 -->
    <fileSet>
      <!-- 配置文件所在目录 -->
      <directory>src/main/assembly/conf</directory>
      <!-- 打包后的输出目录 -->
      <outputDirectory>conf</outputDirectory>
      <!-- 指定文件权限，没有可执行权限 -->
      <fileMode>0644</fileMode>
    </fileSet>
  </fileSets>

  <!-- 提取本项目的所有依赖包 -->
  <dependencySets>
    <dependencySet>
      <!-- 输出到 lib 目录下 -->
      <outputDirectory>lib</outputDirectory>
    </dependencySet>
  </dependencySets>
</assembly>
```

### 3.2.4、添加 logback.xml 文件

由于在 POM 文件中指定了使用 logback 作为日志输出组件，因此还需要在 `logback.xml` 文件中对其进行配置。该文件需要摆放在 `src/main/resources` 目录下，有关该配置文件的具体内容请参见代码仓库，有关配置的详细解释，请参考[官方文档](https://logback.qos.ch/manual/configuration.html)。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration debug="true">
  <property name="applicationName" value="hello-service-provider" />

  <contextName>${applicationName}</contextName>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{yyyy-MM-dd hh:mm:ss.SSS} %-5p %logger.%M [%t] - %m%n
      </pattern>
    </encoder>
  </appender>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

### 3.2.5、打包

由于已经在 POM 文件中定义了打包的相关配置，因此直接在 `hello-service-provider` 目录下运行以下命令即可：

```bash
$ mvn clean package
```

成功执行以后，会在其 target 目录下生成一个名为 `hello-service-provider-0.1.0-SNAPSHOT-assembly.tar.gz` 的压缩包，里面的内容如图所示：

![压缩包里面的内容](/static/uploads/2017/dubbo/assembly.png)

### 3.2.6、运行

如此配置完成以后，就可以使用如下命令来启动服务：

```bash
MAVEN_OPTS="-Djava.net.preferIPv4Stack=true" mvn exec:java
```

**注：在 macOS 系统里，使用 multicast 机制进行服务注册与发现，需要添加 `-Djava.net.preferIPv4Stack=true` 参数，否则会抛出异常。**

可以使用如下命令来判断服务是否正常运行：

```bash
netstat -antl | grep 20880
```

如果有类似如下的信息输出，则说明运行正常。

![监听 20880 端口](/static/uploads/2017/dubbo/netstat.png)

如果是在正式环境运行，就需要将上一步生成的压缩包解压，然后运行 `bin` 目录下的相应脚本即可。

### 3.2.7、总结

使用这种方式来暴露远程服务具有如下一些优势：

1. 👍 使用 Dubbo 进行远程服务暴露，无需关注底层实现细节
2. 👍 对原系统没有任何入侵，已有系统可以继续按照原来的方式启动和运行
3. 👍 暴露过程可插拔
4. 👍 Dubbo 服务与原有服务在开发期和运行期均可以共存
5. 👍 无需编写任何代码

## 3.3、引用远程服务

### 3.3.1、添加服务引用

与 `hello-service-provider` 模块的处理方式相同，为了不侵入原有系统，我们创建另外一个模块，叫做 `hello-service-reference`。这个模块只有一个配置文件 `dubbo-references.xml` 放置在 `src/main/resources/META-INF/spring/` 目录下。文件的内容非常简单明了：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
  xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

  <!-- 定义该 Dubbo 应用的名称 -->
  <dubbo:application name="hello-service-reference" />

  <!-- 使用广播机制进行服务注册与发现，注意需要跟 hello-service-provider 使用相同的广播地址 -->
  <dubbo:registry address="multicast://224.5.6.7:1234" />

  <!-- 定义 RPC 传输协议 -->
  <dubbo:protocol name="dubbo" />

  <!-- 引用 hello-service-provider 暴露出来的 UserService 服务 -->
  <dubbo:reference id="userServiceReference" interface="net.tangrui.demo.dubbo.hello.service.UserService"
    version="1.0" />

  <!-- 引用 hello-service-provider 暴露出来的 HelloService 服务 -->
  <dubbo:reference id="helloServiceReference" interface="net.tangrui.demo.dubbo.hello.service.HelloService"
    version="1.0" />
</beans>
```

但不同于 `hello-service-provider` 模块的一点在于，该模块只需要打包成一个 jar 即可，POM 文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>net.tangrui</groupId>
    <artifactId>hello-dubbo</artifactId>
    <version>0.1.0-SNAPSHOT</version>
  </parent>

  <artifactId>hello-service-reference</artifactId>
  <packaging>jar</packaging>
  <name>Hello Service Reference</name>

  <properties>
    <dubbo.version>2.6.0</dubbo.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>net.tangrui</groupId>
      <artifactId>hello-api</artifactId>
      <version>${project.version}</version>
    </dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>${dubbo.version}</version>
    </dependency>
  </dependencies>
</project>
```

总结一下，我们曾经的遗留系统分为三个模块 `hello-web`, `hello-core` 和 `hello-api`。经过微服务化处理以后，`hello-core` 和 `hello-api` 被剥离了出去，加上 `hello-service-provider` 模块，形成了一个可以独立运行的 `hello-service-provider` 系统，因此需要打包成一个完整的应用；而 `hello-web` 要想调用 `hello-core` 提供的服务，就不能再直接依赖 `hello-core` 模块了，而是需要依赖我们这里创建的 `hello-service-reference` 模块，因此 `hello-service-reference` 是作为一个依赖库出现的，其目的就是远程调用 `hello-service-provider` 暴露出来的服务，并提供本地代理。

这时 `hello-web` 模块的依赖关系就发生了变化：原来 `hello-web` 模块直接依赖 `hello-core`，再通过 `hello-core` 间接依赖 `hello-api`，而现在我们需要将其改变为直接依赖 `hello-service-reference` 模块，再通过 `hello-service-reference` 模块间接依赖 `hello-api`。改造前后的依赖关系分别为：

```xml
<!-- 改造前 -->
<dependencies>
  ...
  <dependency>
    <groupId>net.tangrui</groupId>
    <artifactId>hello-core</artifactId>
    <version>${project.version}</version>
  </dependency>
  ...
</dependencies>

<!-- 改造后 -->
<dependencies>
  ...
  <dependency>
    <groupId>com.yunzhijia</groupId>
    <artifactId>hello-service-reference</artifactId>
    <version>${project.version}</version>
  </dependency>
  ...
</dependencies>
```

### 3.3.2、启动服务

因为是测试环境，只需要执行以下命令即可，但在进行本操作之前，需要先启动 `hello-service-provider` 服务。

```bash
MAVEN_OPTS="-Djava.net.preferIPv4Stack=true" mvn spring-boot:run
```

Oops！系统并不能像期望的那样正常运行，会抛出如下异常：

![启动失败](/static/uploads/2017/dubbo/failed-to-start.png)

意思是说 `net.tangrui.demo.dubbo.hello.web.Controller` 这个类的 `helloService` 字段需要一个类型为  `net.tangrui.demo.dubbo.hello.service.HelloService` 的 Bean，但是没有找到。相关代码片段如下：

```java
@RestController
Public class Controller {
  @Autowired
  private HelloService helloService;

  @Autowired
  private UserService userService;

  ...
}
```

显然，`helloService` 和 `userService` 都是无法注入的，这是为什么呢？

原因自然跟我们修改 `hello-web` 这个模块的依赖关系有关。原本 `hello-web` 是依赖于 `hello-core` 的，`hello-core` 里面声明了 `HelloService` 和 `UserService` 这两个服务（通过 `@Service` 注解），然后 `Controller` 在 `@Autowired` 的时候就可以自动绑定了。但是，现在我们将 `hello-core` 替换成了 `hello-service-reference`，在 `hello-service-reference` 的配置文件中声明了两个对远程服务的引用，按道理来说这个注入应该是可以生效的，但显然实际情况并非如此。

仔细思考不难发现，我们在执行 `mvn exec:java` 命令启动 `hello-service-provider` 模块的时候指定了启动 `com.alibaba.dubbo.container.Main` 类型，然后才会开始启动并加载 Dubbo 的有关配置，这一点从日志中可以得到证实（日志里面会打印出来很多带有 `[DUBBO]` 标签的内容），显然在这次运行中，我们并没有看到类似这样的日志，说明 Dubbo 在这里没有被正确启动。归根结底还是 Spring Boot 的原因，即 Spring Boot 需要一些配置才能够正确加载和启动 Dubbo。

让 Spring Boot 支持 Dubbo 有很多种方法，比如前面提到的 `spring-boot-starter-dubbo` 起始包，但这里同样为了简单和通用，我们依旧采用经典的方式来解决。

继续思考，该模块没有成功启动 Dubbo，仅仅是因为添加了对 `hello-service-reference` 的引用，而 `hello-service-reference` 模块就只有一个文件 `dubbo-references.xml`，这就说明 Spring Boot 并没有加载到这个文件。顺着这个思路，只需要让 Spring Boot 能够成功加载这个文件，问题就可以了。Spring Boot 也确实提供了这样的能力，只可惜无法完全做到代码无侵入，只能说这些改动是可以被接受的。修改方式是替换 `Application` 中的注解（至于为什么要修改成这样的结果，超出了本文的讨论范围，请自行 Google）。

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
@ImportResource("classpath:META-INF/spring/dubbo-references.xml")
public class Application {
  public static void main(String[] args) throws Exception {
    SpringApplication.run(Application.class, args);
  }
}
```

这里的主要改动，是将一个 `@SpringBootApplication` 注解替换为 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 和 `@ImportResource` 四个注解。不难看出，最后一个 `@ImportResource` 就是我们需要的。

这时再重新尝试启动，就一切正常了。

![访问 http://127.0.0.1:8080/hello/tom](/static/uploads/2017/dubbo/hello-tom.png)

但是，我们如何验证结果确实是从 `hello-service-provider` 服务过来的呢？这时就需要用到 `DefaultUserService` 里面的那几行日志输出了，回到 `hello-service-provider` 服务的控制台，能够看到类似这样的输出：

![HERE IT IS](/static/uploads/2017/dubbo/here-it-is.png)

如此便可以确信系统的拆分是被成功实现了。再试试创建用户的接口：

```bash
curl -X POST 'http://127.0.0.1:8080/create?userId=huckleberry&name=Huckleberry%20Finn'
```

![创建用户失败](/static/uploads/2017/dubbo/create-user-failed.png)

等等，什么！括号里面的创建时间为什么是 `N/A`，这说明 `createdTime` 字段根本没有值！

## 3.4、保持副作用效果

让我们先来回顾一下 1.6 节所提到的副作用效果。在 `DefaultUserService.create` 方法中，我们为传入的 `user` 参数设置了创建时间，这一操作就是我们要关注的具有副作用效果的操作。

先说单体系统的情况。单体系统是运行在一个 Java 虚拟机中的，所有对象共享一片内存空间，彼此可以互相访问。系统在运行的时候，先是由 `Controller.create` 方法获取用户输入，将输入的参数封装为一个 `user` 对象，再传递给 `UserService.create` 方法（具体是在调用 `DefaultUserService.create` 方法），这时 `user` 对象的 `createdTime` 字段就被设置了。由于 Java 是以引用的方式来传递参数，因此在 `create` 方法中对 `user` 对象所做的变更，是能够反映到调用方那里的——即 `Controller.create` 方法里面也是可以获取到变更的，所以返回给用户的时候，这个 `createdTime` 就是存在的。

再说微服务系统的情况。此时系统是独立运行在两个虚拟机中的，彼此之间的内存是相互隔离的。起始点同样是 `hello-web` 系统的 `Controller.create` 方法：获取用户输入，封装 `user` 对象。可是在调用 `UserService.create` 方法的时候，并不是直接调用 `DefaultUserService` 中的方法，而是调用了一个具有相同接口的本地代理，这个代理将 `user` 对象序列化之后，通过网络传输给了 `hello-service-provider` 系统。该系统接收到数据以后，先进行反序列化，生成跟原来对象一模一样的副本，再由 `UserService.create` 方法进行处理（这回调用的就是 `DefaultUserService` 里面的实现了）。至此，这个被设置过 `createdTime` 的 `user` 对象副本是一直存在于 `hello-service-provider` 系统的内存里面的，从来没有被传递出去，自然是无法被 `hello-web` 系统读取到的，所以最终打印出来的结果，括号里面的内容就是 `N/A` 了。记得我们有在 `DefaultUserService.create` 方法中输出过日志，所以回到 `hello-service-provider` 系统的控制台，可以看到如下的日志信息，说明在这个系统里面 `createdTime` 字段确实是有值的。

![服务提供方的日志](/static/uploads/2017/dubbo/create-user-failed-provider-log.png)

那么该如何让这个副作用效果也能够被处于另外一个虚拟机中的 `hello-web` 系统感知到呢，方法只有一种，就是将变更后的数据回传。

### 3.4.1、为方法添加返回值

这是最容易想到的一种实现方式，简单的说就是修改服务接口，将变更后的数据返回。

首先，修改 `UserService` 接口的 `create` 方法，添加返回值：

```java
public interface UserService {
  ...

  // 为方法添加返回值
  User create(User user);
}
```

然后，修改实现类中相应的方法，将变更后的 `user` 对象返回：

```java
@Service("userService")
public class DefaultUserService implements UserService {
  ...

  @Override
  public User create(User user) {
    user.setCreatedTime(new Date());
    this.userRepository.create(user);
    LOGGER.info(user.toString());
    // 将变更后的数据返回
    return user;
  }
}
```

最后，修改调用方实现，接收返回值：

```java
@RestController
public class Controller {
  ...

  @RequestMapping(path = "/create", method = RequestMethod.POST)
  public String createUser(@RequestParam("userId") String userId, @RequestParam("name") String name) {
    User user = new User();
    user.setId(userId);
    user.setName(name);

    // 接收返回值
    User newUser = this.userService.create(user);
    return newUser.toString();
  }
}
```

编译、运行并测试（如下图），正如我们所期望的，括号中的创建时间又回来了。其工作原理与本节开始时所描述的是一样的，只是方向相反而已。在此不再详细展开，留给大家自行思考。

![创建用户功能又可以正常工作了](/static/uploads/2017/dubbo/create-user-worked-again.png)

这种修改方式有如下一些优缺点：

1. 👍 方法简单，容易理解
2. 👎 改变了系统接口，且改变后的接口与原有接口不兼容（违背了 1.4 节关于“保持原有接口不变或向后兼容”原则的要求）
3. 👎 由此也不可避免的造成了对遗留系统内部代码的修改（违背了 1.7 节关于“尽量少改动（最好不改动）遗留系统的内部代码”原则的要求）
4. 👎 修改方式不可插拔（违背了 1.9 节“改造结果可插拔”原则的要求）

由此可见，这种改法虽然简单，却是利大于弊的，除非我们能够完全掌控整个系统，否则这种修改方式的风险会随着系统复杂性的增加而急剧上升。

### 3.4.2、添加一个新方法

如果不能做到不改变接口，那我们至少要做到改变后的接口与原有接口向后兼容。保证向后兼容性的一种解决办法，就是不改变原有方法，而是添加一个新的方法。过程如下：

首先，为 `UserService` 接口添加一个新的方法 `__rpc_create`。这个方法名虽然看起来有些奇怪，但却有两点好处：第一、不会和已有方法重名，因为 Java 命名规范不建议使用这样的标识符来为方法命名；第二、在原有方法前加上 `__rpc_` 前缀，能够做到与原有方法对应，便于阅读和理解。示例如下：

```java
public interface UserService {
  ...

  // 保持原有方法不变
  void create(User user);

  // 添加一个方法，新方法需要有返回值
  User __rpc_create(User user);
}
```

然后，在实现类中实现这个新方法：

```java
@Service("userService")
public class DefaultUserService implements UserService {
  ...

  // 保持原有方法实现不变
  @Override
  public void create(User user) {
    user.setCreatedTime(new Date());
    this.userRepository.create(user);
    LOGGER.info(user.toString());
  }

  // 添加新方法的实现
  @Override
  public User __rpc_create(User user) {
    // 调用原来的方法
    this.create(user);
    // 返回变更后的数据
    return user;
  }
}
```

有一点需要展开解释：在 `__rpc_create` 方法中，因为 `user` 参数是以引用的方式传递给 `create` 方法的，因此 `create` 方法对参数所做的修改是能够被 `__rpc_create` 方法获取到的。这以后就与前面回传的逻辑是相同的了。

第三，在服务引用端添加本地存根（有关本地存根的概念及用法，请参考[官方文档](http://dubbo.io/books/dubbo-user-book/demos/local-stub.html)）。

需要在 `hello-service-reference` 模块中添加一个类 `UserServiceStub`，内容如下：

```java
public class UserServiceStub implements UserService {
  private UserService userService;

  public UserServiceStub(UserService userService) {
    this.userService = userService;
  }

  @Override
  public User getById(String id) {
    return this.userService.getById(id);
  }

  @Override
  public void create(User user) {
    User newUser = this.__rpc_create(user);
    user.setCreatedTime(newUser.getCreatedTime());
  }

  @Override
  public User __rpc_create(User user) {
    return this.userService.__rpc_create(user);
  }
}
```

该类型即为本地存根。简单来说，就是在调用方调用本地代理的方法之前，会先去调用本地存根中相应的方法，因此本地存根与服务提供方和服务引用方需要实现同样的接口。本地存根中的构造函数是必须的，且方法签名也是被约定好的——需要传入本地代理作为参数。其中 `getById` 和 `__rpc_create` 方法都是直接调用了本地代理中的方法，不必过多关注，重点来说说 `create` 方法。首先，`create` 调用了本地存根中的 `__rpc_create` 方法，这个方法透过本地代理访问到了服务提供方的相应方法，并成功接收了返回值 `newUser`，这个返回值是包含修改后的 `createdTime` 字段的，于是我们要做的事情就是从 `newUser` 对象里面获取到 `createdTime` 字段的值，并设置给 `user` 参数，以达到产生副作用的效果。此时 `user` 参数会带着新设置的 `createdTime` 的值，将其“传递”给 `create` 方法的调用方。

最后，在 `dubbo-references.xml` 文件中修改一处配置，以启用该本地存根：

```xml
<!-- 最后一行声明了使用该本地存根 -->
<dubbo:reference id="userServiceReference"
  interface="net.tangrui.demo.dubbo.hello.service.UserService"
  version="1.0"
  stub="net.tangrui.demo.dubbo.hello.service.stub.UserServiceStub" />
```

鉴于本地存根的工作机制，我们是不需要修改调用方 `hello-web` 模块中的任何代码及配置的。编译、运行并测试，同样可以达到我们想要的效果。

![第二种实现方式](/static/uploads/2017/dubbo/create-user-using-stub.png)

这种实现方式会比第一种方式改进不少，但也有致命弱点：

1. 👍 保持了接口的向后兼容性
2. 👍 引入本地存根，无需修改调用方代码
3. 👍 通过配置可以实现改造结果的可插拔
4. 👎 实现复杂，尤其是本地存根的实现，如果遗留系统的代码对传入参数里的内容进行了无节制的修改的话，那么重现该副作用效果是非常耗时且容易出错的
5. 👎 难以理解

# 五、总结

至此，将遗留系统改造为微服务系统的任务就大功告成了，而且基本上满足了文章最开始提出来的十点改造原则与要求（此处应给自己一些掌声👏👏👏），不知道是否对大家有所帮助？虽然示例项目是为了叙述要求而量身定制的，但文章中提到的种种理念与方法却实实在在是从实践中摸索和总结出来的——踩过的坑，遇到的问题，解决的思路以及改造的难点等都一一呈现给了大家。

微服务在当下已经不是什么新鲜的技术了，但历史包袱依然是限制其发展的重要因素，希望这篇文章能带给大家一点启发，在接下来的工作中更好的拥抱微服务带来的变革。

**版权所有，欢迎转载，转载请注明作者及出处。**
