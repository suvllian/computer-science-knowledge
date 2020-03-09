## SQL查询

### 1. if语句

if语句用法如下，如果expr1成立返回value1，否则返回value2，可以用`case when`语句实现同样的需求。
``` sql
if (expr1, value1, value2)
```

``` sql
select if (sex = 1, '男', '女') as sex from student
```

### 2. 使用正则表达式

``` sql
SELECT vend_name FROM vendors WHERE vend_name REGEXP '.'
```

### 3. 联结查询

联结不是物理实体，它在实际的物理表中并不存在，存在于执行的查询当中。

#### 3.1 内联结
内联结是给予两个表的等值联结，只有两个表中的记录满足条件才能返回结果。

两个表中都有vend_id字段，且字段值一样的记录才能返回。

``` sql
SELECT vend_name, prod_name, prod_price FROM verdors INNER JOIN products ON vendors.vend_id = products.vend_id
```

等价于

``` sql
SELECT vend_name, prod_name, prod_price FROM verdors, products WHERE vendors.vend_id = products.vend_id
```

#### 3.2 自联结

``` sql
select p1.prod_id, p1.prod_name from products as p1, products as p2 where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR';
```

等价于自查询
```sql
select prod_id, prod_name from products where vend_id = (select vend_id from products where prod_id = 'DTNTR');
```

#### 3.3 外联结
许多联结将一个表中的行与另一个表中的行相关联，但有时候会需要包含没有关联行的那些行，这种联结方式叫外联结。

在使用`OUTER JOIN`时，必须使用`RIGHT`或`LEFT`关键字指定包括其所有行的表。RIGHT JOIN指的是OUTER JOIN右边的表的内容，LEFT JOIN指的是OUTER JOIN左边的表。


### 4. 组合查询
UNION中的每个查询必须包含相同的列、表达式或聚集函数；UOION必须由两条或者两条以上的SELECT语句组成。

``` sql
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price > 5
UNION
SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001, 1002)
```

