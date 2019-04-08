## 其他函数

### 1. 数据类型转换

#### 1.1 cast(value, type)

The type for the result can be one of the following values:

* 二进制: BINARY
* 字符型: CHAR
* 日期: DATE
* 时间: TIME
* 日期时间型: DATETIME
* 浮点数: DECIMAL
* 整数: SIGNED
* 无符号整数: UNSIGNED

**Basic Usage**

``` sql
SELECT CAST(prod_code AS INT) FROM product;
```

#### convert