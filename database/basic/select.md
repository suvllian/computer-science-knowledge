### SQL查询

#### 1. 将数据库查询结果转换为显示自定义字段

在开发时经常会遇到数据中用单字符保存简单信息，如性别、状态等。如果需要在查询时展示对应的字段，比如性别：1表示男，0表示女。

**Basic Usage**

将state字段进行转换：0、1、2分别转换为正常、删除、禁用

``` sql
SELECT name, age, sex, CASE WHEN state = 0 THEN '正常' WHEN state = 1 THEN '删除' ELSE '禁用' END AS state
from student
```

#### 2. 根据查询结果进行分组

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

#### 3. if语句

if语句用法如下，如果expr1成立返回value1，否则返回value2，可以用`case when`语句实现同样的需求。
``` sql
if (expr1, value1, value2)
```

``` sql
select if (sex = 1, '男', '女') as sex from student
```

#### 4. 使用正则表达式

``` sql
SELECT vend_name FROM vendors WHERE vend_name REGEXP '.'
```

#### 5. 联结查询

联结不是物理实体，它在实际的物理表中并不存在，存在于执行的查询当中。

##### 5.1 内联结
内联结是给予两个表的等值联结

``` sql
SELECT vend_name, prod_name, prod_price FROM verdors INNER JOIN products ON vendors.vend_id = products.vend_id
```

等价于

``` sql
SELECT vend_name, prod_name, prod_price FROM verdors, products WHERE vendors.vend_id = products.vend_id
```

##### 5.2 自联结

``` sql
select p1.prod_id, p1.prod_name from products as p1, products as p2 where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR';
```

等价于自查询
```sql
select prod_id, prod_name from products where vend_id = (select vend_id from products where prod_id = 'DTNTR');
```

##### 5.3 外联结
许多联结将一个表中的行与另一个表中的行相关联，但有时候会需要包含没有关联行的那些行，这种联结方式叫外联结。

在使用`OUTER JOIN`时，必须使用`RIGHT`或`LEFT`关键字制定包括其所有行的表。（right指的是outer join右边的表，left指的是outer join左边的表）

#### 6. 组合查询
UNION中的每个查询必须包含相同的列、表达式或聚集函数；UOION必须由两条或者两条以上的SELECT语句组成。

``` sql
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price > 5
UNION
SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002)
```