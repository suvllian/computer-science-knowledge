## 面向切面编程

用过express、koa或者redux的同学应该都知道它们都有**“中间件”**(前端意义上的中间件，不是指平台与应用之间的通用服务)这样一个概念，在redux中我们可以通过中间件的方式使用`redux-thunk`和`loger`的功能，在koa中我们可以通过中间件对请求上下文`context`进行处理。

通过中间件，我们可以在一些方法执行前添加统一的处理（如登录校验，打印操作日志等），中间件的设计思想都是面向切面编程的思想，把一些跟业务无关的逻辑进行抽离，在需要使用的场景中再切入，降低耦合度，提高可重用性，而且使代码更简洁。

下面我们通过源码分析redux和koa是如何实现中间件的，最后详细介绍**面向切面编程**解决的一些问题。

### 一、redux中间件

我们先来看下redux中间件的用法。redux暴露了`applyMiddleware`方法，接受一个函数数组作为参数，函数的返回值作为第二参数传入`createStore`。

``` javascript
import { createStore, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { createLogger } from 'redux-logger'
import reducer from './reducers'

const middleware = [ thunk ]
if (process.env.NODE_ENV !== 'production') {
  middleware.push(createLogger())
}

const store = createStore(
  reducer,
  applyMiddleware(...middleware)
)
```

接下来看redux中间件是怎么实现的，下面是applyMiddleware的源码，其中对dispatch方法进行了compose处理，处理后的dispatch在每次调用时都会链式调用中间件函数。

``` javascript
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

compose的源码如下，其主要作用是依次调用函数数组中的中间件函数，并将前一个函数的返回结果作为后一个函数的入参，从而实现了在调用`dispath`时调用其他方法的需求。

``` javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

### 二、koa中间件

同样的，我们来看下koa中间件的用法。首先实例化一个`Koa`对象，然后通过对象中的`use`方法添加中间件函数，最后调用`listen`方法启动node服务器

``` javascript
const app = new Koa();
const logger = async function(ctx, next) {
  let res = ctx.res;

  // 拦截操作请求 request
  console.log(`<-- ${ctx.method} ${ctx.url}`);

  await next();

  // 拦截操作响应 request
  res.on('finish', () => {
    console.log(`--> ${ctx.method} ${ctx.url}`);
  });
};

app.use(logger)

app.use((ctx, next) => {
  ctx.cookies.set('name', 'jon');
  ctx.status = 204;

  await next(); 
});

app.use(async(ctx, next) => {
  ctx.body = 'hello world';
})

const server = app.listen();
```

我们来看下koa中间件的实现，我们通过实例的`use`方法添加中间件，在调用实例的`listen`方法后，在`callback`中会对中间件进行`compose`处理，最后在`handleRequest`中调用处理过的中间件。

``` javascript
listen(...args) {
  const server = http.createServer(this.callback());
  return server.listen(...args);
}

use(fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  if (isGeneratorFunction(fn)) {
    fn = convert(fn);
  }
  this.middleware.push(fn);
  return this;
}

callback() {
  const fn = compose(this.middleware);

  if (!this.listenerCount('error')) this.on('error', this.onerror);

  const handleRequest = (req, res) => {
    const ctx = this.createContext(req, res);
    return this.handleRequest(ctx, fn);
  };

  return handleRequest;
}

handleRequest(ctx, fnMiddleware) {
  const res = ctx.res;
  res.statusCode = 404;
  const onerror = err => ctx.onerror(err);
  const handleResponse = () => respond(ctx);
  onFinished(res, onerror);
  return fnMiddleware(ctx).then(handleResponse).catch(onerror);
}
```

koa中的`compose`方法和redux中的`compose`方法原理是一样的，都是用洋葱模型的方式依次调用中间件函数，但是`koa-compose`是通过Promise的方式实现的。我们来看下koa中`compose`方法的实现。

``` javascript
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

### 三、面向切面编程

通过对redux和koa中间件实现的简单分析，大家应该对面向切面编程有了一个简单的理解，下面一部分内容就是详细介绍面向切面编程，以及如果不用面向切面编程的方式，我们还能用什么方式来满足需求，以及这些方式有什么问题。

> 面向切面编程(Aspect Oriented Programming，也叫面向方面编程)是一种非侵入式扩充对象、方法和函数行为的技术。

其核心思想是通过对方法的拦截，在预编译或运行时进行动态代理，实现在方法被调用时可以以对业务代码无侵入的方式添加功能。

比如像日志、事务等这些功能，和核心业务逻辑没有直接关联，通过切面的方式和核心业务逻辑进行剥离，让业务同学只需关心业务逻辑的开发，当需要用到这些功能的时候就把切面插拔到业务流程的某些点上，做到了切面和业务的分离。

我们举个例子，来帮助理解面向切面编程的使用场景。

农场的水果包装流水线一开始只有**采摘-清洗-贴标签**三步。

![](./images/aop1.png)

为了提高销量，想加上两道工序**分类**和**包装**但又不能干扰原有的流程，同时如果没增加收益可以随时撤销新增工序。

![](./images/aop2.png)

最后在流水线的中的空隙插上两个工人去处理，形成**采摘-分类-清洗-包装-贴标签**的新流程，而且工人可以随时撤回。

回到AOP的作用这个问题。

> AOP就是在现有代码程序中，在不影响原有功能的基础上，在程序生命周期或者横向流程中加入/减去一个或多个功能。

我们通过一个实际问题来分析AOP的好处。

现在有一个类`Foo`，类中包含了方法`doSomething`，我想在每次方法`doSomething`执行前和执行后打印一段日志，想实现的效果如下：

``` javascript
class Foo {
  doSomething() {
    let result;

    // dosomething

    return result
  }
}

