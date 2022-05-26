# 一.业务分析

## 1.用户登录，显示欢迎界面

1. 根据`uname`和`pwd`获取用户信息
1. 获取用户的购物车信息
1. 欢迎界面显示用户的名字，购物车项数量
1. 显示库存的图书信息

## 2.购物车操作

1. 将书籍添加到购物车
    - 创建购物车详情项
    - 添加或者更新购物车信息
2. 跳转到购物车
    - 重新加载用户的购物车信息
3. 在购物车中添加或者减少书籍
    - 添加或者更新购物车信息

## 3.结账

1. 订单表添加一条记录
    - 创建订单项，设置其属性
    - 添加订单项
2. 订单详情添加记录
    - 遍历购物车项，添加到订单详情中
3. 清空购物车记录
    - 遍历购物车项，删除购物车记录
4. 更新库存信息（未做）

## 4.查看订单

1. 获取用户信息
2. 获取用户对应的订单详情信息

# 二.数据库设计

![](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/1%E4%B8%9A%E5%8A%A1%E5%85%B3%E7%B3%BB.jpg)

### 2.1`t_book`: 

| 含义 | 序号  |    书名    |   作者    |  价格   |    销量    |    库存     |   封面    |     状态     |
| :--: | :---: | :--------: | :-------: | :-----: | :--------: | :---------: | :-------: | :----------: |
| 名称 | `id`  | `bookName` | `author`  | `price` | `salCount` | `bookCount` | `bookImg` | `bookStatus` |
| 类型 | `int` | `varchar`  | `varchar` | `float` | `varchar`  |    `int`    | `varchar` |  `tinyint`   |
| 约束 | 主键  | 唯一，非空 |   非空    |         |            |             |           |    默认0     |

### 2.2`t_user`:

| 含义 | 序号 |   用户名   |   密码    |   邮箱    |   角色    |
| :--: | :--: | :--------: | :-------: | :-------: | :-------: |
| 名称 | `id` |  `uname`   |   `pwd`   |  `email`  |  `role`   |
| 类型 | int  | `varchar`  | `varchar` | `varchar` | `tinyint` |
| 约束 | 主键 | 唯一，非空 |   非空    |           |           |

### 2.3`t_cart_item`

| 含义 | 序号  |    书籍    |    数量    |  所属用户  |
| :--: | :---: | :--------: | :--------: | :--------: |
| 名称 | `id`  | `bookBean` | `buyCount` | `userBean` |
| 类型 | `int` |   `int`    |   `int`    |   `int`    |
| 约束 | 主键  | 外键，非空 |            |    外键    |

### 2.4`t_order`

| 含义 | 序号  |  订单编号  |    日期     |     金额     |     数量     |   订单状态    |  所属用户  |
| :--: | :---: | :--------: | :---------: | :----------: | :----------: | :-----------: | :--------: |
| 名称 | `id`  | `orderNo`  | `orderDate` | `orderMoney` | `orderCount` | `orderStatus` | `userBean` |
| 类型 | `int` | `varchar`  | `datetime`  |   `double`   |    `int`     |     `int`     | `tinyint`  |
| 约束 | 主键  | 唯一，非空 |    非空     |              |              |               |    外键    |

### 2.5`t_order_detail`

| 含义 | 序号  |    商品    |    数量    |  所属订单   |
| :--: | :---: | :--------: | :--------: | :---------: |
| 名称 | `id`  | `bookBean` | `buyCount` | `orderBean` |
| 类型 | `int` |   `int`    |   `int`    |    `int`    |
| 约束 | 主键  | 外键，非空 |            | 外键，非空  |

# 三.项目细节

## 1.`thymeleaf`注意

1. 头部: `<html lang="en" xmlns:th="http://www.thymeleaf.org">`

2. 绝对引用：`@{}`表示省略到了`web/`。可以在配置文件中设置前缀和后缀。

3. 注意"|"的使用：包裹在`${}`最远的一层，但是要在其他的`{}`内。

    ```html
    <img th:src="@{|/static/uploads/${book.bookImg}|}" alt="图片加载失败"}>
    <p th:text="|书名：${book.bookName}|">书名:活着</p>
    ```

