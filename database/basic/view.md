## 视图
视图是虚拟的表，只包含使用时动态检索数据的查询。

使用视图的优点：

* 复用sql。
* 简化复杂的sql操作。
* 使用表的组成部分而不是整个表。
* 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
* 更改数据格式和表示。

视图的规则和限制：

* 视图命名必须唯一。
* 视图不能索引，不能有关联的触发器或者默认值。
* 视图可以和表一起使用。

``` sql
create view productcustomers as 
select cust_name, cust_contact, prod_id from customers, orders, orderitems 
where customers.cust_id = orders.cust_id and orderitems.order_num = orders.order_name
```