---
title: 技术学习：黑马程序员 JavaWeb 后端部分
date: 2025-06-07
tags: 
- Java
categories:
- 算道（课外IT技能学习）
- Lecture
description: 黑马程序员 JavaWeb 课程学习笔记
---

## Maven

Apache 下一个开源 Java 项目，用于管理，构建 Java 项目的工具。

>[!NOTE]
>
>Apache 是最大的开源软件基金会。

1. 方便快捷地管理项目依赖（jar 包），避免冲突问题。手动导入 jar 包，一个项目可能会导入上百个，彼此之间还有相互依赖，一个升级了会影响很多也要一起升级才能正常工作。maven 只需要在 pom.xml 文件里面寥寥几句就能定义要引入的 jar 包。
2. 提供统一的项目结构。不同 IDE 构建生成的项目（如 eclipse， IDEA）彼此之间不通用，而 maven 提供了一套标准项目结构。
3. 标准跨平台的自动化项目构建方式。不同平台构建项目方式可能也不同，maven 则提供了一套标准，统一的项目结构，编译后放到 target 下，还可快捷打包。2,3 如下图。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506072332264.png" alt=" " style="zoom:67%;" />

main：实际项目资源。

Java：源代码目录。

resource：配置文件目录。

test：测试项目资源。

pom.xml：项目配置文件。

> [!NOTE]
>
> pom 全称是 Project Object Model 项目对象模型。

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506072336543.png)

pom 是在 pom.xml 中的一小段信息用于描述该项目。包括组织名（groupId，通常是域名反写），模块名（项目名称，通常是模块名称，比如 order-service），版本号（这三个合起来被称作 maven 的坐标，可以唯一定义项目，或引入依赖）。

dependency 就是 pom.xml 中添加的依赖，maven 会自动去我们电脑的仓库中找对应的依赖包（本地仓库）。

中央仓库是全球唯一的，maven 核心团队运营的，几乎包含了全世界发布的所有 jar 包。

远程仓库（私服）是公司团队搭建的私有仓库。

优先查找本地 -> 私服 -> 中央仓库，只要有一个人用到一个新 jar 包，这个包就会从中央仓库下载到私服。

*安装我也不赘述了，网上好教程很多。*

引入其他项目的 maven 就是在 IDEA 中导入其 pom.xml 文件。

### 依赖配置

在 pom.xml 中写 dependencies 依赖，在其中写 dependency 单个依赖，并声明要引入的包的坐标。

然后需要点击刷新才能引入。

```xml
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.13</version>
    </dependency>
</dependencies>
```

可以去 mvnrepository.com 中找要引入的包，按下载量选择版本号比较好。

### 依赖传递

上面这个依赖其实需要其他两个依赖。不过点击刷新的时候自动全部引入了。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506080913143.png" alt=" " style="zoom: 80%;" />

直接依赖：我们通过依赖配置引入的依赖。

间接依赖：直接依赖项对其他依赖的依赖。

可以在 IDEA 中以图表形式查看依赖。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506080916676.png" alt=" " style="zoom:67%;" />

排除依赖：比如 A 依赖了 B，B 依赖了 C，但是 A 不想依赖 C：

```java
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.13</version>
    </dependency>
    <exclusion>
    	<exclusion>
    		<groupId></groupId> <!-- 在这里写不想依赖的包的组织名和模块名 -->
        	<artifactId></artifactId>
    	</exclusion>
    </exclusion>
</dependencies>
```

### 依赖范围

可以限定 jar 包在主程序范围内是否生效（main 文件夹），测试程序中是否也生效（test 文件夹），是否参与打包运行（package 指令）。