## 2.跳转页面注意

1. 页面所有需要的信息重新加载。
2. 重定向只是刷新页面，其本质是调用了`index`方法重新加载页面。若需要跳转到非`index`页面，则不能使用重定向。

## 3.`html`点击跳转事件使用

1. 调用`java`方法，如登录。需要提交表单

    - 从请求中获取`POST`请求的参数：`request.getParameter(参数名)`。这一步由发报机自动完成。

2. 调用`jsp`方法，如加购物车。没有提交表单，但是仍然需要提交传参数，且参数为动态参数。

    - 设置点击事件

        ```html
        th:onclick="|editCart(${cartItem.id}, ${cartItem.buyCount - 1})|"
        ```
        
    - 使用`.js`函数调用`java`

        ```javascript
        function editCart(cartItemId, buyCount){
            window.location.href="cart.do?operate=editCart&cartItemId=" + cartItemId + "&buyCount=" + buyCount;
        }
        ```

    - 在`java`中实现功能
## 4.字符集转换

1. `HTML` --> `JAVA` --> `MySQL`：重点关注`HTML`和`JAVA`之间的字符集设置

    - 设置注解：将字符集设置在参数中

        ```java
        @WebFilter(urlPatterns = {"*.do"}, initParams = {@WebInitParam(name = "encoding", value = "utf-8")})
        ```

    - 重写`init方法`：设置默认字符集

        ```java
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
            String encodingStr = filterConfig.getInitParameter("encoding");
            if(StringUtil.isNotEmpty(encodingStr)){
                encoding = encodingStr;
            }
        }
        ```

    - 重写doFIlter方法，修改字符集

        ```java
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            //修改默认字符编码
            ((HttpServletRequest)servletRequest).setCharacterEncoding(encoding);
            //放行
            filterChain.doFilter(servletRequest, servletResponse);
        
        }
        ```

        

2. `MySQL` --> `JAVA` --> `HTML`：同上，但无需设置。

3. 注意：`get`请求和`post`请求的字符集设置方式不同

    - **这里之后补充**

## 5.时间转换问题

1. `DAO`类型`java.util.Date` --> 数据库类型`datetime`。可以直接转换
2. 数据库类型`datetime` --> 返回类型  `LocalDataTime` --> `DAO`类型`java.util.Date`。**报错！！！**
    - 解决：把`orderDate`的类型改成`java.time.LocalDataTime`

## 6.使用过滤器限制用户访问的网页

1. 使用过滤器，但是要注意访问过滤器要放在字符转换过滤器和事务过滤器之后

1. 过滤器先后顺序规则：

    - 使用注解：按照定义的过滤器名字`filterName`顺序过滤，从a-z。
    - 使用配置文件：按照`web.xml`的顺序执行过滤。

1. 用户限制过滤器

    - 注解：配置过滤内容和过滤白名单

        ```java
        @WebFilter(urlPatterns = {"*.do", "*.html"},
                initParams = {
                    @WebInitParam(name = "bai", value = "/pro25/page.do?operate=page&page=user/login,/pro25/user.do?null")
                })
        ```

    - 重写`init方法`：获取注解中的白名单列表

        ```java
        @Override
        public void init(FilterConfig config) throws ServletException {
            String bai = config.getInitParameter("bai");
            String[] baiArr = bai.split(",");
            baiList = Arrays.asList(baiArr);
        }
        ```

    - 重写`doFilter方法`

        ```java
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            //将请求和响应强制类型转换
            HttpServletRequest request = (HttpServletRequest)servletRequest;
            HttpServletResponse response = (HttpServletResponse)servletResponse;
        
            //http://localhost:8080/pro25/page.do?operate=page&page=user/login
            //System.out.println("request.getRequestURI() = " + request.getRequestURI());
            //System.out.println("request.getQueryString() = " + request.getQueryString());
        
            //拼接请求字符串
            String requestURI = request.getRequestURI();
            String queryString = request.getQueryString();
            String str = requestURI + "?" + queryString;
        
            if(baiList.contains(str)){//如果请求白名单内容，放行
                filterChain.doFilter(request, response);
            }else {
                HttpSession session = request.getSession();
                Object curUserObj = session.getAttribute("curUser");
                if (curUserObj == null) {//如果用户没有登录，跳转到登录界面
                    response.sendRedirect("page.do?operate=page&page=user/login");
                } else {//若用户已经登录，放行
                    filterChain.doFilter(request, response);
                }
            }
        }
        ```
