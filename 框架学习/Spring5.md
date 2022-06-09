[TOC]

# 一.`Spring5`的下载与安装

1. 打开官网：`spring.io`，找到`SpringFramework`查看版本（GA为稳定版），点击`GitHub`

2. Access to Binaries - Spring Framework Artifacts --> Downloading a Distribution - https://repo.spring.io --> Artifacts - release - `org` - `springframework` - spring 

3. 在`IDEA`中导入`Spring5`相关`jar包`

4. 框架结构

    ![](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/Spring5%E6%A8%A1%E5%9D%97.bmp)

# 二.`IOC`容器

## 1.目的

解耦，减低类与类之间的耦合度。

解耦方法：工厂模式，`IOC容器`

## 2.入门使用方法

1. `xml`配置文件，配置对象

    ```xml
    <!--配置User类的创建-->
    <bean id="user" class="com.atlff.spring5.User"></bean>
    ```

2. 调用配置对象

    ```java
    //1.加载spring配置对象
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    //2.获取配置创建的对象
    User user =  context.getBean("user", User.class);
    ```

## 3.知识点了解

### 3.1`IOC接口`

`BeanFactory`和`ApplicationContext`

1. `BeanFactory`：`IOC`容器的基本实现，`spring`内部接口。不建议开发人员使用
    - 加载配置文件时不创建对象，获取对象时才创建对象
2. `ApplicationContext`：`BeanFactory`的子接口，功能更加强大。<span style="color:blue;font-weight:bold">建议开发人员使用</span>
    - 加载配置文件时，立即创建对象

ps: F4 可以查看继承树结构（康师傅快捷键）

### 3.2`Bean`管理

指的是<span style="color:blue">创建对象</span>和<span style="color:blue">注入对象</span>的属性

#### 3.2.1. 基于`xml`方式创建对象

依赖包：导入基本的核心依赖包以及logging即可

![image-20220602081532501](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206020815987.png)

1. 在spring配置文件中，使用bean标签，标签里面添加对应的属性，可以实现对象的创建
    - id属性：唯一表示
    - class属性：类的全路径（src为根路径）
    - 创建时默认调用<span style="color:blue">无参数</span>>构造方法
    
2. 注入属性，DI：依赖注入

    - 方式一：**set方法注入**

        1. 在类中添加set方法
        2. 在配置文件中，设置`Bean类`的`property属性(name,value)`
        
    - 方式二：有参构造器注入
    
      1.  在类中创建含参数的构造方法
      2. 在配置文件中，设置`Bean类`的`constructor-arg属性(name,value)`
      
    - 属性注入的注意：
        1. 字面量
            - 空值：不使用value，使用\<null/\>标签
            - 特殊符号：
                1. 转义。只用\&lt;和\&gt;代替<>
                2. 使用CADATA：`<value><![CDATA[<<中国>>]]></value>`
            
        2. 注入对象
        
            - 在类中，创建set方法
        
            - 在配置文件中，设置`Bean类`的\<property\>属性(name,<span style="color:red;">ref</span>)
        
            - 方式二(级联赋值)：可以使用内部bean
        
                ```xml
                <property name="dept">
                    <bean id="dept" class="com.atlff.spring5.bean.Dept">
                        <property name="dname" value="后端"></property>
                    </bean>
                </property>
                ```
        
        3. 注入集合
        
            - 数组类型属性
        
                创建类及属性----生成set()方法----配置文件
        
                ```xml
                <property name="courses">
                    <array>
                        <value>java</value>
                        <value>spring</value>
                        <value>lff</value>
                    </array>
                </property>
                ```
        
            - List集合类型属性
        
                ```xml
                <property name="list">
                    <list>
                        <value>李非凡</value>
                        <value>王林林</value>
                        <value>李二帆</value>
                    </list>
                </property>
                ```
        
            - Map集合类型属性
        
                ```xml
                <property name="maps">
                    <map>
                        <entry key="李非凡" value="帅"></entry>
                        <entry key="王林林" value="美"></entry>
                    </map>
                </property>
                ```
        
            - Set集合类型属性
        
                ```xml
                <property name="sets">
                    <set>
                        <value>李非凡</value>
                        <value>王林林</value>
                    </set>
                </property>
                ```

        4. 注入特殊集合
        
            - 类属性
        
                ```xml
                <property name="courseList">
                    <list>
                        <ref bean="course1"></ref>
                        <ref bean="course2"></ref>
                    </list>
                </property>
                ```
        
            - 抽取集合公共部分
        
                1). 在spring名称空间引入util
        
                ```xml
                <beans xmlns="http://www.springframework.org/schema/beans"
                       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                       xmlns:util="http://www.springframework.org/schema/util"
                       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
                ```
        
                2). 使用util标签完成list集合注入提取
        
                ```xml
                <!-- 1.提取集合类型属性 -->
                <util:list id="bookList">
                    <value>上下五千年</value>
                    <value>细菌战</value>
                    <value>克隆人</value>
                </util:list>
                <!-- 2.属性注入 -->
                <bean id="book" class="com.atlff.spring5.collectiontype.Book">
                    <property name="list" ref="bookList"></property>
                </bean>
                ```
        
        5. FactoryBean的使用
        
            返回类型与注入类型可以不同
        
            - 使调用的类继承`FactoryBean`接口
            - 重写`getObject()`方法

