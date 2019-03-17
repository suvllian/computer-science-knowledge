### SQL查询

#### 将数据库查询结果转换为显示自定义字段

在开发时经常会遇到数据中用单字符保存简单信息，如性别、状态等。如果需要在查询时展示对应的字段，比如性别：1表示男，0表示女。

**Basic Usage**

将state字段进行转换：0、1、2分别转换为正常、删除、禁用

``` sql
SELECT name, age, sex, CASE WHEN state = 0 THEN '正常' WHEN state = 1 THEN '删除' ELSE '禁用' END AS state
from student
```