# Oracle使用指南

此github旨在解决项目实施过程中遇到的一些共性问题。总结的难免有纰漏之处。如果有任何建议，还望各位读者指出。如果有所补充，也欢迎
新建一个branch或者邮箱联系我予以添加。本人邮箱wuboyu88@163.com

![oracle_logo](./image/oracle_logo.png)

## 展示方式说明
所有的展示都是通过以下三个部分组成：<br />
   1、问题 <br />
   2、代码or解决方案 <br />
   3、参考资料 <br />
   
##
Q1: oracle不像其他sql语言一样，可以用drop table if exists table_name这种方式删表。
那么在oracle中如何实现类似的功能呢？<br />
Code:
```sql
BEGIN
    EXECUTE IMMEDIATE 'drop table table_name';
EXCEPTION
    WHEN OTHERS THEN
      IF SQLCODE <> -942 THEN
        RAISE;
      END IF;
END; 
```
References:<br />
https://stackoverflow.com/questions/1799128/oracle-if-table-exists

##
Q2: oracle建表和插入数据顺序需保持一致<br />
Code:
```sql
-- 建表(字段顺序为a->b)
CREATE TABLE table1
  (
     a NUMBER,
     b NUMBER
  );

/*错误的插表方式*/
-- 插入数据(字段顺序为b->a,与建表时不一致)
INSERT INTO table1
SELECT b,
       a
FROM   table2;

COMMIT;

/*正确的插表方式*/
-- 插入数据(字段顺序为a->b,与建表时一致)
INSERT INTO table1
SELECT a,
       b
FROM   table2;

COMMIT;    
```
References:<br />
Not found

##
Q3: oracle如何解决plsql中保存的.sql文件中文注释为乱码的情况？<br />
Solution:<br />
configure --> preferences --> files --> format --> encoding --> always UTF8<br />
References:<br />
https://www.jianshu.com/p/1ecb687d1433

##
Q4: oracle报"ORA-00054 资源正忙，但指定以 NOWAIT 方式获取资源，或者超时失效"问题如何解决？<br />
Code:
```sql
-- 1.找出引发锁的会话和表名 
SELECT t1.session_id, 
       t2.owner, 
       t2.object_name, 
       t3.username, 
       t3.sid, 
       t3.serial#, 
       t3.logon_time 
FROM   v$locked_object t1 
       JOIN dba_objects t2 
         ON t1.object_id = t2.object_id 
       JOIN v$session t3 
         ON t1.session_id = t3.sid 
ORDER  BY t3.logon_time; 

-- 2.赋予username权限 
GRANT ALTER SYSTEM TO upro_test; --对应上面的username 

-- 3.杀掉会话 
ALTER SYSTEM KILL SESSION 'sid,serial#'; --对应上面的sid和serial# 
```
References:<br />
https://blog.csdn.net/deniro_li/article/details/81085758 <br />
https://blog.csdn.net/hehuyi_in/article/details/89669553

##
Q5: oracle表空间查询<br />
Code:
```sql
-- 查看单表占用空间(注意表名需大写) 
SELECT segment_name, 
       bytes / ( 1024 * 1024 * 1024 ) AS space_gb 
FROM   user_segments 
WHERE  segment_name = 'S0CBI_P1_RESULT'; 

-- 查看剩余表空间
SELECT SUM(bytes) / ( 1024 * 1024 * 1024 ) AS space_gb 
FROM   dba_free_space 
WHERE  tablespace_name = 'UPRO';
```
References:<br />
https://www.cnblogs.com/pejsidney/p/8057372.html

##
Q6: oracle数据库无法匹配中文<br />
Solution:<br />
添加环境变量<br />
变量名：NLS_LANG<br />
变量值：SIMPLIFIED CHINESE_CHINA.ZHS16GBK<br />
References:<br />
https://www.cnblogs.com/1201x/p/6513681.html

##
Q7: oracle得到指定日期的上个月最后一天<br />
Code:
```sql
SELECT LAST_DAY(ADD_MONTHS(yourdate,-1))
```
References:<br />
https://stackoverflow.com/questions/4957224/getting-last-day-of-previous-month-in-oracle-function

##
Q8: oracle存储过程默认参数如何设置<br />
Code:
```sql
CREATE OR REPLACE PROCEDURE my_procedure(cutoff_date IN VARCHAR2 DEFAULT '2020-05-31',
                                         nb_days     IN NUMBER DEFAULT 7)
```
References:<br />
https://oracle-patches.com/en/databases/sql/3777-pl-sql-procedure-setting-default-parameter-values

##
Q9: oracle建表之后加主键约束<br />
Code:
```sql
ALTER TABLE tablename
  ADD CONSTRAINT pk_id PRIMARY KEY (id); 
```
References:<br />
https://blog.csdn.net/yuan_csdn1/article/details/78158247

##
Q10: oracle自定义函数性能优化<br />
Solution: oracle通过关键字DETERMINISTIC来表明一个函数（Function）是确定性的，确定性函数可以用于创建基于函数的索引。
因此自定义函数加了DETERMINISTIC关键字可以提高执行效率。<br />
Code:
```sql
/*自定义get_last_day_n_month获得前n个月的最后一天*/ 
CREATE OR REPLACE FUNCTION get_last_day_n_month(date_str IN VARCHAR2, 
                                                n        IN NUMBER) 
  RETURN DATE DETERMINISTIC IS 
  res DATE; 
BEGIN 
  res := LAST_DAY(ADD_MONTHS(TO_DATE(date_str, 'YYYY-MM-DD'), -n)); 
  RETURN res; 
END;
```
References:<br />
https://www.cnblogs.com/kerrycode/p/9099507.html

##
Q11: oracle斜杠(/)的使用场景<br />
Solution: 我们都知道sql语句可以以分号(;)结尾，通常情况下这么做都没有任何问题。
但是当我们想批量执行代码块时，比如想批量执行100个建表语句或者存储过程时，如果不加
斜杠(/)就会只执行第一个代码块，后续无法执行。因此建议用斜杠(/)对工作单元进行隔离。
<br />
Code:
```sql
create OR replace PROCEDURE procedure1 AS 
BEGIN 
... 
END;
/
CREATE OR replace PROCEDURE procedure2 AS 
BEGIN 
... 
END;
/
```
References:<br />
https://stackoverflow.com/questions/1079949/when-do-i-need-to-use-a-semicolon-vs-a-slash-in-oracle-sql



## License
[MIT](https://choosealicense.com/licenses/mit/)