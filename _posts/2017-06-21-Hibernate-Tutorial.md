---
title: "Hibernate"
layout: post
date: 2017-06-21 22:44
tag:
- Hibernate
- JDBC
- ORM
- Configuration
- Environment
star: true
category: blog
author: jiaqixu
description: Hibernate Knowledge
---

## Hibernate 教程
Hibernate 是一个高性能的对象/关系型持久化存储和查询的服务，其遵循开源的 GNU Lesser General Public License (LGPL) 而且可以免费下载。Hibernate 不仅关注于从 Java 类到数据库表的映射（也有 Java 数据类型到 SQL 数据类型的映射），另外也提供了数据查询和检索服务。

这个教程将指导你如何以简单的方式使用 Hibernate 来开发基于数据库的 Web 应用程序。

### 目录
* TOC
{:toc}

## ORM 概览

什么是JDBC？<br>

JDBC 代表 Java Database Connectivity ，它是提供了一组 Java API 来访问关系数据库的 Java 程序。这些 Java APIs 可以使 Java 应用程序执行 SQL 语句，能够与任何符合 SQL 规范的数据库进行交互。

JDBC 提供了一个灵活的框架来编写操作数据库的独立的应用程序，该程序能够运行在不同的平台上且不需修改，能够与不同的 DBMS 进行交互。

为什么是对象关系映射（ORM）？<br>

当我们工作在一个面向对象的系统中时，存在一个对象模型和关系数据库不匹配的问题。RDBMSs 用表格的形式存储数据，然而像 Java 或者 C# 这样的面向对象的语言它表示一个对象关联图。考虑下面的带有构造方法和公有方法的 Java 类：
{% highlight html %}
public class Employee {
   private int id;
   private String first_name;
   private String last_name;   
   private int salary;  

   public Employee() {}
   public Employee(String fname, String lname, int salary) {
      this.first_name = fname;
      this.last_name = lname;
      this.salary = salary;
   }
   public int getId() {
      return id;
   }
   public String getFirstName() {
      return first_name;
   }
   public String getLastName() {
      return last_name;
   }
   public int getSalary() {
      return salary;
   }
}
{% endhighlight %}

现考虑以上的对象需要被存储和索引进下面的 RDBMS 表格中：
{% highlight html %}
create table EMPLOYEE (
   id INT NOT NULL auto_increment,
   first_name VARCHAR(20) default NULL,
   last_name  VARCHAR(20) default NULL,
   salary     INT  default NULL,
   PRIMARY KEY (id)
);
{% endhighlight %}

第一个问题，如果我们开发了几页代码或应用程序后，需要修改数据库的设计怎么办？第二个问题，在关系型数据库中加载和存储对象时我们要面临以下五个不匹配的问题:粒度，继承，身份，关联和导航。

Solution: Object-Relational Mapping (ORM) 是解决以上所有不匹配问题的方案。

什么是ORM？<br>

ORM 表示 Object-Relational Mapping (ORM)，是一个方便在关系数据库和类似于 Java， C# 等面向对象的编程语言中转换数据的技术。一个 ORM 系统相比于普通的 JDBC 有以下的优点：<br>
<ol>
<li>使用业务代码访问对象而不是数据库中的表</li>
<li>从面向对象逻辑中隐藏 SQL 查询的细节</li>
<li>基于 JDBC 的 'under the hood'</li>
<li>没有必要去处理数据库实现</li>
<li>实体是基于业务的概念而不是数据库的结构</li>
<li>事务管理和键的自动生成</li>
<li>应用程序的快速开发</li>
</ol>

一个 ORM 解决方案由以下四个实体组成：
<ol>
<li>一个 API 来在持久类的对象上实现基本的 CRUD 操作</li>
<li>一个语言或 API 来指定引用类和属性的查询</li>
<li>基一个可配置的服务用来指定映射元数据</li>
<li>没有必要去处理数据库实现</li>
<li>一个技术和事务对象交互来执行 dirty checking, lazy association fetching 和其它优化的功能</li>
</ol>

## Hibernate 简介
Hibernate 是由 Gavin King 于 2001 年创建的开放源代码的对象关系框架。它强大且高效的构建具有关系对象持久性和查询服务的 Java 应用程序。

Hibernate 将 Java 类映射到数据库表中，从 Java 数据类型中映射到 SQL 数据类型中，并把开发人员从 95% 的公共数据持续性编程工作中解放出来。

Hibernate 是传统 Java 对象和数据库服务器之间的桥梁，用来处理基于 O/R 映射机制和模式的那些对象。
<img src="/assets/images/blog/HibernateORM.JPG">

