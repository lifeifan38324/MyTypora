# 一. 语法

1. 注意：在Linux环境下是大小写敏感的。

2. 导入数据库：`source 全路径 *.sql`

3. 登录：

    `mysql -h 主机名 -P 端口号 -u 用户名 -p密码` 

3. 建议基本规范：

    - 数据库名、表名、表别名、字段名、字段别名等都小写

    - SQL 关键字、函数名、绑定变量等都大写
    
4. 基本查看命令：
    - `SHOW DATABASES / TABLES`：查询所有数据库或者当前数据库的所有表
    - `DESC 表名`：查询表的字段信息

## 1. `DQL`数据查询语言

### 1.1 基本查询结构

```sql
SELECT [DISTINCT 去重] 列名1 AS 别名1, 列名2 AS 别名2
FROM 表名1 AS 表别名1  [INNER | LEFT(OUTER) | RIGHT(OUTER)] JOIN 表名2 AS 表别名2
ON 关联条件
WHERE 过滤条件
GROUP BY 列名
ORDER BY 列名字(默认升序ASC) DESC
LIMIT [位置偏移量,] 行数
```

### 1.2 运算符

#### 1.2.1算术运算符：

1. 加（+）、减（-）、乘（*）、除（/）和取模（%）运算
2. 在MySQL中+只表示数值相加。如果遇到非数值类型，先尝试转成数值，如果转失败，就按0计算。（补充：MySQL中字符串拼接要使用字符串函数`CONCAT()`实现）
3. 在MySQL中，一个数除以0为NULL。

#### 1.2.2 比较运算符：

![image-20220531073620847](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531073620847.png)

1. 如果等号两边的值一个是整数，另一个是字符串，则MySQL会将字符串转化为数字进行比较。

2. 如果等号两边的值、字符串或表达式中有一个为NULL，则比较结果为NULL。

3. 安全等于运算符（<=>）与等于运算符（=）的作用是相似的， 唯一的区别 是‘<=>’可以用来对NULL进行判断。在两个操作数均为NULL时，其返回值为1，而不为NULL；当一个操作数为NULL时，其返回值为0，而不为NULL。

4. 不等于运算符（<>和!=）用于判断两边的数字、字符串或者表达式的值是否不相等，如果不相等则返回1，相等则返回0。不等于运算符不能判断NULL值。如果两边的值有任意一个为NULL，或两边都为NULL，则结果为NULL。

    ![image-20220531074534559](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531074534559.png)

5. LIKE运算符：LIKE运算符通常使用如下通配符

    - “%”：匹配0个或多个字符。 

    - “_”：只能匹配一个字符。 

    - ESCAPE：如果使用\表示转义，要省略ESCAPE。如果不是\，则要加上ESCAPE。

        ```sql
        SELECT job_id 
        FROM jobs 
        WHERE job_id 
        LIKE ‘IT$_%‘ escape ‘$‘;
        ```

        

6. REGEXP运算符：`expr REGEXP 匹配条件`

#### 1.2.3 逻辑运算符

![image-20220531075632445](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531075632445.png)

1. 优先级：OR可以和AND一起使用，但是在使用时要注意两者的优先级，由于AND的优先级高于OR，因此先对AND两边的操作数进行操作，再与OR中的操作数结合。

#### 1.2.4 位运算符

![image-20220531075851449](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531075851449.png)

### 1.3 多表查询

#### 1.3.1 外连接

1. 如果是左外连接，则连接条件中左边的表也称为 主表 ，右边的表称为 从表 

    如果是右外连接，则连接条件中右边的表也称为 主表 ，左边的表称为 从表 。

2. 合并查询结果：

    - UNION操作符：合并并去重

    - UNION ALL操作符：合并但不去重