#### 3.2.2 Bean的作用域

1. 指的是可以设置Bean实例时单实例还是多实例，默认是单实例.
2. 通过配置文件的`scope`属性设置作用域：
    - singleton（单例）：加载配置文件时创建对象
    - prototype（多实例）：调用getBean方法时创建对象

#### 3.2.3 Bean的生命周期

1. 通过构造器创建Bean实例（无参数构造器）

2. 为Bean的属性设置值（调用set方法）

3. <span style="color:gray">把bean实例传递bean前置处理器的方法</span>

    添加后置处理器：

    - 创建类，实现接口`BeanPostProcessor`，创建后置处理器，重写方法

        ```java
        public class MyBeanPost implements BeanPostProcessor {
            @Override
            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("初始化之前执行的方法被调用。。。");
                return bean;
            }
            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("初始化之后执行的方法被调用。。。");
                return bean;
            }
        }
        ```

    - 配置文件配置后置处理器：

        ```xml
        <bean id="myBeanPost" class="com.atlff.spring5.bean.MyBeanPost"></bean>
        ```

        

3. 调用bean的初始化方法（需要进行配置初始化的方法）

    - 在`bean类`中：编写初始化方法

    - 配置文件中：使用bean标签的`init-method属性`

        ```xml
        <bean id="orders" class="com.atlff.spring5.bean.Orders" init-method="initMethod"></bean>
        ```

3. <span style="color:gray">把bean实例传递bean后置处理器的方法</span>

4. bean可以使用了

5. 当关闭容器的时候，调用bean的销毁的方法（需要进行配置销毁的方法）

    - 在`bean类`中：编写销毁方法

    - 配置文件中：使用bean标签的`destroy-method属性`

        ```xml
        <bean id="orders" class="com.atlff.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod"></bean>
        ```
        
    - 销毁`bean`：`((ClassPathXmlApplicationContext) context).close();`
    
    - 注意：多实例作用域的类，不执行销毁方法。

#### 3.2.4 自动注入属性(不建议使用)

配置文件bean标签属性`autowrite="byName"`或者`autowrite="byType"`

```xml
<bean id="emp" class="com.atlff.spring5.autowrite.Emp" autowire="byName"></bean>
```

#### 3.2.5 引入外部属性文件

原始方法：xml配置数据库连接池

```xml
<bean id="dateSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value=""></property>
    <property name="url" value=""></property>
    <property name="username" value=""></property>
    <property name="password" value=""></property>
</bean>
```

使用外部配置文件改进方法：引入外部属性文件配置数据库连接池

1. 编写`jdbc.properties`配置文件

    ```
    prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/bookdb
    prop.userName=
    prop.password=
    ```

