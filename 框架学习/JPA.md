# 一. 使用IDEA创建JPA工程，HelloWorld

[参考视频](https://www.bilibili.com/video/BV1Z64y1o7ap?p=10&spm_id_from=pageDriver&vd_source=d5037450fe460e4ece53c0a79dd0ca5f)

## 1. 创建普通的Maven项目

Archetype：`org.apache.maven.archetypes:maven-archetype-quickstart`

## 2. 导入依赖包

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <project.hibernate.version>5.0.7.Final</project.hibernate.version>
</properties>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    <!-- hibernate对jpa的支持包 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>${project.hibernate.version}</version>
    </dependency>
    <!-- c3p0 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-c3p0</artifactId>
        <version>${project.hibernate.version}</version>
    </dependency>
    <!-- log日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <!-- Mysql and MariaDB -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.16</version>
    </dependency>
</dependencies>
```

## 3. 创建并配置核心配置文件(可以从ProjectStructure配置)

- 位置：`main/resources/META-INF`的文件夹下
- 名称：`persistence.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!--配置持久化单元 
        name：持久化单元名称 
        transaction-type：事务类型
             RESOURCE_LOCAL：本地事务管理 
             JTA：分布式事务管理 -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--配置JPA规范的服务提供商
 			实际上为javax.persistence.spi.PersistenceProvider的实现类-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <!-- 数据库驱动 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver" />
            <!-- 数据库地址 -->
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/ssh" />
            <!-- 数据库用户名 -->
            <property name="javax.persistence.jdbc.user" value="root" />
            <!-- 数据库密码 -->
            <property name="javax.persistence.jdbc.password" value="111111" />

            <!--jpa提供者的可选配置：我们的JPA规范的提供者为hibernate，所以jpa的核心配置中兼容hibernate的配 -->
            <!-- 显示sql : false|true -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <!-- 自动创建数据库表 : hibernate.hbm2ddl.auto
                          create : 程序运行时创建数据库表（如果有表，先删除表再创建）
                          update : 程序运行时创建表（如果有表，不会创建表）
                            none : 不会创建表-->
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```

## 4. 创建持久化类

```java
@Table(name = "JPA_CUSTOMER")
@Entity
public class Customer {
    private Integer id;
    private String lastName;
    private String email;
    private int age;

    @GeneratedValue(strategy = GenerationType.AUTO)
    @Id
    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    @Column(name = "LAST_NAME")
    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

## 5.  注册持久化类

```xml
<persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
    <!--持久化类-->
    <class>com.atlff.jpa.helloworld.Customer</class>
</persistence-unit>
```

## 6. 测试

```java
public class Main {
    public static void main(String[] args) {
        // 1.创建EntityManagerFactory对象
        String persistenceUnitName = "myJpa";
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory(persistenceUnitName);
        // 2.创建EntityManager对象
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        // 3.开启事务
        EntityTransaction entityTransaction = entityManager.getTransaction();
        entityTransaction.begin();
        // 4.执行持久化操作
        Customer customer = new Customer();
        customer.setLastName("lff");
        customer.setEmail("sfda@qq.com");
        customer.setAge(12);
        entityManager.persist(customer);
        // 5.提交事务
        entityTransaction.commit();
        // 6.关闭EntityManager
        entityManager.close();
        // 7.关闭创建EntityManagerFactory对象
        entityManagerFactory.close();
    }
}
```

# 二. 基本注解

## 1. 类-@Entity

指出该Java 类为实体类，将映射到指定的数据库表。

## 2. 类-@Table

- **当实体类与其映射的数据库表名不同名时**需要使用 **@Table** 标注说明，该标注与 **@Entity标注并列使用**，置于实体类声明语句之前，可写于单独语句行，也可与声明语句同行。
- @Table 标注的常用选项是 **name**，用于指明数据库的表名
- @Table标注还有一个两个选项 catalog 和 schema 用于设置表所属的数据库目录或模式，通常为数据库名。uniqueConstraints 选项用于设置约束条件，通常不须设置。

## 3. get方法-@Id

- @Id 标注用于声明一个实体类的属性映射为数据库的**主键列**。该属性通常置于属性声明语句之前，可与声明语句同行，也可写在单独行上。
- @Id标注也可置于属性的**getter**方法之前。

## 4. get方法-@GeneratedValue

- @GeneratedValue  用于标注主键的生成策略，通过 strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略：SqlServer 对应 identity，MySQL 对应 auto increment。
- 在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略：

----IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式；
----AUTO： JPA自动选择合适的策略，是默认选项；
----SEQUENCE：通过序列产生主键，通过 @SequenceGenerator 注解指定序列名，MySql 不支持这种方式
----TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。

## 5. get方法-@Basic

- @Basic 表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的 getXxxx() 方法,默认即为@Basic
- fetch: 表示该属性的读取策略,有 EAGER 和 LAZY 两种,分别表示主支抓取和延迟加载,默认为EAGER.
- optional:表示该属性是否允许为null, 默认为true

## 6. get方法-@Column

- 当实体的属性与其映射的数据库表的列不同名时需要使用@Column 标注说明，该属性通常置于实体的属性声明语句之前，还可与 @Id 标注一起使用。
- @Column 标注的常用属性是 name，用于设置映射数据库表的列名。此外，该标注还包含其它多个属性，如：unique 、nullable、length 等。
- @Column 标注的 columnDefinition 属性: 表示该字段在数据库中的实际类型.通常 ORM 框架可以根据属性类型自动判断数据库中字段的类型,但是对于Date类型仍无法确定数据库中字段类型究竟是DATE,TIME还是TIMESTAMP.此外,String的默认映射类型为VARCHAR, 如果要将 String 类型映射到特定数据库的 BLOB 或TEXT 字段类型.
- @Column标注也可置于属性的getter方法之前

## 7. get方法-@Transient

- 表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性.
- 如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic

## 8. get方法-@Temporal

- 在核心的 Java API 中并没有定义 Date 类型的精度(temporal precision).  而在数据库中,表示 Date 类型的数据有 DATE, TIME, 和 TIMESTAMP 三种精度(即单纯的日期,时间,或者两者 兼备). 在进行属性映射时可使用@Temporal注解来调整精度.

## 9. 用table来生成主键

![image-20220615225608583](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206152256714.png)

---

![image-20220615225725023](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206152257063.png)

# 三. 相关的API

## 1. Persistence

- createEntityManagerFactory 的 静态方法。用于获取 EntityManagerFactory 实例。

## 2. EntityManagerFactory

- createEntityManager()：用于创建实体管理器对象实例。
- close()：关闭 EntityManagerFactory 。 

## 3. EntityManager

**实体的状态:**
----<span style="color:blue;font-weight:bold">新建状态:  </span> 新创建的对象，<span style="color:red">尚未拥有持久性主键</span>。
----<span style="color:blue;font-weight:bold">持久化状态：</span>已经拥有持久性主键并和<span style="color:red">持久化建立了上下文环境M</span>
----<span style="color:blue;font-weight:bold">游离状态：</span><span style="color:red">拥有持久化主键，但是没有与持久化建立上下文环境</span>
----<span style="color:blue;font-weight:bold">删除状态: </span> 拥有持久化主键，已经和持久化建立上下文环境，<span style="color:red">但是从数据库中删除</span>。

- `find (Class<T> entityClass,Object primaryKey)`：返回指定的 OID 对应的实体类对象，如果这个实体存在于当前的持久化环境，则返回一个被缓存的对象；否则会创建一个新的 Entity, 并加载数据库中相关信息；若 OID 不存在于数据库中，则返回一个 null。第一个参数为被查询的实体类类型，第二个参数为待查找实体的主键值。

- `getReference (Class<T> entityClass,Object primaryKey)`：与find()方法类似，不同的是：如果缓存中不存在指定的 Entity, EntityManager 会创建一个 Entity 类的代理，但是不会立即加载数据库中的信息，**只有第一次真正使用此 Entity 的属性才加载**，所以如果此 OID 在数据库不存在，getReference() 不会返回 null 值, 而是抛出EntityNotFoundException

- `persist (Object entity)`：用于将新创建的 Entity 纳入到 EntityManager 的管理。该方法执行后，传入 persist() 方法的 Entity 对象转换成持久化状态。
    ----如果传入 persist() 方法的 Entity 对象已经处于持久化状态，则 persist() 方法什么都不做。
    ----如果对删除状态的 Entity 进行 persist() 操作，会转换为持久化状态。
    ----如果对游离状态的实体执行 persist() 操作，可能会在 persist() 方法抛出 EntityExistException(也有可能是在flush或事务提交后抛出)。
    
- `remove (Object entity)`：删除实例。如果实例是被管理的，即与数据库实体记录关联，则同时会删除关联的数据库记录。

    ----注意只能移除持久化对象，无法移除游离化对象。

- `merge (T entity)`：merge() 用于处理 Entity 的同步。即数据库的插入和更新操作。

![image-20220616075218770](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206160752377.png)

- `flush ()`：同步持久上下文环境，即将持久上下文环境的所有未保存实体的状态信息保存到数据库中。

## 4. EntityTransaction

- begin (): 用于启动一个事务，此后的多个数据库操作将作为整体被提交或撤消。若这时事务已启动则会抛出 IllegalStateException 异常。

- commit (): 用于提交当前事务。即将事务启动以后的所有数据库更新操作持久化至数据库中。
- rollback (): 撤消(回滚)当前事务。即撤消事务启动后的所有数据库更新操作，从而不对数据库产生影响。

# 四. 映射关系关联

![image-20220616094834261](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206160948923.png)

## 1. 映射单向<u>多对一</u>的关联关系

- 在 many 方指定 @ManyToOne 注释，并使用 @JoinColumn 指定外键名称。

```java
@Table(name = "JPA_ORDERS")
@Entity
public class Order {
	@JoinColumn(name = "CUSTOMER_ID")
    @ManyToOne([targetEntity=Customer.class],[fetch=FetchType.LAZY])//默认为预先加载
    public Customer getCustomer() {
        return customer;
    }
}
```

## 2. 映射单向<u>一对多</u>的关联关系

- 可以在 one 方指定 @OneToMany 注释，并使用 @JoinColumn 指定外键名称。

```java
@Table(name = "JPA_CUSTOMER")
@Entity
public class Customer {
	@JoinColumn(name = "CUSTOMER_ID")
    @OneToMany(fetch = FetchType.EAGER,cascade = {CascadeType.REMOVE})//fetch默认为饿加载,删除默认为外键制空
    public Set<Order> getOrders() {
        return orders;
    }
}
```

## 3. 映射双向多对一的关联关系

- 双向一对多关系中，必须存在一个关系维护端，在 JPA 规范中，要求  many 的一方作为关系的维护端(owner side), one 的一方作为被维护端(inverse side)。
- 可以在 one 方指定 @OneToMany 注释并<span style="color:red">设置 mappedBy 属性，以指定它是这一关联中的被维护端，many 为维护端</span>。
- 在 many 方指定 @ManyToOne 注释，并使用 @JoinColumn 指定外键名称。

```java
@Table(name = "JPA_CUSTOMER")
@Entity
public class Customer {
    。。。
    @JoinColumn(name = "CUSTOMER_ID")//使用mappedBy属性时,需要删除改注释
	@OneToMany(fetch = FetchType.EAGER,cascade = {CascadeType.REMOVE},mappedBy = "customer")//fetch默认为饿加载,删除时默认为外键制空。mappedBy放弃维护关系
    public Set<Order> getOrders() {
        return orders;
    }
}

@Table(name = "JPA_ORDERS")
@Entity
public class Order {
    。。。
    @JoinColumn(name = "CUSTOMER_ID")
    @ManyToOne
    public Customer getCustomer() {
        return customer;
    }
}
```

这里有问题：外键为NULL

![image-20220616111750669](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206161117844.png)

原因：文中设置属性时，设置的为One的Many属性。改为设置Many的One属性即可。

## 4. 双向一对一映射

- 基于外键的 1-1 关联关系：在双向的一对一关联中，需要在关系被维护端(inverse side)中的 @OneToOne 注释中指定 mappedBy，以指定是这一关联中的被维护端。同时需要在关系维护端(owner side)建立外键列指向关系被维护端的主键列。

```java
@Table(name = "JPA_DEPARTMENTS")
@Entity
public class Department {
    ...
    @JoinColumn(name = "MGR_ID",unique = true)
    @OneToOne
    public Manager getMgr() {
        return mgr;
    }
}


@Table(name = "JPA_MANAGERS")
@Entity
public class Manager {
    ...
    @OneToOne(mappedBy = "mgr")
    public Department getDept() {
        return dept;
    }
}
```

### 插入：双向 1-1 不延迟加载的问题

- `find(`): 默认情况下，若获取**维护或者不维护**关联关系的一方，则会通过左外连接获取其关联的对象。但可以通过修改`@OneToOne`的fetch属性来修改加载策略。

----修改维护方的`@OneToOne`的fetch属性为`fetch = FetchType.LAZY`，可以使用一条不含外连接的语句加载一个完整的维护对象。此时被维护对象为代理类。

----修改被维护方的`@OneToOne`的fetch属性`fetch = FetchType.LAZY`，则变为两条加载一个完整的被维护对象。（应该避免此情况出现）

![image-20220616153928886](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206161539997.png)

## 5. 映射双向多对多的关联关系

```java
@Table(name = "JPA_ITEMS")
@Entity
public class Item {
    @JoinTable(name = "ITEM_CATEGORY",
            joinColumns={@JoinColumn(name = "ITEM_ID",referencedColumnName = "ID")},
            inverseJoinColumns = {@JoinColumn(name = "CATEGORY_ID",referencedColumnName = "ID")})
    @ManyToMany
    public Set<Category> getCategories() {
        return categories;
    }
}

@Table(name = "JPA_CATEGORYS")
@Entity
public class Category {
    @ManyToMany(mappedBy = "categories")
    public Set<Item> getItems() {
        return items;
    }
}
```

![image-20220616160832523](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206161608509.png)

- `find()`: 默认使用懒加载的策略，即调用关联对象时，再执行一次查询。

# 五. 二级缓存

**可以使得`EntityManager`跨越事务使用。**

## 1. 导入依赖

参考文章：

[普通工程](http://t.zoukankan.com/liguangsunls-p-7040421.html)

[MAVEN工程](https://www.iteye.com/blog/tangzongyun-2386963)

![image-20220616171336097](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206161713256.png)

```xml
<!--Ehcache-core 包 -->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.9</version>
</dependency>
<!--添加Hibernate-Ehcache包, 注意与上文的版本一致 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>${project.hibernate.version}</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-nop -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>
```

## 2. 配置文件

`persistence.xml`

```xml
<!-- 二级缓存相关 -->
<property name="hibernate.cache.use_second_level_cache" value="true"/>
<property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
<!--开启查询缓存-->
<property name="hibernate.cache.use_query_cache" value="true"/>
```

在src下增加一个配置文件：`ehcache.xml`

```xml
<ehcache>

    <!-- Sets the path to the directory where cache .data files are created.

         If the path is a Java System Property it is replaced by
         its value in the running VM.

         The following properties are translated:
         user.home - User's home directory
         user.dir - User's current working directory
         java.io.tmpdir - Default temp file path -->
    <diskStore path="java.io.tmpdir"/>


    <!--Default Cache configuration. These will applied to caches programmatically created through
        the CacheManager.

        The following attributes are required for defaultCache:

        maxInMemory       - Sets the maximum number of objects that will be created in memory
        eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                            is never expired.
        timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                            if the element is not eternal. Idle time is now - last accessed time
        timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                            if the element is not eternal. TTL is now - creation time
        overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                            has reached the maxInMemory limit.

        -->
    <defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true"
        />

    <!--Predefined caches.  Add your cache configuration settings here.
        If you do not have a configuration for your cache a WARNING will be issued when the
        CacheManager starts

        The following attributes are required for defaultCache:

        name              - Sets the name of the cache. This is used to identify the cache. It must be unique.
        maxInMemory       - Sets the maximum number of objects that will be created in memory
        eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                            is never expired.
        timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                            if the element is not eternal. Idle time is now - last accessed time
        timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                            if the element is not eternal. TTL is now - creation time
        overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                            has reached the maxInMemory limit.

        -->

    <!-- Sample cache named sampleCache1
        This cache contains a maximum in memory of 10000 elements, and will expire
        an element if it is idle for more than 5 minutes and lives for more than
        10 minutes.

        If there are more than 10000 elements it will overflow to the
        disk cache, which in this configuration will go to wherever java.io.tmp is
        defined on your system. On a standard Linux system this will be /tmp"
        -->
    <cache name="sampleCache1"
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="300"
        timeToLiveSeconds="600"
        overflowToDisk="true"
        />

    <!-- Sample cache named sampleCache2
        This cache contains 1000 elements. Elements will always be held in memory.
        They are not expired. -->
    <cache name="sampleCache2"
        maxElementsInMemory="1000"
        eternal="true"
        timeToIdleSeconds="0"
        timeToLiveSeconds="0"
        overflowToDisk="false"
        /> -->

    <!-- Place configuration for your caches following -->

</ehcache>
```

## 3. 启动缓存

再需要开启二级缓存的类添加注解：`@Cacheable(true)`

## 4. 测试

```java
@Test
public void testSecondLevalCache(){
    Customer customer1 = entityManager.find(Customer.class, 1);

    entityManager.close();
    entityManager = entityManagerFactory.createEntityManager();

    Customer customer2 = entityManager.find(Customer.class, 1);
}
```

# 六. JPQL

Java Persistence Query Language 的简称。

JPQL语言的语句可以是 <span style="color:blue">select 语句、update 语句或delete语句</span>，它们都通过<span style="color:red">Query 接口</span>封装执行。

```
Query接口的主要方法
    ++ int executeUpdate()
    用于执行update或delete语句。
    ++ List getResultList()
    用于执行select语句并返回结果集实体列表。
    ++ Object getSingleResult()
    用于执行只返回单个结果实体的select语句。
    ++ Query setFirstResult(int startPosition)
    用于设置从哪个实体记录开始返回查询结果。
    ++ Query setMaxResults(int maxResult) 
    用于设置返回结果实体的最大数。与setFirstResult结合使用可实现分页查询。
    ++ Query setFlushMode(FlushModeType flushMode) 
    设置查询对象的Flush模式。参数可以取2个枚举值：FlushModeType.AUTO 为自动更新数据库记录，FlushMode Type.COMMIT 为直到提交事务时才更新数据库记录。
    ++ setHint(String hintName, Object value) 
    设置与查询对象相关的特定供应商参数或提示信息。参数名及其取值需要参考特定 JPA 实现库提供商的文档。如果第二个参数无效将抛出IllegalArgumentException异常。
    ++ setParameter(int position, Object value) 
    为查询语句的指定位置参数赋值。Position 指定参数序号，value 为赋给参数的值。
    ++ setParameter(int position, Date d, TemporalType type) 
    为查询语句的指定位置参数赋 Date 值。Position 指定参数序号，value 为赋给参数的值，temporalType 取 TemporalType 的枚举常量，包括 DATE、TIME 及 TIMESTAMP 三个，，用于将 Java 的 Date 型值临时转换为数据库支持的日期时间类型（java.sql.Date、java.sql.Time及java.sql.Timestamp）。
    ++ setParameter(int position, Calendar c, TemporalType type) 
    为查询语句的指定位置参数赋 Calenda r值。position 指定参数序号，value 为赋给参数的值，temporalType 的含义及取舍同前。
    ++ setParameter(String name, Object value) 
    为查询语句的指定名称参数赋值。
    ++ setParameter(String name, Date d, TemporalType type) 
    为查询语句的指定名称参数赋 Date 值。用法同前。
    ++ setParameter(String name, Calendar c, TemporalType type) 
    为查询语句的指定名称参数设置Calendar值。name为参数名，其它同前。该方法调用时如果参数位置或参数名不正确，或者所赋的参数值类型不匹配，将抛出 IllegalArgumentException 异常。
```

## 1. 获取Query

Query接口封装了执行数据库查询的相关方法。调用 EntityManager 的 createQuery、create NamedQuery 及 createNativeQuery 方法可以获得查询对象。

- 使用createQuery

```java
public void testHelloJPAL(){
    String jpql = "FROM Customer c WHERE c.lastName like ?";
    Query query = entityManager.createQuery(jpql);
    query.setParameter(1,"BB");
    List<Customer> customers = query.getResultList();
    System.out.println(customers);
}
```

- create NamedQuery

```java
@NamedQuery(name="testNamedQuery", query = "select c from Customer c where c.id = ?")
@Cacheable(true)
@Table(name = "JPA_CUSTOMER")
@Entity
public class Customer {
	。。。
}

@Test
public void testNamedQuery(){
    Query testNamedQuery = entityManager.createNamedQuery("testNamedQuery").setParameter(1, 1);
    Customer customer = (Customer) testNamedQuery.getSingleResult();
    System.out.println(customer);
}
```

- createNativeQuery

```java
@Test
public void testNativeQuery(){
    String sql = "select LAST_NAME from jpa_customer where id = ?";
    Query query = entityManager.createNativeQuery(sql);
    query = query.setParameter(1, 2);
    Object result = query.getSingleResult();
    System.out.println("result = " + result);
}
```

## 2. 设置查询缓存

首先要在配置文件里开启查询缓存

```xml
<!--开启查询缓存-->
<property name="hibernate.cache.use_query_cache" value="true"/>
```

调用：

`.setHint(QueryHints.CACHEABLE,true)`

```java
@Test
public void testSerchCache(){
    String jpql = "select c from Customer c where c.id = ?";
    Query query1 = entityManager.createQuery(jpql).setParameter(1,2).setHint(QueryHints.CACHEABLE,true);
    List list1 = query1.getResultList();

    Query query2 = entityManager.createQuery(jpql).setParameter(1,2).setHint(QueryHints.CACHEABLE,true);
    List list2 = query1.getResultList();
}
```

## 3. Order By 和 Group By

直接加在jpql语句中。

## 4. 关联查询

```java
@Test
public void testLfetOuterJoinFetch(){
    String jpql = "from Customer c LEFT OUTER JOIN FETCH c.orders WHERE c.id = ?";
    Customer customer = (Customer)entityManager.createQuery(jpql).setParameter(1, 2).getSingleResult();
    System.out.println(customer);
    System.out.println(customer.getOrders().size());
}
```

## 5. 子查询

```java
@Test
public void testSubSearch(){
    String jpql = "SELECT o FROM Order o " +
        "WHERE o.customer = (SELECT c FROM Customer c WHERE c.lastName = ?)";
    List<Order> list = entityManager.createQuery(jpql).setParameter(1, "BB").getResultList();
    System.out.println("list = " + list);
}
```

疑问：为什么有两条查询语句

![image-20220617090322118](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206170903486.png)

# 七. 框架整合（Eclipse非Maven）

[尚硅谷JPA](https://www.bilibili.com/video/BV1vW411M7zp?p=24&spm_id_from=pageDriver&vd_source=d5037450fe460e4ece53c0a79dd0ca5f)