3. 7种SQL  JOINS的实现

   ![image-20220531082207696](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531082207696.png)
   
   ```sql
   #中图：内连接 A∩B 
   SELECT employee_id,last_name,department_name 
   FROM employees e JOIN departments d 
   ON e.`department_id` = d.`department_id`;
   ```
   
   ```sql
   #左上图：左外连接 
   SELECT employee_id,last_name,department_name 
   FROM employees e LEFT JOIN departments d 
   ON e.`department_id` = d.`department_id`;
   ```
   
   ```sql
   #右上图：右外连接 
   SELECT employee_id,last_name,department_name 
   FROM employees e RIGHT JOIN departments d 
   ON e.`department_id` = d.`department_id`;
   ```
   
   ```sql
   #左中图：A - A∩B 
   SELECT employee_id,last_name,department_name 
   FROM employees e LEFT JOIN departments d 
   ON e.`department_id` = d.`department_id` 
   WHERE d.`department_id` IS NULL
   ```
   
   ```sql
   #右中图：B-A∩B 
   SELECT employee_id,last_name,department_name 
   FROM employees e RIGHT JOIN departments d 
   ON e.`department_id` = d.`department_id` 
   WHERE e.`department_id` IS NULL
   ```
   
   ```sql
   #左下图：满外连接 # 左中图 + 右上图 A∪B 
   SELECT employee_id,last_name,department_name 
   FROM employees e LEFT JOIN departments d 
   ON e.`department_id` = d.`department_id` 
   WHERE d.`department_id` IS NULL 
   
   UNION ALL #没有去重操作，效率高 
   
   SELECT employee_id,last_name,department_name 
   FROM employees e RIGHT JOIN departments d 
   ON e.`department_id` = d.`department_id`;
   ```
   
   ```sql
   #右下图 #左中图 + 右中图 A ∪B- A∩B 或者 (A - A∩B) ∪ （B - A∩B） 
   SELECT employee_id,last_name,department_name 
   FROM employees e LEFT JOIN departments d 
   ON e.`department_id` = d.`department_id` 
   WHERE d.`department_id` IS NULL 
   
   UNION ALL  #没有去重操作，效率高 
   
   SELECT employee_id,last_name,department_name 
   FROM employees e RIGHT JOIN departments d 
   ON e.`department_id` = d.`department_id` 
   WHERE e.`department_id` IS NULL
   ```

### 1.4 单行函数

#### 1.4.1 数值函数

1. 基本函数

    ![image-20220531124410401](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531124410401.png)
    
2.  角度与弧度互换函数

    ![image-20220531124701676](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531124701676.png)

3. 三角函数

    ![image-20220531124740789](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531124740789.png)

4. 指数与对数

    ![image-20220531124836678](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531124836678.png)

5. 进制间的转换

    ![image-20220531125028140](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531125028140.png)

#### 1.4.2 字符串函数

| **函数**                          | **用法**                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| ASCII(S)                          | 返回字符串S中的第一个字符的ASCII码值                         |
| CHAR_LENGTH(s)                    | 返回字符串s的字符数。作用与CHARACTER_LENGTH(s)相同           |
| LENGTH(s)                         | 返回字符串s的字节数，和字符集有关                            |
| CONCAT(s1,s2,......,sn)           | 连接s1,s2,......,sn为一个字符串                              |
| CONCAT_WS(x, s1,s2,......,sn)     | 同CONCAT(s1,s2,...)函数，但是每个字符串之间要加上x           |
| INSERT(str, idx, len, replacestr) | 将字符串str从第idx位置开始，len个字符长的子串替换为字符串replacestr |
| REPLACE(str, a, b)                | 用字符串b替换字符串str中所有出现的字符串a                    |
| UPPER(s) 或 UCASE(s)              | 将字符串s的所有字母转成大写字母                              |
| LOWER(s) 或LCASE(s)               | 将字符串s的所有字母转成小写字母                              |
| LEFT(str,n)                       | 返回字符串str最左边的n个字符                                 |
| RIGHT(str,n)                      | 返回字符串str最右边的n个字符                                 |
| LPAD(str, len, pad)               | 用字符串pad对str最左边进行填充，直到str的长度为len个字符     |
| RPAD(str ,len, pad)               | 用字符串pad对str最右边进行填充，直到str的长度为len个字符     |
| LTRIM(s)                          | 去掉字符串s左侧的空格                                        |
| RTRIM(s)                          | 去掉字符串s右侧的空格                                        |
| TRIM(s)                           | 去掉字符串s开始与结尾的空格                                  |
| TRIM(s1 FROM s)                   | 去掉字符串s开始与结尾的s1                                    |
| TRIM(LEADING s1 FROM s)           | 去掉字符串s开始处的s1                                        |
| TRIM(TRAILING s1 FROM s)          | 去掉字符串s结尾处的s1                                        |
| REPEAT(str, n)                    | 返回str重复n次的结果                                         |
| SPACE(n)                          | 返回n个空格                                                  |
| STRCMP(s1,s2)                     | 比较字符串s1,s2的ASCII码值的大小                             |
| SUBSTR(s,index,len)               | 返回从字符串s的index位置起len个字符，作用与SUBSTRING(s,n,len)、 MID(s,n,len)相同 |
| LOCATE(substr,str)                | 返回字符串substr在字符串str中首次出现的位置，作用于POSITION(substr IN str)、INSTR(str,substr)相同。未找到，返回0 |
| ELT(m,s1,s2,…,sn)                 | 返回指定位置的字符串，如果m=1，则返回s1，如果m=2，则返回s2，如果m=n，则返回sn |
| FIELD(s,s1,s2,…,sn)               | 返回字符串s在字符串列表中第一次出现的位置                    |
| FIND_IN_SET(s1,s2)                | 返回字符串s1在字符串s2中出现的位置。其中，字符串s2是一个以逗号分隔的字符串 |
| REVERSE(s)                        | 返回s反转后的字符串                                          |
| NULLIF(value1,value2)             | 比较两个字符串，如果value1与value2相等，则返回NULL，否则返回value1 |

