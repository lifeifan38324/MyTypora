[TOC]

# 一.业务需求分析

1. 用户登录
2. 登陆成功，显示主界面。左侧显示好友列表；上端显示欢迎词。如果不是自己空间，显示超链接：返回我主页
3. 查看日志详情：
    - 日志本身的信息（作者头像，昵称，日志标题，日志内容，日志的日期）
    - 回复列表（回复作者的头像，昵称，回复内容，回复日期）
    - 主人回复信息
4. 删除日志
5. 删除特定的回复
6. 删除特定主人的回复
7. 添加日志、添加回复、添加主人回复
8. 点击左侧好友，进入好友的空间

# 二.数据库的设计

1. 抽取实体：用户登录信息、用户详情信息、日志、回帖、主人回帖
2. 属性分析
3. 实体之间的关系

![QQZone](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/QQZone.png)



# 三.代码实现


## 	1. `pojo`类的实现

### 1.1设计原则

1. **主键应该是没有实际意义的列**，不要和业务由关联。因为将来可能涉及两表合并，主键将无法合并。

2. 设计时间类型时：

    --年月日时分秒（`java.util.Date`），

    --年月日（`java.sql.Date`），时分秒（`java.sql.TIme`）

3. 类与类之间的关系也要建立

    ```java
    public class UserBasic {
        private Integer id;
        private String loginId;
        private String nickName;
        private String pwd;
        private String headImg;
    	
        //类与类之间的关系
        private UserDetail userDetail;  //1:1
        private List<Topic> topicList;  //1:N
        private List<UserBasic> friendList; //M:N
    
        public UserBasic() {
        }
    }
    ```

### 1.2可能的异常

1. 查询好友列表时，查询到的`rsmd.getColumnName`为`fid`，而`UserBasic`中的属性为`id`。可以在`sql`语句中使用别名代替，并`rsmd.getColumnName`替代
2. 数据库连接失败：1.驱动是否导入，2.数据库选择，3.用户名和密码。
3. 获取`Topic`列表时调用`setValue(entity,columnName,columnValue)`设置`UserBasic author`属性，但是数据库中返回的类型是`Integer`。
4. 

## 2. `dao`类

## 3. `service`类

### 3.1 接口

### 3.2 接口的实现类

## 4. `controller`类

### 4.1`UserController`

### 4.2`TopicController`

需要根据`topicId`获取对应的`topic`信息

## 5.`HTML`的设计

### 4.1可能的异常`

1. 页面无法渲染。
   
    原因：没有经过`processTemplate(methodReturnStr,req,resp);`语句的渲染，而是直接跳转到静态页面
    
    解决：通过`PageController`控制渲染，调用`PageController`中的`page`方法跳转到`Page`参数对应的页面。
    
   ```html
    iframe th:src="@{/page.do?operate=page&page=frames/left}"
   ```
   
    ```java
    public class PageController {
        protected String page(String page){
           return page;
       }
    }
    ```
   
2. 点击好友名称时，在`iframe`中刷新整个页面。

    原因：`target`没有设置

    解决：超链接后添加` target="_top"`语句，使得新页面在顶层显示。

3. 访问动态页面时，一定要经过渲染。案例使用`PageController`渲染

4. `return`语句的使用：

### 4.2`login`登录界面

1. 静态页面：可以直接用地址访问。本案例使用`page.do?operate=page&page=login`渲染得到。

2. 启动时，访问的页面是:

    `http://	localhost	:8080	/pro23	/page.do	?operate=page&page=login`

    协议		`ServerIP`	`port`	`context root`	`request.getPath`	`query String`

### 4.3`index`展示界面

#### 4.3.1`top`页面信息

1. 显示欢迎进入好友页面：`th:text="|欢迎进入${session.friend.nickName}的空间！|"`

2. 如果不是在自己空间，显示连接返回自己空间（即将`friend`更新为当前`userBasic`）

    ```html
    <span th:if="${session.userBasic.id!=session.friend.id}">
        <a th:href="@{|/user.do?operate=friend&id=${session.userBasic.id}|}" target="_top">返回自己的空间!</a>
    </span>
    ```

    

#### 4.3.2`left`好友列表

1. 点击好友时，页面刷新为好友的日志列表。
    - 根据id查询好友的`userBasic`信息，覆盖`session.friend`
    - main页面展示friend的`topicList`，而不是`userBasic`中的`topicList`。方法：给连接添加`target="_top"`
    - 

#### 4.3.3`main`日志显示列表

1. 日期显示的格式化与解析

    在`java`中

    ```java
    //解析：看的懂 ---> 看不懂
    String dateStr = "2022-04-11 12:58:13";
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    try{
        Date date = sdf.parse(dateStr);
    }catch (ParseException e){
        e.printStackTrance();
    }
    //格式化：看不懂 ---> 看得懂
    Date date = new Date();
    String dateStr = sdf.format(date);
    ```

    在`thymeleaf`中

    ```html
    #dates.format(topic.topicDate,'yyyy-MM-dd HH:mm:ss')
    ```
    
    

#### 4.3.4`detial`日志详情

1. 一句里面不能有两个`style`

2. 配置`thymeleaf`文件

    `@{}`:表示省略了`pro21/`的绝对路径

    ```html
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" th:href="@{css/common.css}">
        <link rel="stylesheet" th:href="@{css/main.css}">
    </head>
    ```
    
