---
title: "使用 cucumber 进行行为驱动开发（BDD）"
date: "2020-10-20 13:56:39"
tags: [ "Cucumber", "BDD" ]
categories:  ["总结"]
description: "cucumber 是 BDD 的一款很老牌的框架，使用 Ruby 进行开发，并对 Java 做了兼容"
---

# 前言

保证软件质量的方式多种多样，根据测试金字塔上所描绘的，单元测试是最有效的，但是在公司内部实践的时候却遇到了一些问题，比如赶进度来不及写单元测试、改了代码逻辑单元测试也需要跟着变、验收时同样会有很多 bug，总结下来大概有几个原因：

- 需求分析时有很多细节没有考虑到（这个不可避免）
- 测试用例覆盖的场景不全
- 分层架构需要对每一层都进行测试，会占用大量时间

还有一个原因是**公司技术环境**，我司是业务驱动的，主营业务在线下，最有价值是业务而非技术。

综合多方面的原因，决定尝试 BDD 的测试方式，这样做的好处可以分为以下几点：

- 活文档，好的用例能够体现业务的整个面貌
- 将关注点放到更有价值的业务上来，而非技术
- 对于目前团队现状，BDD 的投入产出比要比单元测试更好



# 步骤

以**用户登录**的用户故事举例：

```
作为一名供应商，我希望在输入用户名：”admin“，密码：“admin” 后能够进到系统首页，然后能够进行数据操作

验收标准：
 场景1：用户名密码正确
  Given：用户名：admin，密码：admin
  When：我在登录页上输入用户名 admin 和密码 admin
  Then：我将进入到系统首页
  
 场景2：用户名或密码不正确
 Given：用户名：admin，密码：admin
 When：我在登录页上输入用户名 admin 和密码 admin123
 Then：我被告知 “用户名或者密码错误”
 AND：此时我还是未登录状态
 
```

注意：对于用户故事验收标准这部分应该以用户行为为主，避免UI操作的字段出现，比如“我点击了提交按钮”，这样是不推荐的。

## 添加依赖

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
```

## 添加 feature 文件

在 /src/test/resource 文件夹下新建 feature 文件夹，并在此文件夹中创建 login.feature 文件。

添加一个场景

```java
#language: zh-CN
功能：供应商登录
   场景大纲: 用户名密码正确
     假设已注册的供应商用户名密码 admin admin
     当供应商输入用户名密码<username><password>
     那么能够进入到系统首页
     例子:
     |username|password|
     |admin   |admin   |
```

说明：

- \#language: zh-CN 指定使用的语言
- 功能：对应 user story 的标题
- 场景：对应功能的其中一种情况
- 假设、当、那么：cucumber 指定的中文 DSL，对于 Given、When、Then
- 对于要使用变量的地方使用 ``<`` ``>`` 标注出来并在下面的例子中列出数据内容

注意：feature 中的标点符号**全部为英文标点**

## 右键运行

此时我们还未创建任何业务代码，但是还是会有高亮展示

![image-20201020175455431](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-10-20-095457.png)

此时，我们右键点击会有下面的按钮

![image-20201020175654194](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-10-20-095658.png)

点击运行

![image-20201020175752514](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-10-20-095754.png)

因为我们还没有编写代码，所以此时测试一定会报错，那么接下来需要做的是写代码让这段测试通过

## 步骤定义

在  /src/test/java 下新建一个 login 的文件夹，并新建 **LoginStepDef** java 文件。我们的测试逻辑将在这个文件中完成。本次仅对密码正确的情况做演示

```java
@CucumberContextConfiguration
public class LoginStepDef {

    private  Account account;
        @假设("^已注册的供应商用户名密码 admin admin$")
        public void setUp(){
            account = new Account("admin","admin");
        }

        @当("^供应商输入用户名密码(\\S*)(\\S*)$")
        public boolean when_input_account_info(String username,String password){
            if (account.valid(username,password)){
                return true;
            }
            return false;
        }
       @那么("^能够进入到系统首页$")
        public void get_into_index(){

       }

    }
```

说明：

1. ``@CucumberContextConfiguration`` cucumber 提供的 Spring 集成注解

2. ``@假设("^已注册的供应商用户名密码 admin admin$")``@假设是中文的注解（中英文对照表请参见文末表格），与 feature 文件相对应。``^`` `` $``  和 shell 功能类似，代表一行的开头和结尾，固定格式。

3. 对于需要传递参数的方法需要使用正则表达式进行占位

- 整型  (\\d+)

- 字符串 (\\S*)

- 布尔值 (true|false)

- 列表 使用 DataTable 举例：

  > Given the following animals:
  >     | cow   |
  >     | horse |
  >     | sheep |
  >
  > 定义时：
  >
  > ```java
  >  @Given("the following animals:")
  >   public void the_following_animals(List<String> animals) {
  >   }
  > ```

## 配置 Junit 引导

在 /src/test/java/login 中创建名为 LoginIntegerationTest 的 java 文件

```java
@RunWith(Cucumber.class)
@CucumberOptions(features = "classpath:features/login",
        plugin = {"pretty", "html:target/cucumber/login.html"})
public class LoginIntegrationTest {
}

```

说明：

1. ``@CucumberOptions``  cucumber 的配置注解
2. ``features`` 指定 feature 文件的路径
3. ``plugin`` 将测试结果以 html 的格式输出到指定目录

## 再次运行

![image-20201021115138478](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-10-21-035140.png)

## 查看测试报告

根据配置的目录，找到对应的 html 文件

![image-20201021115248832](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-10-21-035250.png)

##  Jenkins 集成

Github 中有开源的 Jenkins Plugin： [Publish pretty cucumber-jvm reports on Jenkins](https://github.com/jenkinsci/cucumber-reports-plugin)

# 总结

如何选择合适的测试手段，需要结合企业研发团队的现状和经营模式做选型，TDD 虽然能提升质量，但并不适用所有的企业。

本文对 cucumber 做了一个基础的演示，对于复杂的环境，推荐结合 [Rest Assured](https://rest-assured.io/) 对 API 做测试。

想要了解更多可以查看文末参考资料。

# 附表

## 中英文关键字的对照表

| 英文关键字       | 中文关键字                   |
| ---------------- | ---------------------------- |
| feature          | "功能"                       |
| background       | "背景"                       |
| scenario         | "场景", "剧本"               |
| scenario_outline | "场景大纲", "剧本大纲"       |
| examples         | "例子"                       |
| given            | "* ", "假如", "假设", "假定" |
| when             | "* ", "当"                   |
| then             | "* ", "那么"                 |
| and              | "* ", "而且", "并且", "同时" |
| but              | "* ", "但是"                 |
| given (code)     | "假如", "假设", "假定"       |
| when (code)      | "当"                         |
| then (code)      | "那么"                       |
| and (code)       | "而且", "并且", "同时"       |
| but (code)       | "但是"                       |

# 参考资料

https://vitzhou.gitbooks.io/java-test-learning/content/cucumber/practice/chinese.html

https://vitzhou.gitbooks.io/java-test-learning/content/cucumber/practice/pass_parameters.html

https://developer.ibm.com/zh/articles/j-lo-cucumber01/

https://thepracticaldeveloper.com/cucumber-tests-spring-boot-dependency-injection/

# 相关书籍

[Cucumber：行为驱动开发指南](https://book.douban.com/subject/24843412/)