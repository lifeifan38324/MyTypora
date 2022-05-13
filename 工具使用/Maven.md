# 一.项目构建原理

1. 构建步骤
    - 清理：将之前编译得到的字节码文件删除，为下一次编译做准备
    - 编译：将Java源程序编译成class字节码文件
    - 测试：自动测试，自动调用`junit`程序
    - 报告：测试程序执行的结果
    - 打包：动态web工程打成war包
    - 安装：Maven的概念----将打包得到的文件复制到 “仓库” 中的指定目录
    - 部署：将动态web工程生成的war包复制到`Servlet`容器的指定目录下，使其可以运行



# 二.安装Maven核心程序

1. 检查`JAVA_HOME`环境变量
2. 解压Maven核心程序，放在非中文无空格路径下
3. 配置Maven的环境变量
    - 配置`MAVEN_HOME`或者`M2_HOME`
    - 配置`path`
4. 验证：运行`mvn -v`。安装完成

# 三.核心概念

## 1.概念：

- 约定的目录结构
  
- `POM`
  
- 坐标
  
- **依赖**
  
- 仓库
  
- 生命周期/插件/目标
  
- 继承
  
- 聚合

## 2.案例学习：

1. 创建约定的目录结构

    ```
    Hello
    |---src
    |---|---main
    |---|---|---java
    |---|---|---resources
    |---|---test
    |---|---|---java
    |---|---|---resources
    |---pom.xml
    ```

    - 根目录：工程名
    - `src`目录：源码
    - `pom.xml`文件：Maven工程的核心配置文件
    - main目录：存放主程序
    - test目录：存放测试程序
    - `java`目录：存放`java`程序
    - resources目录：存放框架或其他工具的配置文件

2. 与构建过程相关的命令，即核心概念中的命令，必须进入到`pom.xml`所在的目录下

    **常用命令：**

    - `mvn clean`：清理
    - `mvn compile`：编译主程序
    - `mvn test-compile`：编译测试程序
    - `mvn test`：执行测试
    - `mvn package`：打包
    - `mvn install`：安装
    - `mvn site`：生成站点

3. `POM.xml`文件：项目对象模型，与构建相关的一切设置都在这里配置

4. 坐标：

    - 使用如下三个向量在 Maven 的仓库中唯一的确定一个 Maven 工程。

        [1]`groupid`：公司或组织的域名倒序+当前项目名称

        [2]`artifactId`：当前项目的模块名称

        [3]`version`：当前模块的版本

        ```xml
        <groupId>com.atguigu.maven</groupId>
        <artifactId>Hello</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        ```

5. 仓库：

    - 分类：

        1. 本地仓库：地址默认为`Default: ${user.home}/.m2/repository`

            在文件`apache-maven-3.2.2\conf`修改默认地址：`<localRepository>D:\develop_tools\Maven\RepMaven</localRepository>`

        2. 远程仓库：私服，中央仓库，中央仓库镜像。

6. 依赖

    - 在`POM.xml`中添加依赖：

        ```xml
        <dependencies>
            <dependency>
                <groupId>com.atlff.maven</groupId>
                <artifactId>Hello</artifactId>
                <version>0.0.1-SNAPSHOT</version>
                <scope>compile</scope>
            </dependency>
        </dependencies>
        ```

    - 被依赖的文件，必须提前在被依赖的文件的项目中，用`mvn install`添加到本地仓库中

    - 依赖的范围

        - compile范围依赖：
            - 对主程序是否有效：有效
            - 对测试程序是否有效：有效
            - 是否参与打包：参与
        - test范围依赖：
            - 对主程序是否有效：无效
            - 对测试程序是否有效：有效
            - 是否参与打包：不参与
        - provided范围依赖：指的是Tomcat服务器可以提供的jar包
            - 对主程序是否有效：有效
            - 对测试程序是否有效：有效
            - 是否参与打包：不参与

7. 生命周期

    - 执行顺序：参考构建原理
    - 过于多，看尚硅谷文件

# 四.Eclipse中使用Maven

1. 修改插件

    - installation：指定Maven核心程序的位置，建议修改为自己下载的
    - user setting：指定`conf/settings.xml`，进而根据配置文件获取本地仓库位置

2. 设置新建选项：Window，Perspective，Customize Perspective

3. 新建时，勾选`Create a simple project`

4. 修改`JDK`的版本：库右键，`Build Path`，修改`Jre`。之后修改`Compiler`中的`JDK`

    或者通过配置文件永久修改：

    ```xml
    <profile>
    	<id>jdk-1.7</id>
    	<activation>
    		<activeByDefault>true</activeByDefault>                             
    		<jdk>1.7</jdk>                                                      
    	</activation>
    	<properties>
    		<maven.compiler.source>1.7</maven.compiler.source>                  
    		<maven.compiler.target>1.7</maven.compiler.target>                  
    		<maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion>
    	</properties>
    </profile>
    ```

5. 新建web工程：新建时选择war包

# 小操作太碎了，请跳转到课件去查看。

# 五.IDEA中Maven的使用

1. 
