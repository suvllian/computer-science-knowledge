## 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。

### 使用场景
防止一个全局使用的类频繁地创建与销毁。

### 优缺点
* 优点
  * 在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例
  * 避免对资源的多重占用（比如写文件操作）

* 缺点
  * 没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。

### 示例
* [单例模式的Javascript实现](https://github.com/suvllian/learning/tree/master/design-patterns/creational-patterns/singleton)

### 文档