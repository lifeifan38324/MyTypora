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
        
                

#### 3.2.2. 基于注解方式创建对象



















# 三.`Aop`

# 四.`JdbcTemplate`

# 五.事务管理

# 

