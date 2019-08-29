## 职责链模式

### 使用场景
1、有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。 
2、在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
3、可动态指定一组对象处理请求。

### 优缺点
* 优点
  * 降低耦合度。它将请求的发送者和接收者解耦。 
  * 简化了对象。使得对象不需要知道链的结构。 
  * 增强给对象指派职责的灵活性

* 缺点
  * 不能保证请求一定被接收。 
  * 系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。

### 示例
* [职责链模式的PHP实现](https://github.com/suvllian/learning/tree/master/design-patterns/behavioral-patterns/chain-of-responsibility-pattern)

### 文档