<span style="color:red;font-weight:bold">注意：MySQL中，字符串的位置是从1开始的。</span>

#### 1.4.3 日期与时间函数

1. 获取日期、时间

    | 函数                                                         | 用法                           |
    | ------------------------------------------------------------ | ------------------------------ |
    | **CURDATE()** ，CURRENT_DATE()                               | 返回当前日期，只包含年、月、日 |
    | **CURTIME()** ， CURRENT_TIME()                              | 返回当前时间，只包含时、分、秒 |
    | **NOW()** / SYSDATE() / CURRENT_TIMESTAMP() / LOCALTIME() / LOCALTIMESTAMP() | 返回当前系统日期和时间         |
    | UTC_DATE()                                                   | 返回UTC（世界标准时间）日期    |
    | UTC_TIME()                                                   | 返回UTC（世界标准时间）时间    |

2. 日期与时间戳的转换

    | **函数**                 | **用法**                                                     |
    | ------------------------ | ------------------------------------------------------------ |
    | UNIX_TIMESTAMP()         | 以UNIX时间戳的形式返回当前时间。SELECT UNIX_TIMESTAMP() - \>1634348884 |
    | UNIX_TIMESTAMP(date)     | 将时间date以UNIX时间戳的形式返回。                           |
    | FROM_UNIXTIME(timestamp) | 将UNIX时间戳的时间转换为普通格式的时间                       |

3.  获取月份、星期、星期数、天数等函数

    | 函数                                     | 用法                                            |
    | ---------------------------------------- | ----------------------------------------------- |
    | YEAR(date) / MONTH(date) / DAY(date)     | 返回具体的日期值                                |
    | HOUR(time) / MINUTE(time) / SECOND(time) | 返回具体的时间值                                |
    | MONTHNAME(date)                          | 返回月份：January，...                          |
    | DAYNAME(date)                            | 返回星期几：MONDAY，TUESDAY.....SUNDAY          |
    | WEEKDAY(date)                            | 返回周几，注意，周1是0，周2是1，。。。周日是6   |
    | QUARTER(date)                            | 返回日期对应的季度，范围为1～4                  |
    | WEEK(date) ， WEEKOFYEAR(date)           | 返回一年中的第几周                              |
    | DAYOFYEAR(date)                          | 返回日期是一年中的第几天                        |
    | DAYOFMONTH(date)                         | 返回日期位于所在月份的第几天                    |
    | DAYOFWEEK(date)                          | 返回周几，注意：周日是1，周一是2，。。。周六是7 |

4.  日期的操作函数

    | **函数**                | **用法**                                   |
    | ----------------------- | ------------------------------------------ |
    | EXTRACT(type FROM date) | 返回指定日期中特定的部分，type指定返回的值 |

    EXTRACT(type FROM date)函数中type的取值与含义：

    ![image-20220531141317526](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531141317526.png)

    ![image-20220531141339079](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531141339079.png)