2. 引入context命名空间

    ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                               http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    </beans>
    ```

3. 引入外部属性文件

    ```xml
    <context:property-placeholder location="jdbc.properties" />
    <bean id="dateSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
    ```

----



#### 3.2.6 基于注解方式创建对象

1. 创建对象的四个基本注解

    - **@Component**
    - **@Service**
    - **@Controller**
    - **@Repository**

2. 使用注解创建对象的基本步骤

    - 导入`spring-aop`的工具包

        ![image-20220529192136090](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220529192136090.png)

    - 引入名称空间

        ```xml
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
            <context:component-scan base-package="com.atlff.spring5"></context:component-scan></beans>
        ```

    - 开启组件扫描

        ```xml
        <context:component-scan base-package="com.atlff.spring5"></context:component-scan>
        ```

    - 使用注解创建对象

        ```java
        @Component(value = "userService")
        ```

3. 组件扫描的细节问题

    - 默认为所有注解都进行扫描

    - 只扫描`Controller`注解

        ```xml
        <context:component-scan base-package="com.atguigu.springmvc" use-default-filters="false">
        	<context:include-filter type="annotation"
                   expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>
        ```

        

    - 除了Controller注解，其余全部扫描

        ```xml
        <context:component-scan base-package="com.atguigu.springmvc">
        	<context:exclude-filter type="annotation"
                	expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>
        ```

#### 3.2.7 基于注解方式注入属性

1. 四种基本注解

    - **@AutoWired**：根据属性类型自动注入

    - **@Qualifier**：根据属性名称注入

        需要与@AutoWired一起使用，用于指定具体实现类的名称

        ```java
        @Autowired
        @Qualifier(value = "userDaoImpl")
        private UserDao userDao;
        ```

    - **@Resource**：根据属性类型or属性名称注入

        根据类型：`@Resource`

        根据名称：`@Resource(name = "userDaoImpl")`

    - **@Value**：注入普通类型的属性

2. 使用方法

    参考[3.2.6节](#3.2.6 基于注解方式创建对象)

#### 3.2.8 完全注解开发

1. 创建配置类，替代xml配置文件

    ```java
    @Configuration //作为配置类，替换xml文件
    @ComponentScan(basePackages = {"com.atlff.spring5"})
    public class SpringConfig {
    }
    ```

2. 加载配置类

    ```java
    public void test(){
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
        UserService userService = context.getBean("userService", UserService.class);
        userService.add();
    }
    ```

    

# 三. `Aop`(Aspect Oriented Programming)

## 1. 简介

1. 目的

    面向切面编程：利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性

2. 术语

    - 连接点：可以被增强的方法
    - 切入点：真正被增强的方法
    - 通知（增强）
        - 实际增强的逻辑部分称为通知（增强）
        - 类型：
            1. 前置通知
            2. 后置通知
            3. 环绕通知
            4. 异常通知
            5. 最终通知
    - 切面：把通知应用到切入点的过程



## 2. 底层原理：动态代理

### 2.1 有接口情况：使用JDK动态代理

1. 基本原理

    ![image-20220529230121546](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220529230121546.png)

2. 使用JDK的`java.lang.reflect.Proxy`类中的`newProxyInstance`方法

    `newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`

    - 参数1:类加载器
    - 参数2:被代理类实现的接口
    - 参数3:接口的实现类，实现需要增强的功能
    
3. 例子：

    ```java
    public class JDKProxy {
        public static void main(String[] args) {
            Class[] interfaces = {UserDao.class};
            UserDaoImpl userDao = new UserDaoImpl();
            UserDao dao = (UserDao) Proxy.newProxyInstance(UserDao.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
            String lff = dao.update("李非凡");
            System.out.println("lff = " + lff);
        }
    }
    class UserDaoProxy implements InvocationHandler{
        private Object object;
        public UserDaoProxy(Object object){
            this.object = object;
        }
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            String methodName = method.getName();
            System.out.println("方法" + methodName + "被调用之前的增强功能");
            Object res = method.invoke(object, args);
            System.out.println("方法" + methodName + "被调用之后的增强功能");
            return res;
        }
    }
    ```

### 2.2 没有接口情况：使用CGLIB动态代理

![image-20220529230202109](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220529230202109.png)

### 2.3 AOP操作

简介：Spring框架基于AspectJ实现AOP操作，AspectJ不是Spring的组成部分，但通常与Spring一起实现AOP操作

#### 2.3.1 导入AspectJ的包，以及其依赖包

![image-20220530092622862](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220530092622862.png)

![image-20220530092541523](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220530092541523.png)

#### 2.3.2 切入点表达式

语法结构：execution([权限修饰符] [返回类型] [类全路径] [方法名称] [参数列表])

- 举例1：对com.atlff.dao.BookDao类里面的add进行增强

    `execution(* com.atlff.dao.BookDao.add(..))`

- 举例2：对com.atlff.dao.BookDao类里面的所有方法进行增强

    `execution(* com.atlff.dao.BookDao.*(..))`

- 举例3：对com.atlff.dao包里面的所有类的所有方法进行增强

    `execution(* com.atlff.dao.*.*(..))`

#### 2.3.3 在配置文件中引入名称空间，开启注解扫描，开启Aspect生成代理对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 1.开启注解扫描 -->
    <context:component-scan base-package="com.atlff.spring5.aopanno"></context:component-scan>
    <!-- 2.开启Aspect生成代理对象  -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

#### 2.3.4 在类中使用注解

```java
//被代理的类
@Component
public class User {
    public void add(){
        System.out.println("add()...");
    }
}