Hibernate优势
- Hibernate 使用 XML 文件来处理映射 Java 类别到数据库表格中，并且不用编写任何代码。
- 为在数据库中直接储存和检索 Java 对象提供简单的 APIs。
- 如果在数据库中或任何其它表格中出现变化，那么仅需要改变 XML 文件属性。
- 抽象不熟悉的 SQL 类型，并为我们提供工作中所熟悉的 Java 对象。
- Hibernate 不需要应用程序服务器来操作。
- 操控你数据库中对象复杂的关联。
- 最小化与访问数据库的智能提取策略。
- 提供简单的数据询问。

## Hibernate 架构
Hibernate 架构是分层的，作为数据访问层，你不必知道底层 API 。Hibernate 利用数据库以及配置数据来为应用程序提供持续性服务（以及持续性对象）。

下面是一个非常高水平的 Hibernate 应用程序架构视图。
<img src="/assets/images/blog/HibernateFramework.JPG">

下面是一个详细的 Hibernate 应用程序体系结构视图以及一些重要的类。
<img src="/assets/images/blog/HibernateArchitecture.JPG">

Hibernate 使用不同的现存 Java API，比如 JDBC，Java 事务 API（JTA），以及 Java 命名和目录界面（JNDI）。JDBC 提供了一个基本的抽象级别的通用关系数据库的功能， Hibernate 支持几乎所有带有 JDBC 驱动的数据库。JNDI 和 JTA 允许 Hibernate 与 J2EE 应用程序服务器相集成。

下面的部分简要地描述了在 Hibernate 应用程序架构所涉及的每一个类对象。

配置对象
配置对象是你在任何 Hibernate 应用程序中创造的第一个 Hibernate 对象，并且经常只在应用程序初始化期间创造。它代表了 Hibernate 所需一个配置或属性文件。配置对象提供了两种基础组件:
- 数据库连接：由 Hibernate 支持的一个或多个配置文件处理。这些文件是 hibernate.properties 和 hibernate.cfg.xml。
- 类映射设置：这个组件创造了 Java 类和数据库表格之间的联系。

SessionFactory 对象
配置对象被用于创造一个 SessionFactory 对象，使用提供的配置文件为应用程序依次配置 Hibernate，并允许实例化一个会话对象。SessionFactory 是一个线程安全对象并由应用程序所有的线程所使用。

SessionFactory 是一个重量级对象所以通常它都是在应用程序启动时创造然后留存为以后使用。每个数据库需要一个 SessionFactory 对象使用一个单独的配置文件。所以如果你使用多种数据库那么你要创造多种 SessionFactory 对象。

Session 对象
一个会话被用于与数据库的物理连接。Session 对象是轻量级的，并被设计为每次实例化都需要与数据库的交互。持久对象通过 Session 对象保存和检索。

Session 对象不应该长时间保持开启状态因为它们通常情况下并非线程安全，并且它们应该按照所需创造和销毁。

Transaction 对象
一个事务代表了与数据库工作的一个单元并且大部分 RDBMS 支持事务功能。在 Hibernate 中事务由底层事务管理器和事务（来自 JDBC 或者 JTA）处理。

这是一个选择性对象，Hibernate 应用程序可能不选择使用这个接口，而是在自己应用程序代码中管理事务。

Query 对象
Query 对象使用 SQL 或者 Hibernate 查询语言（HQL）字符串在数据库中来检索数据并创造对象。一个查询的实例被用于连结查询参数，限制由查询返回的结果数量，并最终执行查询。

Criteria 对象
Criteria 对象被用于创造和执行面向规则查询的对象来检索对象。