5. 时间和秒钟转换的函数

    | **函数**             | **用法**                                                     |
    | -------------------- | ------------------------------------------------------------ |
    | TIME_TO_SEC(time)    | 将 time 转化为秒并返回结果值。转化的公式为： 小时\*3600+分钟\*60+秒 |
    | SEC_TO_TIME(seconds) | 将 seconds 描述转化为包含小时、分钟和秒的时间                |

6. 计算日期和时间的函数

    **第一组：**

    | **函数**                                                     | **用法**                                       |
    | ------------------------------------------------------------ | ---------------------------------------------- |
    | DATE_ADD(datetime, INTERVAL expr type)， ADDDATE(date,INTERVAL expr type) | 返回与给定日期时间相差INTERVAL时间段的日期时间 |
    | DATE_SUB(date,INTERVAL expr type)， SUBDATE(date,INTERVAL expr type) | 返回与date相差INTERVAL时间间隔的日期           |

    上述函数中type的取值：

    ![image-20220531141858980](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531141858980.png)

    **第二组：**

    | 函数                         | 用法                                                         |
    | ---------------------------- | ------------------------------------------------------------ |
    | ADDTIME(time1,time2)         | 返回time1加上time2的时间。当time2为一个数字时，代表的是秒 ，可以为负数 |
    | SUBTIME(time1,time2)         | 返回time1减去time2后的时间。当time2为一个数字时，代表的是 秒 ，可以为负数 |
    | DATEDIFF(date1,date2)        | 返回date1 - date2的日期间隔天数                              |
    | TIMEDIFF(time1, time2)       | 返回time1 - time2的时间间隔                                  |
    | FROM_DAYS(N)                 | 返回从0000年1月1日起，N天以后的日期                          |
    | TO_DAYS(date)                | 返回日期date距离0000年1月1日的天数                           |
    | LAST_DAY(date)               | 返回date所在月份的最后一天的日期                             |
    | MAKEDATE(year,n)             | 针对给定年份与所在年份中的天数返回一个日期                   |
    | MAKETIME(hour,minute,second) | 将给定的小时、分钟和秒组合成时间并返回                       |
    | PERIOD_ADD(time,n)           | 返回time加上n后的时间                                        |

7. 日期的格式化与解析

    <span style="color:red;font-weight:bold">格式化：DATE对象===》字符串</span>

    <span style="color:red;font-weight:bold">解析：字符串===》DATE对象</span>

    | 函数                              | 用法                                       |
    | --------------------------------- | ------------------------------------------ |
    | DATE_FORMAT(date,fmt)             | 按照字符串fmt格式化日期date值              |
    | TIME_FORMAT(time,fmt)             | 按照字符串fmt格式化时间time值              |
    | GET_FORMAT(date_type,format_type) | 返回日期字符串的显示格式                   |
    | STR_TO_DATE(str, fmt)             | 按照字符串fmt对str进行解析，解析为一个日期 |

    上述 非GET_FORMAT 函数中fmt参数常用的格式符：

    ![image-20220531143006828](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531143006828.png)

    GET_FORMAT函数中date_type和format_type参数取值如下：

    ![image-20220531143245120](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531143245120.png)

#### 1.4.4 流程控制函数

| 函数                                                         | 用法                                            |
| ------------------------------------------------------------ | ----------------------------------------------- |
| IF(value,value1,value2)                                      | 如果value的值为TRUE，返回value1，否则返回value2 |
| IFNULL(value1, value2)                                       | 如果value1不为NULL，返回value1，否则返回value2  |
| CASE WHEN 条件1 THEN 结果1 WHEN 条件2 THEN 结果2 .... [ELSE resultn] END | 相当于Java的if...else if...else...              |
| CASE expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END | 相当于Java的switch...case...                    |

#### 1.4.5 加密与解密函数

| 函数                        | 用法                                                         |
| --------------------------- | ------------------------------------------------------------ |
| PASSWORD(str)               | 返回字符串str的加密版本，41位长的字符串。加密结果 不可 逆 ，常用于用户的密码加密 |
| MD5(str)                    | 返回字符串str的md5加密后的值，也是一种加密方式。若参数为NULL，则会返回NULL |
| SHA(str)                    | 从原明文密码str计算并返回加密后的密码字符串，当参数为NULL时，返回NULL。 SHA加密算法比MD5更加安全 。 |
| ENCODE(value,password_seed) | 返回使用password_seed作为加密密码加密value                   |
| DECODE(value,password_seed) | 返回使用password_seed作为加密密码解密value                   |

