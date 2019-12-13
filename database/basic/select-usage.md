## SQL查询

### 1. 将数据库查询结果转换为显示自定义字段

在开发时经常会遇到数据中用单字符保存简单信息，如性别、状态等。如果需要在查询时展示对应的字段，比如性别：1表示男，0表示女。

**Basic Usage**

将state字段进行转换：0、1、2分别转换为正常、删除、禁用

``` sql
SELECT  name
        ,age
        ,sex
        ,CASE    WHEN state = 0 THEN '正常'
                 WHEN state = 1 THEN '删除' 
                 ELSE '禁用' 
         END AS state
FROM    student

```

原来的结果：

| orderId  | state | 
| ------------- | ------------- |
| 1 | 0  | 
| 2  | 1  | 
| 3  | 2  | 

转换后的结果更容易理解

| orderId  | state | 
| ------------- | ------------- |
| 1 | 正常  | 
| 2  | 删除  | 
| 3  | 禁用  | 

### 2. 根据查询结果进行分组

在开发时，可能需要根据某个字段的值进行分组，但不是简单的`group by`，而是需要一些逻辑判断。

使用`case when`可以得到我们的结果，同时需要注意的是，如果把一次查询结果作为下一次查询的输入，第一次查询结果需要**重命名**。

例如在下面的查询结果中，我们需要统计`userid`为0和不为0的订单量。

| userid  | username | ordertype |
| ------------- | ------------- | ------------- |
| 0  | jack  | 1 |
| 0  | jack  | 2 |
| 2  | tom  | 3 |

``` sql
SELECT 
    COUNT(*) AS 订单数量, 用户名称
    FROM (SELECT
        CASE WHEN userid = 0 THEN 'jack' ELSE '其他人' END AS 用户名称
        FROM cbulst.s_out_bound_order ) AS typeCount 
GROUP BY typeCount.用户名称
```

### 3. 计算总计数量

在日常数据分析过程中，我们查处按照某一维度区分的数据，但是还需要一个统计值作为参考。

将初步的查询结果作为临时表输出，在将临时表中所有数据进行累加输出一行与临时表中的数据进行联合查询。
`需要注意的是，在进行联合查询时，两个表的字段类型一定要一致`

用例如下：需要查询每个班级男生和女生的数量，并且在最后给出男生和女生的总数

| 班级  | malecount | femalecount |
| ------------- | ------------- | ------------- |
| 101  | 20  | 30 |
| 201  | 30  | 40 |
| 全校  | 50  | 70 |

``` sql
DROP TABLE IF EXISTS tmp_student_count_table ;

CREATE TABLE tmp_student_count_table AS
SELECT  classname AS 班级
        ,malecount AS 男生数量
        ,femalecount AS 女生数量;
SELECT  班级
        ,男生数量
        ,女生数量
FROM    (
            SELECT  *
            FROM    tmp_student_count_table
            UNION ALL
            SELECT  '全校' AS 班级
                    ,SUM(malecount) AS 男生数量
                    ,SUM(femalecount) AS 女生数量
            FROM    tmp_student_count_table
        ) t1
;
```

### 4. 处理某一字段重复的记录

#### 4.1 查询表中某一字段重复的记录

``` sql
SELECT  *
FROM    people
WHERE   peopleId IN (SELECT peopleId FROM people GROUP BY peopleId HAVING COUNT(peopleId) > 1)
```

#### 4.2 删除表中某一字段重复的记录，保留最新记录

``` sql
DELETE FROM people
WHERE   peopleId IN (
        SELECT  peopleId
        FROM    people
        GROUP BY peopleId HAVING COUNT(peopleId)>1
                    ) AND id NOT IN (
                            SELECT  max(id)
                            FROM    people
                            GROUP BY peopleId HAVING COUNT(peopleId)>1
                                       )
```

#### 4.3 更新表中某一字段重复的记录，只保留最新记录
**需要特别注意，在更新表时不能通过自查询再更新，需要通过中间表操作。**  

参考：[You can't specify target table for update in FROM clause解决方法](https://blog.csdn.net/fdipzone/article/details/52695371)


``` sql
UPDATE page_info SET delete_flag
                          = '1'
WHERE   url IN (
        SELECT  repeat_url
        FROM    (
                SELECT  url AS repeat_url
                FROM    page_info
                GROUP BY url HAVING COUNT(url)>1
                ) AS repeatUrl
               ) AND id NOT IN (
                       SELECT  max_id
                       FROM    (
                               SELECT  max(id) AS max_id
                               FROM    page_info
                               GROUP BY url HAVING COUNT(url)>1
                               ) AS maxId
                               )
```