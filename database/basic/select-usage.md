## SQL查询

### 1. 将数据库查询结果转换为显示自定义字段

在开发时经常会遇到数据中用单字符保存简单信息，如性别、状态等。如果需要在查询时展示对应的字段，比如性别：1表示男，0表示女。

**Basic Usage**

将state字段进行转换：0、1、2分别转换为正常、删除、禁用

``` sql
SELECT name, age, sex, CASE WHEN state = 0 THEN '正常' WHEN state = 1 THEN '删除' ELSE '禁用' END AS state
from student
```

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

用例如下：

``` sql
drop table if exists tmp_student_count_table;
create table tmp_student_count_table as
select classname as 班级
    ,malecount as 男生数量
    ,femalecount as 女生数量

select 班级
    ,男生数量
    ,女生数量
    from (
        select * from tmp_student_count_table
        union all
        select '全校' as 班级
            ,sum(malecount) as 男生数量
            ,sum(femalecount) as 女生数量
        from tmp_student_count_table
    ) t1;
```