## 7. 通过反射获取方法的参数列表名称

正常情况`Java`编译时，默认反射参数列表的名称为`arg0，arg1...`。想要返回参数列表的字符串名称：参考[CSDN](https://blog.csdn.net/weixin_45820245/article/details/120857380?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165245237216782350953765%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=165245237216782350953765&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-120857380-null-null.142^v9^pc_search_result_control_group,157^v4^control&utm_term=idea%E5%8F%8D%E5%B0%84%E8%BF%94%E5%9B%9E%E7%9A%84%E5%8F%82%E6%95%B0%E5%88%97%E8%A1%A8%E5%90%8D%E7%A7%B0&spm=1018.2226.3001.4187)

建议修改`pom`文件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <compilerArgs>
                    <arg>-parameters</arg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```




# 四.一些其他技术

## 1.`Cookie`

1. 设置`Cookie`

    ```java
    //1.创建cookie对象
    Cookie cookie = new Cookie("uname", "Lifeifan");
    //2.将cookie保存到客户端
    resp.addCookie(cookie);
    //3.跳转页面
    req.getRequestDispatcher("hello01.html").forward(req,resp);
    ```

2. 设置`Cookie`的有效时长：一般采用默认

    ```java
    cookie.setMaxAge(60)		//设置cookie的有效时长60s
    cookie.setDomin(pattern);	//设置
    cookie.setPath(uri);		//设置传递cookie的路径，默认为全部
    ```

3. `Cookie`的应用

    - 记住用户名和密码10天
    - 10天免登录

## 2.`Kaptcha`:验证码

1. 添加jar包

2. 在`web.xml`文件中注册`KaptchaServlet`，并设置图片的相关属性

    - 注册：验证码图片的`src="kaptch.jpg"`，对应到`KaptchaServlet`的`url-pattern`中。

        ```xml
        <servlet>
            <servlet-name>KaptchaServlet</servlet-name>
            <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
        </servlet>
        
        <servlet-mapping>
            <servlet-name>KaptchaServlet</servlet-name>
            <url-pattern>/kaptch.jpg</url-pattern>
        </servlet-mapping>
        ```

        ```html
        <img src="kaptch.jpg" alt="图片加载失败">
        ```

    - 验证码属性：`com.google.code.kaptcha.Constants`。例如：

        ```xml
        <init-param>
            <param-name>kaptcha.border.color</param-name>
            <param-value>red</param-value>
        </init-param>
        ```

3. 校验。`Kaptcha`在生成验证码图片时，会将验证码信息保存到session中。

4. 异常：

    - 没有加载图片：1.检查注册是否正确。2.重启`Toncat`。

## 3.正则表达式

1. 定义：在`JavaScript`中定义

    - 对象形式：`var reg = new RegExp("abc","g");`
    - 直接量形式：`var reg = /abc/g`

2. 匹配模式

    - g 全局匹配
    - i 忽略大小写
    - m 多行匹配

3. 注册时校验用户信息

    - `<form th:action="@{/user.do}" method="post" onsubmit="return preRegist()">`中的`onsubmit`

        - `"return true"`点击时提交表单
        - `"return false"`点击时不提交表单

    - 使用`JavaScript`验证表单信息

        - 获取表单参数

            ```javascript
            //方式一，DOM: Document
            var unameText = document.getElementById("unameText");
            //方式二，BOM: Browser
            var unameText = document.forms[0].uname;
            ```

        - 校验表单

            ```javascript
            //验证邮箱
            var email = $("emailText").value;
            var emailSpan = $("emailSpan");
            var emailReg = /^\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/;
            if(!emailReg.test(email)){
                emailSpan.style.visibility="visible";
                return false
            }else{
                emailSpan.style.visibility="hidden";
            }
            ```

## 4.异步请求Ajax

目的：可以开辟一个新的线程，用来发送异步请求，然后当服务器给我们响应的时候进行回调操作

优势：提高用户体验，避免请求白页面；使得网页局部刷新：降低服务器负担，浏览器压力，网络带宽压力

案例：当用户名失去焦点时，验证当前用户名是否已经被注册。

1. 在`HTML`页面端，设置失去焦点事件

    ```html
    <input id="unameText" type="text" name="uname" placeholder="请输入用户名" value="lff" onblur="ckUname(this.value)"/>
    ```

2. 在`js`文件中定义`ckUname(uname)`方法

    ```javascript
    //1.如果想要发送异步请求，需要一个对象"XMLHttpRequest"
    var xmlHttpRequest ;
    
    //2.因为不同的浏览器的创建规则不同，所以使用一个方法来创建
    function createXMLHttpRequest(){
        if(window.XMLHttpRequest){
            //符合DOM2标准的浏览器，XMLHttpRequest的创建方式
            xmlHttpRequest = new XMLHttpRequest();
        }else if(window.ActiveXObject){//IE浏览器
            try{
                xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
            }catch (e){
                xmlHttpRequest = new ActiveXObject("Msxm12.XMLHTTP");
            }
    
        }
    }
    
    //3.在ckUname方法中创建xmlHttpRequest对象，并发送请求
    function ckUname(uname){
        createXMLHttpRequest();
        var url = "user.do?operate=ckUname&uname=" + uname;
        //3.1调用open方法
        xmlHttpRequest.open("GET",url,true);
        //3.2设置回调函数
        xmlHttpRequest.onreadystatechange = ckUnameCB;
        //3.3发送请求
        xmlHttpRequest.send();
    }
    
    //4.回调函数
    function ckUnameCB(){
        //判断xmlHttpRequest的状态
        //xmlHttpRequest.readyState的响应码：
        //0 - （未初始化）还没有调用send()方法
        //1 - （载入）已调用send()方法，正在发送请求
        //2 - （载入完成）send()方法已执行，已经接收到全部响应内容
        //3 - （交互）正在解析响应内容
        //4 - （完成）响应内容解析完成，客户端可以调用
        if(xmlHttpRequest.readyState == 4 && xmlHttpRequest.status == 200){
            //xmlHttpRequest.responseText表示服务器端响应的文本
            alert(xmlHttpRequest.responseText);
        }
    }
    ```

3. 修改中央控制器：

    ```java
    //3.3请求调用的函数
    public String ckUname(String uname){
        User user = userService.getUser(uname);
        if(user != null){
            //用户名被占用
            return "json:{'uname':'1'}";
        }else{
            //可以注册
            return "json:{'uname':'0'}";
        }
    }
    ```

    ```java
    //中央控制器处理返回字符串
    if(methodReturnStr.startsWith("json:")){
        String jsonStr = methodReturnStr.substring("json:".length());
        PrintWriter writer = resp.getWriter();
        writer.print(jsonStr);
        writer.flush();
    }
    ```

## 5.`VUE`

1. 背景知识：

    - 创建`js`对象

        ```javascript
        //方式一
        var persion = new Object();
        persion.pid = "p001";
        persion.pname = "jim";
        persion.sayHello = function(){
            alert("Hello, " + persion.pid);	
        }
        //方式二
        var persion = {
            "pid":"p001",
            "pname":"李非凡",
            "sayHello":function (){
                alert("Hello, " + persion.pname);
            }
        }
        ```

2. 简单使用

    - 绑定文本信息，属性，和双向绑定

        ```html
        <!--在JavaScript中创建Vue对象，并通过标签id进行绑定-->
        <script language="JavaScript">
            window.onload=function(){
                var vue = new Vue({
                    "el":"#div0",
                    data:{
                        msg:"hello world!",
                        uname:"里非凡"
                    }
                });
            }
        </script>
        <!--在html中进行绑定-->
        <div id="div0">
            <!--在对应的标签中使用msg文本-->
            <span>{{msg}}</span>
            <!--属性：在对应的标签中, 通过"v-bind:"使用属性。注意"v-bind"可以省略，保留":"-->
            <input type="button" v-bind:value="uname" onclick="hello()">
            <!--双向绑定："v-model:value"中的":value"可以省略-->
            <input type="text"  v-model:value="msg" >
        </div>
        ```

    - 条件标签`v-if`和`v-else`。补充`v-show`：通过`display`控制标签的显示。

        ```html
        <div v-if="num%2==0" style="width: 200px; height: 200px; background-color: orange;"></div>
        <div v-else="num%2==0" style="width: 200px; height: 200px; background-color: aqua;"></div>
        ```

    - 迭代标签`v-for`

        ```html
        <tr v-for="fruit in fruitList" align="center">
            <td>{{fruit.fname}}</td>
            <td>{{fruit.price}}</td>
            <td>{{fruit.fcount}}</td>
            <td>{{fruit.remark}}</td>
        </tr>
        ```

    - 绑定点击事件`v-on:click`，可以简写为`@click`

        ```html
        methods:{
            reverseStr:function(){
                this.msg = this.msg.split("").reverse().join("");
            }
        }
        
        <div id="div0">
            <span>{{msg}}</span><br/>
            <input type="button" value="反转" v-on:click="reverseStr">
        </div>
        ```

    - 监听属性值改动

        ```html
        watch:{
            num1:function(newValue){
                this.num3 = newValue + this.num2;
            },
            num2:function(newValue){
                this.num3 = newValue + this.num1;
            }
        }
        
        <input type="text" v-model="num1">
        <input type="text" v-model="num2">
        <span>{{num3}}</span><br/>
        ```

    - 生命周期

        ```javascript
        <script language="JavaScript">
            window.onload=function(){
            var vue = new Vue({
                "el":"#div0",
                data:{
                    msg:"这里是msg。。。"
                },
                methods:{
                    changeMsg:function(){
                        if(this.msg == "这里是msg。。。"){
                            this.msg = "msg被改变";
                        }else{
                            this.msg = "这里是msg。。。";
                        }
                    }
                },
                //生命周期
                beforeCreate:function(){
                    console.log("beforeCreate:vue对象创建之前--------");
                    console.log("msg:" + this.msg);
                },
                created:function(){
                    console.log("created:vue对象创建之后--------");
                    console.log("msg:" + this.msg);
                },
                beforeMount:function(){
                    console.log("beforeMount:数据装载之前--------");
                    console.log("msg:" + document.getElementById("span").innerText);
                },
                mounted:function(){
                    console.log("mounted:数据装载之后--------");
                    console.log("msg:" + document.getElementById("span").innerText);
                },
                beforeUpdate:function(){
                    console.log("beforeUpdate:数据更新之前--------");
                    console.log("msg:" + document.getElementById("span").innerText);
                },
                updated:function(){
                    console.log("updated:数据更新之后--------");
                    console.log("msg:" + document.getElementById("span").innerText);
                },
            });
        }
        </script>
        ```

        ```html
        <div id="div0">
            <span id="span">{{msg}}</span><br/>
            <input type="button" value="改变msg的值" @click="changeMsg">
        </div>
        ```

        

3. 错误原因

    - 创建对象时检查`Vue`的`V`是否大写

## 6.`Axios`:是Ajax的一个框架

1. 导入`axios`的`js`文件：`<script language="JavaScript" src="script/axios.min.js"></script>`

2. 调用方法

    ```html
    <input type="button" value="点击发送异步请求" @click="lff">
    ```

3. 在`vue`的方法中配置`axios`，向服务器发送普通数据格式

    ```javascript
    lff:function(){
        axios({
            method:"POST",
            url:"axiso02.do",
            params:{
                uname:vue.uname,
                pwd:vue.pwd
            }
        })
            .then(function (value) {
            console.log(value);
        })
            .catch(function (reason) {
            console.log(reason);
        })
    }
    ```

## 7.`JSON`数据格式，特点：简洁，节约网络带宽

### 7.1客户端向服务器发送`JSON`数据格式

+++客户端发送`JSON`数据格式

1. 将`params`改成`data`

2. 发送格式

    ```javascript
    axios02:function(){
        axios({
            method:"POST",
            url:"axios02.do",
            data:{
                uname:vue.uname,
                pwd:vue.pwd
            }
        })
            .then(function (value) {
            console.log(value);
        })
            .catch(function (reason) {
            console.log(reason);
        });
    }
    ```

    

3. 问题：**无法发送`JSON`格式的数据，可以发送普通格式**

    - 原因：`axios`的键值`method`写成了`mathod`

+++服务器接受`JSON`格式

1. 使用流`JSON`为字符串

    ```java
    //1.创建接收流
    BufferedReader reader = req.getReader();
    //2.创建缓冲字符串
    String str = null;
    //3.创建拼接字符串
    StringBuffer stringBuffer = new StringBuffer("");
    //4.接收并拼接字符串
    while((str = reader.readLine()) != null){
        stringBuffer.append(str);
    }
    //5.关闭流资源
    reader.close();
    //6.还原JSON
    str = stringBuffer.toString();
    System.out.println("str = " + str);
    ```

2. 将`JSON`字符串转化为类

    ```java
    //1.获取GSON类
    Gson gson = new Gson();
    //2.使用GSON类将字符串转化为对应类
    User user = gson.fromJson(str, User.class);
    ```

### 7.2服务器向客户端发送`JSON`数据格式

+++服务器发送`JSON`格式

1. 将类转换为`JSON`格式

    ```java
    //1.将类转换为`JSON`格式
    String userJsonStr = gson.toJson(user);
    //2.设置响应的编码格式
    resp.setCharacterEncoding("UTF-8");
    //3.设置MIME-TYPE
    resp.setContentType("application/json;charset=UTF-8");
    //4.执行响应
    resp.getWriter().write(userJsonStr);
    ```

    注意：[第三步可以参考链接](https://heavy_code_industry.gitee.io/code_heavy_industry/pro001-javaweb/lecture/chapter06/)

+++客户端接收`JSON`数据并转化为`js`类

1. 接收并手动转化为`js`类

    ```javascript
    var data = value.data;
    console.log(data);
    vue.uname=data.uname;
    vue.pwd=data.pwd;
    ```

2. 若需要`json`对象，将字符串转化为`JSON`对象`JSON.parse()`

    ```javascript
    var str = "{\"uname\":\"lfff\",\"age\":20}";
    //使用JSON.parse()转换
    var user = JSON.parse(str);
    alert(user.uname + "_" + user.age);
    ```

    格式化：看得懂-->看不懂(统一格式)

    解析：看不懂-->看得懂(直观)

    **补充：将`js对象`转化为字符串`JSON.stringify()`**

    ```javascript
    var user = {"uname":"王林林","age":18};//js对象
    var str = JSON.stringify(user);
    ```

    

3. 报错：

    - 无法转换成`json`对象，原因：`“：”`写成了`“=”`

## 8.实战应用：使用`vue+axios`替换`thymeleaf`

1. 导入`vue`和`axios`的`js`文件，并添加引用

2. 将原网页的动态文本替换为`vue文本`，属性替换为`vue属性`

3. 源文件的局部动态数据加载和更新，使用`vue对象`的方法调用`axios`异步请求

    ```javascript
    window.onload=function(){
        var vue = new Vue({
            el:"#cart_div",
            data:{
                cart:{},
            },
            methods:{
                getCart:function(){
                    axios({
                        method:"POST",
                        url:"cart.do",
                        params:{
                            operate:"cartInfo"
                        }
                    })
                        .then(function (value){
                        	//接收返回的数据，并更新vue的属性
                            console.log(value.data);
                            var cart = value.data;
                            vue.cart = cart;
                        })
                        .catch(function (reason){
                            console.log(reason.data);
                        })
                },
                editCart:function(cartItemId, buyCount){
                    axios({
                        method:"POST",
                        url:"cart.do",
                        params:{
                            operate:"editCart",
                            cartItemId:cartItemId,
                            buyCount:buyCount
                        }
                    })
                        .then(function (value){
                            vue.getCart();
                        })
                        .catch(function (reason){
                            console.log(reason.data);
                        })
                }
            },
            mounted:function(){
                this.getCart();
            }
        });
    };
    ```

    