const foo = new Foo()
foo.doSomething(1, 2)

// before doSomething
// after doSomething, result: result
``` 

#### 3.1 修改源代码

最简单粗暴的方法，就是重写`doSomething`方法

``` javascript
class Foo {
  doSomething() {
    console.log(`before doSomething`)
    let result;

    // dosomething

    console.log(`after doSomething, result: ${result}`)
    return result
  }
}

const foo = new Foo()
foo.doSomething(1, 2)

// before doSomething
// after doSomething, result: result
```

这样的坏处很明显，需要改动原有的代码，是侵入性最强的一种做法。如果代码逻辑复杂，修改代码也会变得困难。如果想用类似的方法分析其他方法，同样需要修改其他方法的源代码。

#### 3.2 继承
``` javascript
class Bar extends Foo {
  doSomething () {
    console.log(`before doSomething`)

    const result = super.doSomething.apply(this, arguments)

    console.log(`after doSomething, result: ${result}`)

    return result
  }
}

const bar = new Bar()
bar.doSomething(1, 2)

// before doSomething
// after doSomething, result: result
```

用继承的方式避免了修改父类的源代码，但是每个使用`new Foo`的地方都要改成`new Bar`。

#### 3.3 重写类方法

``` javascript
class Foo {
  doSomething() {
    let result;

    // dosomething

    return result
  }
}

const _doSomething = Foo.prototype.doSomething
Foo.prototype.doSomething = function() {
  if (_doSomething) {
    console.log(`before doSomething`)

    const result = _doSomething.apply(this, arguments)

    console.log(`after doSomething, result: ${result}`)
    
    return result
  }
}

const foo = new Foo()
foo.doSomething(1, 2)

// before doSomething
// after doSomething, result: result
```

这样就多了中间变量`_doSomething`，管理需要成本。

#### 3.4 职责链模式

我们可以通过在Function的原型上添加before和after函数来满足我们的需求。

``` javascript 
Function.prototype.before = function (fn) {
  var self = this;
  return function () {
    fn.apply(this, arguments);
    self.apply(this, arguments);
  }
}
Function.prototype.after = function (fn) {
  var self = this;
  return function () {
    const result = self.apply(this, arguments);
    fn.call(this, result);
  }
}

class Foo {
  doSomething() {
    let result;

    // dosomething
    result = 1

    return result
  }
}

const foo = new Foo()
foo.doSomething.after((result) => {
  console.log(`after doSomething, result: ${result}`)
}).before(() => {
  console.log('before doSomething')
})()

// before doSomething
// after doSomething, result: 1
```

从代码上已经实现了完全解耦，也没有中间变量，但是却有一长串的链式调用，如果处理不当，代码可读性及可维护性较差。

#### 3.5 中间件

我们可以借用中间件思想来分解前端业务逻辑，通过next方法层层传递给下一个业务。首先要有个管理中间件的对象，我们先创建一个名为Middleware的对象：

``` javascript
function Middleware(){
  this.cache = [];
}
Middleware.prototype.use = function (fn) {
  if (typeof fn !== 'function') {
    throw 'middleware must be a function';
  }
  this.cache.push(fn);
  return this;
}

Middleware.prototype.next = function (fn) {
  if (this.middlewares && this.middlewares.length > 0) {
    var ware = this.middlewares.shift();
    ware.call(this, this.next.bind(this));
  }
}
Middleware.prototype.handleRequest = function () {
  this.middlewares = this.cache.map(function (fn) {
    return fn;
  });
  this.next();
}
var middleware = new Middleware();
middleware.use(function (next) {
  console.log(1); next(); console.log('1结束');
});
middleware.use(function (next) {
  console.log(2); next(); console.log('2结束');
});
middleware.use(function (next) {
  console.log(3); console.log('3结束');
});
middleware.use(function (next) {
  console.log(4); next(); console.log('4结束');
});
middleware.handleRequest();

// 输出结果：
// 1
// 2
// 3
// 3结束
// 2结束
// 1结束
```

### 四、总结

本文一开始简单分析了redux及koa中间件的实现方式，然后介绍了面向切面编程，通过一个例子介绍不同的方式实现对方法的扩充，最后我们能明显的体会到面向切面编程的方式能降低代码耦合度，提高可重用性，而且使代码更简洁。

### References
* [koa-compose源代码](https://github.com/koajs/compose/blob/master/index.js)
* [redux中compose的实现](https://github.com/reduxjs/redux/blob/master/src/compose.js)
* [Koa.js的AOP设计](https://chenshenhai.github.io/koajs-design-note/note/chapter02/01.html)
* [编写可维护代码之“中间件模式”](https://zhuanlan.zhihu.com/p/26063036)
* [使用JavaScript拦截和跟踪浏览器中的HTTP请求](https://www.ibm.com/developerworks/cn/web/wa-lo-jshttp/index.html)