// 代理类
@Component
@Aspect
public class UserProxy {
    //Before设置前置通知
    @Before(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
    public void before(){
        System.out.println("Before...");
    }

    //Before设置后置通知
    @AfterReturning(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
    public void afterReturning(){
        System.out.println("AfterReturning...");
    }

    //After设置最终通知
    @After(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
    public void after(){
        System.out.println("After...");
    }

    //AfterThrowing设置异常通知
    @AfterThrowing(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
    public void afterThrowing(){
        System.out.println("AfterThrowing...");
    }

    //Around设置环绕通知
    @Around(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("Around之前...");
        proceedingJoinPoint.proceed();
        System.out.println("Around之后...");
    }
}
```

#### 2.3.5 公共的切入点的抽取：使用Pointcut注解

```java
@Pointcut(value = "execution(* com.atlff.spring5.aopanno.User.add(..))")
public void pointcut(){}

//Before设置前置通知
@Before(value = "pointcut()")
public void before(){
    System.out.println("Before...");
}
```

#### 2.3.6 当有多个增强类增强同一个方法，设置增强类的优先级

`@Order(value = 3)`，值越小优先级越高

```java
@Component
@Aspect
@Order(value = 3)
public class PersonProxy {}
```

#### 2.3.7 完全注解开发

- 创建配置类，不需要xml文件

    ```java
    @Configuration
    @ComponentScan(basePackages = {"com.atlff"})
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public class Config {
    }
    ```

    

### 2.4 基于配置文件实现AOP操作

1. 新建类

2. 配置文件

    ```xml
    <!-- 1.创建对象 -->
    <bean id="book" class="com.atlff.spring5.aopxml.Book"></bean>
    <bean id="bookProxy" class="com.atlff.spring5.aopxml.BookProxy"></bean>
    <!-- 2.配置AOP的增强 -->
    <aop:config>
        <!-- 2.1切入点 -->
        <aop:pointcut id="p" expression="execution(* com.atlff.spring5.aopxml.Book.*(..))"/>
        <!-- 2.2切面-->
        <aop:aspect ref="bookProxy">
            <aop:before method="before" pointcut-ref="p"/>
        </aop:aspect>
    </aop:config>
    ```

<span style="color:red;font-weight:bold">注意：`before`与`around`的执行顺序看配置文件；而`after`总是再`around`之后执行</span>

# 四.`JdbcTemplate`

## 1. 简介

Spring5封装之后的JDBC

## 2. 准备工作

1. 导入jar包

    ![image-20220530122930818](C:/Users/LFF/AppData/Roaming/Typora/typora-user-images/image-20220530122930818.png)

    **注意：mysql8应该选择版本8的驱动**

2. 在xml配置文件中，配置数据库的连接池

   ```xml
   <!-- 数据库连接池 -->
   <bean id="dateSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
       <property name="url" value="jdbc:mysql://:3306/user_db"></property>
       <property name="username" value=""></property>
       <property name="password" value=""></property>
   </bean>
   ```
   
3. 配置JdbcTemplate对象，并注入dateSource

    ```xml
    <!-- JdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入dateSource -->
        <property name="dataSource" ref="dateSource"></property>
    </bean>
    ```

4. 创建Service类和Dao类，在Dao类中注入JdbcTemplate对象

    开启组件扫描

    ```xml
    <!-- 组件扫描 -->
    <context:component-scan base-package="com.atlff.spring5"></context:component-scan>
    ```

    - service

        ```java
        @Service
        public class BookService {
            @Autowired
            private BookDao bookDaoImpl;
            public int update(Book book){
                return bookDaoImpl.updateBook(book);
            }
        }
        ```

    - dao和daoImpl

        ```java
        public interface BookDao {
            public int updateBook(Book book);
        }
        
        @Repository
        public class BookDaoImpl implements BookDao{
            @Autowired
            private JdbcTemplate jdbcTemplate;
            @Override
            public int updateBook(Book book){
                String sql = "insert into t_user values(?,?,?)";
                Object[] args = {book.getId(), book.getName(), book.getStatus()};
                int update = jdbcTemplate.update(sql, args);
                return update;
            }
        }
        ```

    - entity：对象类以及对应的属性

## 3. 增删改查

- 增删改：update

- 查（返回某个值）：queryForObject

    `queryForObject(String sql, Class<T> requiredType)`

    ```java
    queryForObject(String sql, Integer.class)
    ```

- 查（返回某个对象）：

    `queryForObject(String sql, RowMapper<T> rowMapper, Object... args)`

    ```java
    queryForObject(sql, new BeanPropertyRowMapper<>(User.class), name);
    ```

    本质：调用set方法赋值

    注意查询语句的字段名要和entity中的字段名匹配。例如：

    ```sql
    select user_id as id, username as name, ustatus as status from t_user where user_id = ?
    ```

- 查（返回对象列表）：

    `query(sql, new BeanPropertyRowMapper<>(Book.class), Object ... args)`

    ```java
    query(sql, new BeanPropertyRowMapper<>(User.class), args)
    ```

- 批量增删改：

    ```java
    public int[] batchUpdate(String sql, List<Object[]> batchArgs)
    ```


# 五.事务管理

## 1. 简介

1. 定位：数据库操作最基本的逻辑单元
2. 特性（ACID）
    - 原子性：
    - 一致性：
    - 隔离性：
    - 持久性：

## 2. 事务操作

### 2.1 Spring事务管理API

 ![image-20220531211905875](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531211905875.png)

### 2.2 编程式事务管理

传统方法，try...catch...语句

### 2.3 声明式事务管理

底层使用AOP实现事务管理

#### 2.3.1 基于注解方式 

1. 配置文件配置事务管理器

    ```xml
    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dateSource"></property>
    </bean>
    ```

2. 开启事务注解

    1）引入名称空间tx

    ```xml
    xmlns:tx="http://www.springframework.org/schema/tx"
    
    xsi:schemaLocation="http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
    ```

    2）开启事务注解

    ```xml
    <!--开启事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
    ```

3. 在service类上面（或者service类里面的方法上面添加注解）

    事务注解：`@Transactional`

    - 类：所有方法添加事务
    - 方法：该方法实现事务

    ```java
    @Service
    @Transactional
    public class UserService{...}
    ```

4. 事务管理的参数配置

    ![image-20220531222004853](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531222004853.png)

    - **propagation**：事务的传播行为，默认为`REQUIRED`

        ![image-20220531223347686](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531223347686.png)

        ![事务传播行为](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/%E4%BA%8B%E5%8A%A1%E4%BC%A0%E6%92%AD%E8%A1%8C%E4%B8%BA.bmp)

    - **isolation**：事务的隔离级别

        **并发操作中的问题：**

        ![image-20220531231427590](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531231427590.png)

        - 脏读：一个未提交的事务读取到另一个**未提交**事务的数据

        - 不可重复读：一个未提交的事务读取到另一个提交事务的**修改**数据

        - 幻读：一个未提交的事务读取到另一个提交事务的**添加**数据

        **解决问题的方法，设置事务的隔离级别：**默认为`REPEATABLE READ`

        ![事务隔离级别](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB.bmp)
    
        
    
    - **timeout**：超时时间
    
        - 事务需要在一定时间内提交，如果不提交则进行回滚
    
        - 默认值为-1，设置时间以秒为单位进行计算
    
    - readOnly：是否只读
    
        - 读：查询操作，写：增删改操作
        - 默认值：false
    
    - rollbackFor：回滚
    
        - 设置出现那些异常进行事务回滚
    
    - noRrollbackFor：不回滚
    
        - 设置出现那些异常不进行事务回滚 

#### 2.3.2 基于xml配置文件方式

1. 配置事务管理器

    ```xml
    <!--创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入数据源-->
        <property name="dataSource" ref="dateSource"></property>
    </bean>
    ```

    

2. 配置通知

    ```xml
    <!--配置通知-->
    <tx:advice id="txadvice">
        <!--配置事务参数-->
        <tx:attributes>
            <!--指定那种规则的方法上添加事务-->
            <tx:method name="accountMoney" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    ```

    

3. 配置切入点和切面

    ```xml
    <!--配置切入点和切面-->
    <aop:config>
        <!--配置切入点-->
        <aop:pointcut id="pt" expression="execution(* com.atlff.spring5.service.UserService.*(..))"/>
        <!--配置切面-->
        <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
    </aop:config>
    ```

#### 2.3.3 完全注解声明式事务管理

创建配置类，使用配置类代替xml文件

```java
@Configuration //配置类
@ComponentScan(basePackages = "com.atlff.spring5") //开启组件扫描
@EnableTransactionManagement //开启事务
public class TxConfig {
    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("abc123");
        return dataSource;
    }
    //创建JdbcTemplate数据库链接模板
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        //到ioc容器中根据类型找到dataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }
    //创建事务管理器
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```



# 六. Spring5新特性

## 1. 基于java8

整个框架的代码基于java8，运行时兼容JDK9，许多不建议使用的类和方法在代码库中删除

## 2. 自带日志封装

Spring 5.0框架自带了通用的日志封装

- Spring5已经移除Log4ConfigListener，官方建议使用Log4j2
- Spring5框架整合Log4j2

1）导入jar包

![image-20220601170939805](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206011709145.png)

2）新建配置文件`log4j2.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，可以看到log4j2内部各种详细输出-->
<configuration status="INFO">
    <!--先定义所有的appender-->
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="Console" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </console>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <!--root：用于指定项目的根日志，如果没有单独指定Logger，则会使用root作为默认的日志输出-->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>
```

## 3. 支持@Nullable注解

可以用在类，方法上，方法参数，属性上，表示对应值可以为空。

## 4. 函数式风格

`GenericApplicationContext`/`AnnotationConfigApplicationContext`

例子：

```java
@Test
public void testGenericApplicationContext(){
    //1. 创建GenericApplicationContext对象
    GenericApplicationContext context = new GenericApplicationContext();
    //2. 调用context的方法注册对象
    context.refresh();
    context.registerBean("user",User.class, ()->new User());
    //3. 获取在spring中注册的对象
    User user = (User)context.getBean("user");
    System.out.println("user = " + user);
}
```

## 5. 整合JUnit5

### 5.1 同时兼容JUnit4

以下为<span style="color:red">整合JUnit4</span>的方法

1. 引入依赖包

    ![image-20220601190021120](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/202206011900238.png)

    和`JUnit4的依赖包`

2. 通过注解整合JUnit4

    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("classpath:bean.xml") //加载配置文件
    public class JTest4 {
        @Autowired
        private UserService userService;
    
        @Test
        public void test1(){
            userService.accountMoney();
        }
    }
    ```

### 5.2 Spring5整合JUnit5

1. 导入JUnit5的依赖包

    `JUnit5.8.1`

2. 在测试类中使用注解

    ```java
    //@ExtendWith(SpringExtension.class)
    //@ContextConfiguration("classpath:bean.xml") //加载配置文件
    
    @SpringJUnitConfig(locations = "classpath:bean.xml") //符合注解，替换上面两个注解
    public class JTest4 {
        @Autowired
        private UserService userService;
    
        @Test
        public void test1(){
            userService.accountMoney();
        }
    }
    ```

## 6. Webflux

<span style="color:red;font-weight:bold">前置知识：SpringMVC，SpringBoot，Maven，Java8中的新特性</span>

### 6.1 SpringWebflux介绍



### 6.2 响应式编程



### 6.3 Webflux执行流程 和 核心API



### 6.4 SpringWebflux（基于注解编程模型）



### 6.5 SpringWebflux（基于函数式编程模型）