## Environment Configuration
1. 下载Hibernate（我用的是hibernate-distribution-3.6.4.Final ）和msql-jdbc的包(因为我用的是mysql数据库)， 安装在计算机上，并解压如图所示：
<img src="/assets/images/blog/HibernateJARAndmysqlJDBC.JPG">
<img src="/assets/images/blog/HibernateJar.JPG">
2. 导入eclipse, 并且add to build path。<br>
<img src="/assets/images/blog/importJar.JPG">
3. 设置配置文件hibernate.cfg.xml
{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
   <session-factory>
   <property name="hibernate.dialect">
      org.hibernate.dialect.MySQLDialect
   </property>
   <property name="hibernate.connection.driver_class">
      com.mysql.jdbc.Driver
   </property>

   <!-- Assume test is the database name -->
   <property name="hibernate.connection.url">
      jdbc:mysql://localhost/test
   </property>
   <property name="hibernate.connection.username">
      root
   </property>
   <property name="hibernate.connection.password">
      root123
   </property>

   <!-- List of XML mapping files -->
   <mapping resource="Employee.hbm.xml"/>

</session-factory>
</hibernate-configuration>
{% endhighlight %}


## 会话 Session
ession 用于获取与数据库的物理连接。 Session 对象是轻量级的，并且设计为在每次需要与数据库进行交互时被实例化。持久态对象被保存，并通过 Session 对象检索找回。

该 Session 对象不应该长时间保持开放状态，因为它们通常不能保证线程安全，而应该根据需求被创建和销毁。Session 的主要功能是为映射实体类的实例提供创建，读取和删除操作。这些实例可能在给定时间点时存在于以下三种状态之一：

- 瞬时状态(Transient Objects): 由Java的new命令开辟内存空间的java对象也就是普通的java对象，如果没有变量引用它它将会被JVM收回。临时对象在内存中是孤立存在的，它的意义是携带信息载体，不和数据库中的数据由任何的关联。通过Session的save（）方法和saveOrUpdate（）方法可以把一个临时对象和数据库相关联，并把临时对象携带的信息通过配置文件所做的映射插入数据库中，这个临时对象就成为持久化对象。
- 持久状态(Persistent Objects)：持久化对象在数据库中有相应的记录，持久化对象可以是刚被保存的，或者刚被加载的，但都是在相关联的session声明周期中保存这个状态。如果是直接数据库查询所返回的数据对象，则这些对象和数据库中的字段相关联，具有相同的id，它们马上变成持久化对象。如果一个临时对象被持久化对象引用，也立马变为持久化对象。
如果使用delete（）方法，持久化对象变为临时对象，并且删除数据库中相应的记录，这个对象不再与数据库有任何的联系。
持久化对象总是与Session和Transaction关联在一起，在一个session中，对持久化对象的操作不会立即写到数据库，只有当Transaction（事务）结束时，才真正的对数据库更新，从而完成持久化对象和数据库的同步。在同步之前的持久化对象成为脏对象。
当一个session（）执行close（）、clear（）、或evict（）之后，持久化对象就变为离线对象，这时对象的id虽然拥有数据库的识别值，但已经不在Hibernate持久层的管理下，他和临时对象基本上一样的，只不过比临时对象多了数据库标识id。没有任何变量引用时，jvm将其回收。
- 脱管状态(Detached Objects)：Session关闭之后，与此Session关联的持久化对象就变成为脱管对象，可以继续对这个对象进行修改，如果脱管对象被重新关联到某个新的Session上，会在此转成持久对象。
脱管对象虽然拥有用户的标识id，所以通过update（）、saveOrUpdate（）等方法，再次与持久层关联。

若 Session 实例的持久态类别是序列化的，则该 Session 实例是序列化的。一个典型的事务应该使用以下语法：
{% highlight html %}
Session session = factory.openSession();
Transaction tx = null;
try {
   tx = session.beginTransaction();
   // do some work
   ...
   tx.commit();
}
catch (Exception e) {
   if (tx!=null) tx.rollback();
   e.printStackTrace();
}finally {
   session.close();
}
{% endhighlight %}

如果 Session 引发异常，则事务必须被回滚，该 session 必须被丢弃。

## 持久化类 Persistent Classes
Hibernate 的完整概念是提取 Java 类属性中的值，并且将它们保存到数据库表单中。映射文件能够帮助 Hibernate 确定如何从该类中提取值，并将它们映射在表格和相关域中。

在 Hibernate 中，其对象或实例将会被存储在数据库表单中的 Java 类被称为持久化类。若该类遵循一些简单的规则或者被大家所熟知的 Plain Old Java Object (POJO) 编程模型，Hibernate 将会处于其最佳运行状态。以下所列就是持久化类的主要规则，然而，在这些规则中，没有一条是硬性要求。
- 所有将被持久化的 Java 类都需要一个默认的构造函数。
- -为了使对象能够在 Hibernate 和数据库中容易识别，所有类都需要包含一个 ID。此属性映射到数据库表的主键列。
-  private，并具有由 JavaBean 风格定义的 getXXX 和 setXXX 方法。
- Hibernate 的一个重要特征为代理，它取决于该持久化类是处于非 final 的，还是处于一个所有方法都声明为 public 的接口。
- 所有的类是不可扩展或按 EJB 要求实现的一些特殊的类和接口。

POJO 的名称用于强调一个给定的对象是普通的 Java 对象，而不是特殊的对象，尤其不是一个 Enterprise JavaBean。

一个简单的 POJO 的例子

基于以上所述规则，我们能够定义如下 POLO 类：
{% highlight html %}
public class Employee {
   private int id;
   private String firstName;
   private String lastName;   
   private int salary;  

   public Employee() {}
   public Employee(String fname, String lname, int salary) {
      this.firstName = fname;
      this.lastName = lname;
      this.salary = salary;
   }
   public int getId() {
      return id;
   }
   public void setId( int id ) {
      this.id = id;
   }
   public String getFirstName() {
      return firstName;
   }
   public void setFirstName( String first_name ) {
      this.firstName = first_name;
   }
   public String getLastName() {
      return lastName;
   }
   public void setLastName( String last_name ) {
      this.lastName = last_name;
   }
   public int getSalary() {
      return salary;
   }
   public void setSalary( int salary ) {
      this.salary = salary;
   }
}
{% endhighlight %}

## 映射文件 Mapping File
一个对象/关系型映射一般定义在 XML 文件中。映射文件指示 Hibernate 如何将已经定义的类或类组与数据库中的表对应起来。

尽管有些 Hibernate 用户选择手写 XML 文件，但是有很多工具可以用来给先进的 Hibernate 用户生成映射文件。这样的工具包括 XDoclet, Middlegen 和 AndroMDA。

让我们来考虑我们之前定义的 POJO 类，它的对象将延续到下一部分定义的表中。
{% highlight html %}
public class Employee {
    private int id;
    private String firstName;
    private String lastName;   
    private int salary;  

    public Employee() {}
    public Employee(String fname, String lname, int salary) {
        this.firstName = fname;
        this.lastName = lname;
        this.salary = salary;
    }
    public int getId() {
        return id;
    }
    public void setId( int id ) {
        this.id = id;
    }
    public String getFirstName() {
        return firstName;
    }
    public void setFirstName( String first_name ) {
        this.firstName = first_name;
    }
    public String getLastName() {
        return lastName;
    }
    public void setLastName( String last_name ) {
        this.lastName = last_name;
    }
    public int getSalary() {
        return salary;
    }
    public void setSalary( int salary ) {
        this.salary = salary;
    }
}
{% endhighlight %}

对于每一个你想要提供持久性的对象都需要一个表与之保持一致。考虑上述对象需要存储和检索到下列 RDBMS 表中：
{% highlight html %}
create table EMPLOYEE (
    id INT NOT NULL auto_increment,
    first_name VARCHAR(20) default NULL,
    last_name  VARCHAR(20) default NULL,
    salary     INT  default NULL,
    PRIMARY KEY (id)
);
{% endhighlight %}

基于这两个实体之上，我们可以定义下列映射文件来指示 Hibernate 如何将已定义的类或类组与数据库表匹配。
{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
 "-//Hibernate/Hibernate Mapping DTD//EN"
 "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
   <class name="Employee" table="EMPLOYEE">
      <meta attribute="class-description">
         This class contains the employee detail.
      </meta>
      <id name="id" type="int" column="id">
         <generator class="native"/>
      </id>
      <property name="firstName" column="first_name" type="string"/>
      <property name="lastName" column="last_name" type="string"/>
      <property name="salary" column="salary" type="int"/>
   </class>
</hibernate-mapping>
{% endhighlight %}

你需要以格式 <classname>.hbm.xml保存映射文件。我们保存映射文件在 Employee.hbm.xml 中。让我们来详细地看一下在映射文件中使用的一些标签:
- 映射文件是一个以 <hibernate-mapping> 为根元素的 XML 文件，里面包含所有<class>标签。
- <class> 标签是用来定义从一个 Java 类到数据库表的特定映射。Java 的类名使用 name 属性来表示，数据库表明用 table 属性来表示。
- <meta> 标签是一个可选元素，可以被用来修饰类。
- <id> 标签将类中独一无二的 ID 属性与数据库表中的主键关联起来。id 元素中的 name 属性引用类的性质，column 属性引用数据库表的列。type 属性保存 Hibernate 映射的类型，这个类型会将从 Java 转换成 SQL 数据类型。
- 在 id 元素中的 <generator> 标签用来自动生成主键值。设置 generator 标签中的 class 属性可以设置 native 使 Hibernate 可以使用 identity, sequence 或 hilo 算法根据底层数据库的情况来创建主键。
- <property> 标签用来将 Java 类的属性与数据库表的列匹配。标签中 name 属性引用的是类的性质，column 属性引用的是数据库表的列。type 属性保存 Hibernate 映射的类型，这个类型会将从 Java 转换成 SQL 数据类型。

## 映射类型
当你准备一个 Hibernate 映射文件时，我们已经看到你把 Java 数据类型映射到了 RDBMS 数据格式。在映射文件中已经声明被使用的 types 不是 Java 数据类型；它们也不是 SQL 数据库类型。这种类型被称为 Hibernate 映射类型，可以从 Java 翻译成 SQL，反之亦然。

见<a href="http://wiki.jikexueyuan.com/project/hibernate/mapping-types.html" target="_blank">Hibernate映射类型参考</a>


## Video Tutorial
这是一个<a href="https://www.youtube.com/playlist?list=PLaYqF7AnyNPeCFgvptK0TNKxsAIEUn71j" target="_blank">Hibernate的视频教学Website。</a>

Refernece：<a href="http://wiki.jikexueyuan.com/project/hibernate/" target="_blank">理论介绍</a>
