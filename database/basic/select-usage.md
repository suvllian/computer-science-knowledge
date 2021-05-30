## SQL查询场景

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

### 3. 统计数量
#### 3.1 计算总计数量

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
#### 3.2 统计一张表多个状态的数量

``` sql
SELECT COUNT(*)  AS total,
       COUNT(if(mdt_apply_type= '1', 1, NULL)) AS single_diseases_num,
       COUNT(if(mdt_apply_type= '2', 1, NULL)) AS multidisciplinary_num
  FROM t_mdt_apply_list
 WHERE audit_opinion= '3'
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
        FROM    (
                SELECT  peopleId
                FROM    people
                GROUP BY peopleId HAVING COUNT(peopleId) >1
                ) repeat_people
)
AND     id NOT IN (
        SELECT  id
        FROM    (
                SELECT  max(id)
                FROM    people
                GROUP BY peopleId HAVING COUNT(peopleId) >1
                ) max_id
);
```

#### 4.3 更新表中某一字段重复的记录，只保留最新记录
``` sql
UPDATE people SET value = 1
WHERE   peopleId IN (
        SELECT  peopleId
        FROM    (
                SELECT  peopleId
                FROM    people
                GROUP BY peopleId HAVING COUNT(peopleId) >1
                ) repeat_people
)
AND     id NOT IN (
        SELECT  id
        FROM    (
                SELECT  max(id)
                FROM    people
                GROUP BY peopleId HAVING COUNT(peopleId) >1
                ) max_id
);
```

**需要特别注意，在更新表时不能通过自查询再更新，需要通过中间表操作。**  

参考：
* [You can't specify target table for update in FROM clause解决方法](https://blog.csdn.net/fdipzone/article/details/52695371)
* [MYSQL删除重复数据](https://juejin.im/post/5ab4df3151882555635e4363#heading-0)


### 5. 添加虚拟字段

``` sql
SELECT  '品牌商' AS user_type
        ,name
FROM    user_info;
```

### 6. 将SQL查询结果行记录转换为列属性

[参考: sql查询结果的行记录转换为列属性](https://blog.csdn.net/Agly_Clarlie/article/details/87978445)

需要把查询结果进行转换，按周访问次数进行Group，用户类型作为列名

| week_visit_time  | visit_count | user_type |
| ------------- | ------------- | ------------- |
| 1  | 20  | supplier |
| 1  | 30  | brandOwner
| 2  | 60  | supplier | |
| 2  | 50  | brandOwner |

使用`CASE THEN`语句进行转换

``` sql
SELECT  week_visit_time
        ,SUM(
            CASE    WHEN user_type = 'brandOwner' THEN visit_count 
                    ELSE 0 
            END
        ) AS brandOwner
        ,SUM(CASE WHEN user_type = 'supplier' THEN visit_count ELSE 0 END) AS supplier
FROM    tmp_merchant_type_visit_table
GROUP BY week_visit_time
```

转换结果

| week_visit_time  | supplier | brandOwner |
| ------------- | ------------- | ------------- |
| 1  | 20  | 30 |
| 2  | 60  | 50 |


### 7. 分组查询。根据一个或多个字段进行分组，并只取每一组符合特定条件的记录作为结果

[参考：MySQL组内排序问题：分组查询每组的前n条记录](https://www.jianshu.com/p/717c4bdad462)

对于`pv_table`数据，需要查询每个URL访问次数最大的记录

| id | url  | visit_count | date |
| ------------- | ------------- | ------------- | ------------- |
| 3 | suvllian.com  | 20  | 20200301 |
| 5 | suvllian.com  | 30  | 20200301 |
| 7 | suvllian.com  | 40  | 20200301 |
| 9 | suvllian.net  | 30  | 20200301 |
| 12 | suvllian.net  | 30  | 20200301 |
| 16 | suvllian.net  | 40  | 20200301 |

使用临时表关联查询，如果直接使用MAX查询，除了MAX函数中的字段，其他字段值是随机的，不是max记录的值，详见参考文章。

``` sql
SELECT  *
FROM    pv_table
WHERE   id IN( SELECT pv_table.id FROM pv_table,( SELECT url, MAX(pv) AS maxPv FROM `pv_table` GROUP BY url) max_pv WHERE pv_table.url = max_pv.url AND pv_table.pv = max_pv.maxPv)
```

### 8. 一次更新多行多字段

将满足条件的记录对应字段更新成相同的值
``` sql
UPDATE mytable SET myfield = 'value' WHERE other_field = 'other_value';
```

将满足条件的记录的字段更新成不同的值
``` sql
UPDATE mytable
 SET myfield = CASE   other_field
 WHEN 1 THEN 'value'
 WHEN 2 THEN 'value'
 WHEN 3 THEN 'value'
END
 WHERE id IN (1,2,3)
```

参考：[如何在数据库中一次更新多行多字段](http://www.renniaofei.com/code/update-multiple-rows-with-different-values-and-a-single-sql-query.html)

### 9. A/B表为1对多关系，联合查询B表只提取第一条与A记录关联的记录
``` sql
SELECT  username
FROM    (
            SELECT  t1.username
                    ,row_number() OVER (PARTITION BY t1.user_id) AS group_idx
            FROM    t1
                    ,t2
            WHERE   t2.user_address_id = 1
            AND     t1.user_id = t2.user_id
        ) s
WHERE   s.group_idx = 1
;
```