3. 需要获取主人日志，回复列表（每个回复对应的主人日志）。

4. 日志中的主人回复没有显示

    ```html
    <!--head中-->
    <script language="JavaScript">
        function showDelImg(imgId){
            document.getElementById(imgId).style.display='inline';
        }
        function hiddenDelImg(imgId){
            document.getElementById(imgId).style.display='none';
        }
    </script>
    <!--body中-->
    <td  th:onmouseover="|showDelImg('a${reply.id}')|" th:onmouseout="|hiddenDelImg('a${reply.id}')|">
    <a th:unless="${reply.hostReply!=null}" th:id="|a${reply.id}|" th:href="#" style="float: right;display: inline;">主人回复</a>
    </td>
    ```

5. 可以删除的条件

    - 在我自己的空间
    - 这条评论是我发表的
    
6. 异常：删除父表时，需要保证没有子表在引用父表

    ```java
    MySQLIntegrityConstraintViolationException: Cannot delete or update a parent row: a foreign key constraint fails (`qqzonedb`.`t_host_reply`, CONSTRAINT `FK_host_reply` FOREIGN KEY (`reply`) REFERENCES `t_reply` (`id`))	
    ```

    

# 四.开发套路总结

## 1.拷贝`myssm`包

## 2.新建配置文件`applicationContext.xml`或者其他名字，在`web.xml`中指定文件名

## 3.在`web.xml`文件中配置：

1. 配置前缀和后缀，这样`thtmeleaf`引擎就可以根据我们返回的字符串进行拼接，再跳转

    ```xml
    <context-param>
        <param-name>view-prefix</param-name>
        <param-value>/</param-value>
    </context-param>
    <context-param>
        <param-name>view-suffix</param-name>
        <param-value>.html</param-value>
    </context-param>
    ```
    
2. 配置监听器要读取的参数，目的是加载`IOC`容器的配置文件（`applicationContext.xml`）

    ```xml
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>applicationContext</param-value>
    </context-param>
    ```

## 4.开发具体的业务模块：

1. 一个具体的业务模块的纵向结构：
    - `html`页面
    - `POJO`类
    - `DAO`接口和实现类
    - `Service`接口和实现类
    - `Controller`控制器组件
    
2. 如果`html`页面有`thymeleaf`表达式，一定不能够直接访问，必须要经过`PageController`

3. 在`applicationContext.xml`中配置`DAO,Service,Controller`，以及三者之间的依赖关系

4. `DAO`实现类中，继承`BaseDAO`，然后实现具体的接口，注意泛型不能写错，例如：

    `public class UserDAOImpl extends BaseDAO<User> implements UserDAO{}`

 5. `Service`是业务控制类，需要注意

    - 业务逻辑封装在Service这一层，不要分散在`Controller`层。也不要出现在`DAO`层（需要保证`DAO`方法的单精度特性）
    - 当需要使用其他模块的功能的时候，尽量调用别人的`service`，而不是深入到其他模块的`DAO`细节
    
 6. `Controller`类的编写规则

    - 在`applicationContext.xml`中来配置`Controller`

        `<bean id="user" class="com.atlff.qqzone.controllers.UserController">`

        那么，用户在前端发请求时，对应的`servletPath`就是`/User.do`，其中“user”就是此处`bean`的`id`值

    - 在`Controller`中设计的方法名需要和`operate`的值一致
    
        ```java
        public String login(String loginId, String pwd, HttpSession session){
            return "index";
        }
        ```
    
        此时验登录的表单如下:
    
        ```html
        <form th:action="@{/user.do}" method="post">
            <input type="hidden" name="operate" value="login">
        </form>
        ```
        
    - 在表单中,组件的`name`属性和`Controller`中的方法参数名一致
      
      `<input type="text" name="loginId">`
      
      `public String login(String loginId, String pwd, HttpSession session)`
      
     - 另外,`Controller`中的方法不一定都是通过请求参数获取
    
        ```java
        if("request".equals...) else if("resqonse".equals...) else if("session".equals...){
            //直接赋值
        }else{
            //此处才是从request的请求参数中获取
            request.getParameter("loginId")...
        }
        ```
    
     - `DispatcherServlet`中步骤的分类:
    
        0.从`application`作用域获取`IOC`容器
    
        1.解析`servletPath`,在`IOC`容器中寻找对应的`Controlelr`组件
    
        2.准备`operate`指定的方法所要求的参数
    
        3.调用`operate`指定的方法
    
        4.接收到执行`operate`指定的方法的返回值,对返回值进行处理  ----视图处理
        
     - 为什么`DispatcherServlet`能够从`application`作用域获取到`IOC`容器?
    
        `ContextLoaderListener`在容器启动时会执行初始化任务,而他的操作是:
    
        1.解析`IOC`的配置文件,创建一个一个的组件,并完成组件之间的依赖注入关系
    
        2.将`IOC`容器保存到`application`作用域

## 5.数据库连接异常

1. 数据库连接失败

    原因:`druid`没有添加到部署包

    解决:在项目结构中的`ProjectStructure`中将`jar`包添加部署

2. 数据库报错连接太多

    `Data source rejected establishment of connection, message from server:"Too many connections"`

    原因:`DruidDataSourceFactory.createDataSource(properties);`出现在`createConnection`中,导致创建多个连接池.

    解决:将连接池放置在初始化的代码段,而不是出现在方法中.















