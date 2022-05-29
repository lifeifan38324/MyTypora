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
1. 在spring配置文件中，使用bean标签，标签里面添加对应的属性，可以实现对象的创建
    - id属性：唯一表示
    - class属性：类的全路径（src为根路径）
    - 创建时默认调用<span style="color:blue">无参数</span>>构造方法
    
2. 注入属性，DI：依赖注入

    - 方式一：set方法注入

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

#### 3.2.3Bean的生命周期

1. 通过构造器创建Bean实例（无参数构造器）

2. 为Bean的属性设置值（调用set方法）

3. <span style="color:gray">把bean实例传递bean后置处理器的方法</span>

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

    - @Component
    - @Service
    - Controller
    - Repository

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

    - @AutoWired：根据属性类型自动注入

    - @Qualifier：根据属性名称注入

        需要与@AutoWired一起使用，用于指定具体实现类的名称

        ```java
        @Autowired
        @Qualifier(value = "userDaoImpl")
        private UserDao userDao;
        ```

        

    - @Resource：根据属性类型or属性名称注入

        根据类型：`@Resource`

        根据名称：`@Resource(name = "userDaoImpl")`

    - @Value：注入普通类型的属性

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

    



















# 三.`Aop`

# 四.`JdbcTemplate`

# 五.事务管理

# 