#### 1.4.6 MySQL信息函数

| 函数                                                  | 用法                                                     |
| ----------------------------------------------------- | -------------------------------------------------------- |
| VERSION()                                             | 返回当前MySQL的版本号                                    |
| CONNECTION_ID()                                       | 返回当前MySQL服务器的连接数                              |
| DATABASE()，SCHEMA()                                  | 返回MySQL命令行当前所在的数据库                          |
| USER()，CURRENT_USER()、SYSTEM_USER()，SESSION_USER() | 返回当前连接MySQL的用户名，返回结果格式为“主机名@用户名” |
| CHARSET(value)                                        | 返回字符串value自变量的字符集                            |
| COLLATION(value)                                      | 返回字符串value的比较规则                                |

#### 1.4.7 其他函数

| 函数                           | 用法                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| FORMAT(value,n)                | 返回对数字value进行格式化后的结果数据。n表示 四舍五入 后保留到小数点后n位 |
| CONV(value,from,to)            | 将value的值进行不同进制之间的转换                            |
| INET_ATON(ipvalue)             | 将以点分隔的IP地址转化为一个数字                             |
| INET_NTOA(value)               | 将数字形式的IP地址转化为以点分隔的IP地址                     |
| BENCHMARK(n,expr)              | 将表达式expr重复执行n次。用于测试MySQL处理expr表达式所耗费的时间 |
| CONVERT(value USING char_code) | 将value所使用的字符编码修改为char_code                       |

### 1.5 聚合函数

#### 1.5.1 简单函数

1. AVG和SUM函数
2. MIN和MAX函数
3. COUNT函数

#### 1.5.2 GROUP BY

1. 单列分组

    ```sql
    SELECT department_id, AVG(salary) 
    FROM employees 
    GROUP BY department_id ;
    ```
    
2. 多列分组

    ```sql
    SELECT department_id dept_id, job_id, SUM(salary) 
    FROM employees 
    GROUP BY department_id, job_id ;
    ```

3. GROUP BY中使用WITH ROLLUP

    ```sql
    SELECT department_id,AVG(salary) 
    FROM employees 
    WHERE department_id > 80 
    GROUP BY department_id WITH ROLLUP;
    ```

    注意：当使用ROLLUP时，不能同时使用ORDER BY子句进行结果排序，即ROLLUP和ORDER BY是互相排斥的。

#### 1.5.3 HAVING

1. 与WHERE类似，对分组结果进行筛选

2. 使用注意

    - 行已经被分组。
    - 使用了聚合函数
    - 满足HAVING 子句中条件的分组将被显示
    - HAVING 不能单独使用，必须要跟 GROUP BY 一起使用

3. 与WHERE对比

    - 区别1：WHERE 可以直接使用表中的字段作为筛选条件，但不能使用分组中的计算函数作为筛选条件；HAVING 必须要与 GROUP BY 配合使用，可以把分组计算的函数和分组字段作为筛选条件。
    - 区别2：如果需要通过连接从关联表中获取需要的数据，WHERE 是先筛选后连接，而 HAVING 是先连接后筛选。

#### 1.5.4 SELECT的执行过程

1. 语法结构
    ![image-20220531160706340](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531160706340.png)
    
2. 执行顺序

    `FROM -> WHERE -> GROUP BY -> HAVING -> SELECT 的字段 -> DISTINCT -> ORDER BY -> LIMIT `

## 2. `DDL`数据定义语言

定义了不同的数据库、表、视图、索引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。

主要的语句关键字包括 CREATE 、 DROP 、 ALTER 等

### 2.1 数据类型

![image-20220531161933300](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531161933300.png)

其中，常用的几类类型介绍如下：