通过 scope 标签声明。

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.13</version>
</dependency>
<scope>test</scope>
```

| scope 值        | 主程序 | 测试程序 | 打包（运行） | 范例        |
| --------------- | ------ | -------- | ------------ | ----------- |
| compile（默认） | Y      | Y        | Y            | log4j       |
| test            | -      | Y        | -            | junit       |
| provided        | Y      | -        | -            | servlet-api |
| runtime         | -      | Y        | Y            | jdbc 驱动   |

在 IDEA - maven 列表 - 生命周期 - package 双击可以进行打包测试看设置不同 scope 值的 jar 包是否被引入了。

### 生命周期

maven 的生命周期就是为了对所有 maven 项目的构建进行统一。

Maven 中有 3 套相互独立的生命周期：

- **clean**：清理工作。
- **default**：核心工作，如：编译、测试、打包、安装、部署等。
- **site**：生成报告、发布站点等。

>[!NOTE]
>
>在同一套生命周期中，运行后面的工作前面也会运行，比如运行打包，编译测试也会运行。不过 clean 不会运行因为不属于同一套生命周期。

IDEA 中双击对应的生命周期项目就能执行，点击上面这个符号可以跳过测试。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506080939516.png" alt=" " style="zoom:80%;" />

也可以终端 `mvn compile` 这样执行。

>[!NOTE]
>
>其实真正执行这些命令的时候，生命周期是依赖下面这些插件来具体执行的。
>
><img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506080943557.png" alt=" " style="zoom:80%;" />

## MVC 架构

Controller：控制层，接收前端发送的请求，处理请求，响应数据。

Service：业务逻辑处理。

Dao：有的时候用 Mapper 表示，数据访问层（持久层），CRUD 数据。

分为三层架构实现，提高程序的扩展性和可维护性。

具体拆分示例如下，建立 `controller/xxxController` `servoce/xxxService` `dao/xxxDao` 三个文件将功能分开。

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506080957969.png)

### 解耦

内聚：软件中各个功能模块 **内部** 的功能联系。

耦合：软件中各个层，依赖 **之间** 的关联程度。

软件设计原则是高内聚低耦合，比如一个用户服务类，里面就尽量只包含和用户相关的服务不要包含其他的；而几个模块之间的耦合性要尽可能低。

比如上面这个例子，controller 中 新建了一个 service 类，service 中新建了一个 dao 类。这就是耦合了。如果 service dao 中发生变化了比如换了一个其他的 service 类，那么 controller service 中也要相应地改动新建对象的部分。

我们可以通过下面这种容器思想进行进一步解耦，把 service 对象放在容器里，controller 只负责从容器中拿一个 EmpService 的对象，不用关心其怎么实现的。

<img src="https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506081004138.png" alt=" " style="zoom:80%;" />

控制反转（Inversion Of Control IOC）：对象的创建控制权不是在程序中，而是转移到外部（容器）。

依赖注入（Dependency Injection DI）：容器为应用程序提供运行时所依赖的资源。

Bean 对象：IOC 容器中创建，管理的对象。比如上面的 A 对象就是 bean。

### IOC DI

IOC 实现：在要放入容器的类的声明前加一句 `@Component`：

```java
@Component
public class EmpServiceA implements EmpService {}

@Component
public class EmpDaoA implements EmpDao {}
```

这样就指明了要将其交给容器管理，并成为 IOC 容器中的 bean。

DI 实现：在声明对象，申请 bean 的语句前加一个 `@Autowired`：

```java
public class EmpController {
    @Autowired
    private EmpService empService;
    ...
}


@Component
public class EmpServiceA implements EmpService {
    @Autowired
    private EmpDao empDao;
    ...
}
```

`@Controller` `@Service` `@Repository` 分别用于定义 Controller Service Dao 的 bean 对象，如果不属于以上三种的用 `@Component`。*Repository 用的比较少，一般用 mybatis 整合。*

默认的 bean 对象名就是类名首字母小写，也可以自己指定 `@Service(value = "serviceA")` 如果只有一个参数而且是 value，可以省略 `@Service("serviceA")`

>[!WARNING]
>
>Springboot 集成 web 开发的时候控制器必须用 `@Controller` 定义。

### Bean 组件扫描

其实声明完注解，要想生效，还需要被组件扫描注解 `@ComponentScan` 扫描到。比如 `@SpringBootApplication` 中默认的扫描范围是启动类所在包以及其子包。

`@ComponentScan(value = "要扫描的包1", “要扫描的包2”)` value 可以省略。不过不推荐这种方法，最好还是 `@SpringBootApplication` 而且将所有包放在启动类所在包中。

### Bean 指定注入

如果容器中有多个符合条件的 bean，就无法分辨要注入哪个而报错。解决方式有以下两种：

1. 声明默认的 bean：

   ```java
   @Component
   @Primary
   public class Dog implements Animal { ... }
   ```

2. 在使用的时后指定要注入哪一个，这个比较常用。

   ```java
   @Autowired
   @Qualifier("cat")	// 可以直接声明对应类名小写，这样 Spring 就知道我们选的是哪个类。
   // 如果想自定义其他类名，需要在 @Component 里也声明对应的 value，如@Component("objectName")
   private Animal animal;
   ```

3. `@Resource`。

   ```java
   @Resource(name = "cat")	// 不用写 Autowired 了。Resource 是 jdk 中的注解不是 Spring 中的。
   // Autowired 默认是按照类型进行注入的，而 Resource 默认是按照名称注入的
   private Animal animal;
