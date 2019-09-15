## 字符编码

### 个人总结
* 字符集：一个系统支持的所有抽象字符的集合。ASCII字符集、GB2312字符集、Unicode字符集。
* 字符编码：一套将自然语言与其他东西结合配对的规则。UTF-8编码。

Unicode字符集将世界上所有符号都纳入其中。  
UTF-8是可变长编码，可以节省存储大小，是互联网中使用最多的对Unicode的实现方式。

**为什么会出现乱码？**

使用了不同的编解码方式。例如使用UTF-8编码，却使用GBK解码，就会导致乱码。

### 参考文章
* [十分钟搞清字符集和字符编码](http://cenalulu.github.io/linux/character-encoding/)
* [ASCII，Unicode，GBK和UTF-8字符编码的区别和联系](https://www.cnblogs.com/geons/p/9352256.html)