![image-20220531162012926](https://typora-lff.oss-cn-guangzhou.aliyuncs.com/image-20220531162012926.png)

### 2.2 数据库操作

1. 创建库

    ```sql
    CREATE DATABASE [IF NOT EXISTS] 数据库名 [CHARACTER SET 字符集];
    ```

2. 查看库

    - 查看当前所有的数据库：

        ```sql
        SHOW DATABASES; #有一个S，代表多个数据库
        ```

    - 查看当前正在使用的数据库:

        ```sql
        SELECT DATABASE(); #使用的一个 mysql 中的全局函数
        ```

    - 查看数据库的创建信息：

        ```sql
        SHOW CREATE DATABASE 数据库名;
        ```

    - 切换数据库

        ```sql
        SHOW CREATE DATABASE 数据库名;
        ```

3. 修改库字符集

    ```sql
    ALTER DATABASE 数据库名 CHARACTER SET 字符集; #比如：gbk、utf8等
    ```

4. 删除数据库

    ```sql
    DROP DATABASE IF EXISTS 数据库名;
    ```

### 2.3 表操作

需要具备CREATE TABLE权限

1. 创建表

    方式一：

    ```sql
    CREATE TABLE [IF NOT EXISTS] 表名(
        字段1, 数据类型 [约束条件] [默认值],
        字段2, 数据类型 [约束条件] [默认值],
        字段3, 数据类型 [约束条件] [默认值],
        ……
        [表约束条件]
    );
    ```

    方式二：

    ```sql
    CREATE TABLE dept80 
    AS
    SELECT employee_id, last_name, salary*12 ANNSAL, hire_date 
    FROM employees 
    WHERE department_id = 80;
    ```

2. 查看表

    ```sql
    SHOW CREATE TABLE 表名;
    ```

3. 重命名表

    ```sql
    RENAME TABLE emp TO myemp;
    ```

4. 删除表

    ```sql
    DROP TABLE [IF EXISTS] 数据表1 [, 数据表2, …, 数据表n];
    ```

5. 清空表

    ```sql
    TRUNCATE TABLE detail_dept;
    ```

    TRUNCATE语句**不能回滚**

    ```sql
    DELETE FROM emp2;
    ```

### 2.4 列操作

1. 增加

    ```sql
    ALTER TABLE 表名 ADD 【COLUMN】 字段名 字段类型 【FIRST|AFTER 字段名】;
    ```

2. 修改

    修改字段数据类型、长度、默认值、位置

    ```sql
    ALTER TABLE 表名 MODIFY 【COLUMN】 字段名1 字段类型 【DEFAULT 默认值】【FIRST|AFTER 字段名 2】;
    ```

3. 重命名列

    ```sql
    ALTER TABLE 表名 CHANGE 【column】 列名 新列名 新数据类型;
    ```

4. 删除

    ```sql
    ALTER TABLE 表名 DROP 【COLUMN】字段名
    ```

### 2.5 约束

1. 非空约束：`NOT NULL`

2. 唯一约束：`UNIQUE`

    - 单字段

        ```sql
        create table student(
            sid int, sname varchar(20),
            tel char(11) unique,
            cardid char(18) unique key
        );
        ```

    - 多字段

        ```sql
        CREATE TABLE USER(
            id INT NOT NULL, #列级模式
            NAME VARCHAR(25),
            PASSWORD VARCHAR(16),
        	-- 使用表级约束语法
            [CONSTRAINT uk_name_pwd] UNIQUE(NAME,PASSWORD)  #表级模式
        );
        ```

    添加

    ```sql
    #方式1： 
    alter table 表名称 add unique key(字段列表);
    #方式2： 
    alter table 表名称 modify 字段名 字段类型 unique;
    ```

    删除

    ```sql
    ALTER TABLE 表名 
    DROP INDEX uk_name_pwd;
    ```

    

3. 主键约束

    主键约束相当于**唯一约束+非空约束的组合**

    ```sql
    create table 表名称(
        字段名 数据类型 primary key, #列级模式
        字段名 数据类型,
        字段名 数据类型 
    );
    ```

    建表后增加主键约束

    ```sql
    ALTER TABLE 表名称 
    ADD PRIMARY KEY(字段列表); 
    #字段列表可以是一个字段，也可以是多个字段，如果是多 个字段的话，是复合主键
    ```

    删除主键

    ```sql
    alter table 表名称 
    drop primary key;
    ```

4. 自增列

    建立表时添加

    ```sql
    create table employee(
        eid int primary key auto_increment,
        ename varchar(20)
    );
    ```

    建立表后添加

    ```sql
    alter table 表名称 
    modify 字段名 数据类型 auto_increment;#给这个字段增加自增约束
    ```

    删除

    ```sql
    alter table 表名称 
    modify 字段名 数据类型; #去掉auto_increment相当于删除
    ```

5. 外键约束<span style="color:red">（阿里：不得使用外键与级联，一切外键概念必须在应用层解决。）</span>

    建表时添加外键

    ```sql
    create table 从表名称(
        字段1 数据类型 primary key,
        字段2 数据类型,
        [CONSTRAINT <外键约束名称>] FOREIGN KEY（从表的某个字段) references 主表名(被参考字段)
    );
    #(从表的某个字段)的数据类型必须与主表名(被参考字段)的数据类型一致，逻辑意义也一样 
    #(从表的某个字段)的字段名可以与主表名(被参考字段)的字段名一样，也可以不一样
    ```

    建表后添加

    ```sql
    ALTER TABLE 从表名 ADD [CONSTRAINT 约束名] FOREIGN KEY (从表的字段) REFERENCES 主表名(被引用 字段) [on update xx][on delete xx];
    ```

    约束等级

    - `Cascade`方式 ：在父表上update/delete记录时，同步update/delete掉子表的匹配记录
    - Set null方式 ：在父表上update/delete记录时，将子表上匹配记录的列设为null，但是要注意子表的外键列不能为not null
    - `No action`方式 ：如果子表中有匹配的记录，则不允许对父表对应候选键进行update/delete操作
    - `Restrict`方式 ：同no action， 都是立即检查外键约束
    - Set default方式 （在可视化工具SQLyog中可能显示空白）：父表有变更时，子表将外键列设置成一个默认的值，但Innodb不能识别
    - 默认：如果没有指定等级，就相当于Restrict方式。
    - 建议：对于外键约束，最好是采用: ON UPDATE CASCADE ON DELETE RESTRICT 的方式。

    例子：

    ```sql
    create table dept(
        did int primary key, #部门编号
        dname varchar(50) #部门名称
    );
    
    create table emp(
        eid int primary key, #员工编号
        ename varchar(5), #员工姓名
        deptid int, #员工所在的部门
    	foreign key (deptid) references dept(did) on update cascade on delete set null #把修改操作设置为级联修改等级，把删除操作设置为set null等级
    );
    ```

    删除

    ```sql
    ALTER TABLE 从表名 DROP INDEX 索引名;
    ```

6. CHECK 约束(MySQL 8.0中可以使用check约束了)

7. .DEFAULT约束

    建表时

    ```sql
    create table 表名称(
        字段名 数据类型 primary key,
        字段名 数据类型 unique key not null,
        字段名 数据类型 unique key,
        字段名 数据类型 not null default 默认值,
    );
    ```

    建表后

    ```sql
    alter table 表名称 modify 字段名 数据类型 default 默认值;
    ```

    删除

    ```sql
    alter table 表名称 modify 字段名 数据类型 ;#删除默认值约束，也不保留非空约束
    ```

    

8. 查看约束 和 索引

    ```sql
    SELECT * 
    FROM information_schema.table_constraints 
    WHERE table_name = '表名'; #查看都有哪些约束
    ```

    ```sql
    SHOW INDEX FROM 表名称; #查看某个表的索引名
    ```

    

## 3. `DML`数据操作语言

用于添加、删除、更新和查询数据库记录，并检查数据完整性。

主要的语句关键字包括 INSERT 、 DELETE 、 UPDATE 、 SELECT 等

1. 插入记录

    ```sql
    INSERT INTO 表名(column1 [, column2, …, columnn]) 
    VALUES 
    (value1 [,value2, …, valuen]), 
    (value1 [,value2, …, valuen]), 
    ……
    (value1 [,value2, …, valuen]);
    ```

2. 更新记录

    ```sql
    UPDATE table_name 
    SET column1=value1, column2=value2, … , column=valuen 
    [WHERE condition]
    ```

3. 删除记录

    ```sql
    DELETE FROM table_name 
    [WHERE <condition>];
    ```

新特性：**计算列**

```sql
CREATE TABLE tb1(
    id INT,
    a INT, 
    b INT, 
    c INT GENERATED ALWAYS AS (a + b) VIRTUAL 
);
```



## 4. `DCL`数据控制语言

用于定义数据库、表、字段、用户的访问权限和安全级别。

主要的语句关键字包括 GRANT 、 REVOKE 、 COMMIT 、 ROLLBACK 、 SAVEPOINT 等



## 5. `TCL`事务控制语言





