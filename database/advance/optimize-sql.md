## 慢SQL优化

### 1. 理论知识
通过建立索引优化，建立索引原则：

* 最左前缀匹配原则
* =和in可以乱序
* 尽量选择区分度高的列作为索引
* 索引列不能参与计算
* 尽量的扩展索引，不要新建索引

### 2. 实践

#### 2.1 反向查询不能使用索引

not in/not exists都不是好习惯

**bad case:**
``` sql
select * from order where status!=0 and stauts!=1
```

**good case:**
``` sql
select * from order where status in(2,3)
```

#### 2.2 前导模糊查询不能使用索引

非前导模糊查询可以使用索引

**bad case:**
``` sql
select * from order where desc like '%XX'
```

**good case:**
``` sql
select * from order where desc like 'XX%'
```

#### 2.3 在属性上进行计算不能命中索引

**bad case:**
``` sql
select * from order where YEAR(date) < = '2020'
```

**good case:**
``` sql
select * from order where date < = CURDATE()
```

#### 2.4 如果明确知道返回结果记录数量，使用limit语句能够提高效率

**bad case:**
``` sql
select * from user where login_name=?
```

**good case:**
``` sql
select * from user where login_name=? limit 1
```

#### 2.5 把计算放到业务层而不是数据库层，除了节省数据的CPU，还有意想不到的查询缓存优化效果

**bad case:**
``` sql
select * from order where date < = CURDATE()
```

**good case:**
``` php
$curDate = date('Y-m-d');
$res = mysql_query(
    'select * from order where date < = $curDate'
);
```

#### 2.6 强制类型转换会全表扫描


#### 2.7 插入大量数据时，考虑批量插入


#### 2.8 不要有超过5个以上的表连接

### 3. 参考
* [MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)