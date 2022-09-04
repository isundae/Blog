---
title: Koa2框架原理及实现
date: 2022-07-18 12:14:29
tags: [typescript,koa,实现原理]
categories: typescript
cover: https://6973-isundae-3gbm0y3523031b2a-1258186980.tcb.qcloud.la/blog/202209041400101.png
---

# Koa2框架原理及实现

    [Koa2](https://koa.bootcss.com/)是一个基于Node实现的Web框架，特点是优雅、简洁、健壮、体积小、表现力强。它所有的功能通过插件的形式来实现。

    下面自己实现一个简单的Koa，通过这种方式来深入理解Koa原理，尤其是中间件部分的理解。Koa的具体实现可以看的[koa的源码](https://github.com/koajs/koa)。

```typescript
// koa 的简单使用
import Koa from 'koa'

const app = new Koa()
const port = 3000

app.use((ctx) => {
  ctx.body = 'hello koa'
})

app.listen(port, () => {
  console.log(`server is running at Http://localhost${port}`)
})
```

    启动应用，浏览器中打开 [http://localhost:3000/](http://localhost:3000/)

    通过上面的代码，如果要实现koa，需要实现三个模块，分别是http的封装，ctx对象的构建，中间件机制的实现，当然koa还实现了错误捕获和错误处理。

## 封装http模块

    通过阅读Koa2的[源码](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fkoajs%2Fkoa)可知Koa是通过封装原生的node http模块。

```typescript
// server.js
import http from 'http'
const port = 3000

const app = http
  .createServer((req, res) => {
    res.end('hello nodejs')
  })
  .listen(port, () => {
    console.log(`server is running at http://localhost:${port}`)
  })
```

    以上是使用Node.js创建一个HTTP服务的代码片段，关键是使用http模块中的createServer()方法，接下来对上面这面这部分过程进行一个封装。

    创建application.js，并创建一个Application类用于创建Koa实例。

    通过创建use()方法来注册中间件和回调函数。

    通过listen()方法开启服务监听实例，并传入use()方法注册的回调函数。

如下代码所示：

```typescript
// application.js

import http from 'http'

export default class Application {
  constructor() {
    this.callback = () => {}
  }

  use(callback) {
    this.callback = callback
    return this
  }

  listen(...args) {
    http
      .createServer((req, res) => {
        this.callback(req, res)
      })
      .listen(...args)
  }
}
```

    接下来创建一个server.js，引入application.js进行测试

```typescript
// server.js
import MiniKoa from './application.js'
const port = 3000
const app = new MiniKoa()
app
 .use((req, res) => {
 res.end('hello world')
 })
 .listen(port, () => {
 console.log(`server is running at http://localhost:${port}`)
 })
```

    启动后，在浏览器中输入 [http://localhost:3000/](http://localhost:3000/) 就能看到显示"hello world"。这样就完成http server的简单封装了。

## 构造ctx对象

    Koa 的 Context 把 Node 的 Request 对象和 Response 对象封装到单个对象中，并且暴露给中间件等回调函数。比如获取 url，封装之前通过req.url的方式获取，封装之后只需要ctx.url就可以获取。因此需要达到以下效果：

```typescript
app.use(async ctx => {
  ctx                             // 这是 Context
  ctx.request             // 这是 koa Request
  ctx.response             // 这是 koa Response
})
```

### JavaScript 的 getter 和 setter

    在此之前，需要了解 setter 和 getter 属性，通过 setter 和 getter 属性，可以自定义属性的特性。

```typescript
// test.js

let person = {
  _name: 'old name',
  get name() {
    return this._name
  },
  set name(val) {
    console.log('---修改name的值---')
    this._name = val
  }
}
                                                            // 输出：
console.log(person.name)      // old name
person.name = 'new name'      // ---修改name的值---
console.log(person.name)      // new name
```

### 构造 context

    使用 getter 和 setter 来构造 context:

```typescript
// application.js

import http from 'http'

export default class Application {
  constructor() {
    // this.callback = () => {}
    // 把 context、request 和 response 挂载到 Application 里面
    this.context = context
    this.request = request
    this.response = response
  }

  use(callback) {
    this.callback = callback
    return this
  }

  // 改造 listen
  listen(...args) {
    // http
    //   .createServer((req, res) => {
    //     this.callback(req, res)
    //   })
    //   .listen(...args)

    // 可能是一个 异步函数 因此需要 async
    http
      .createServer(async (req, res) => {
        const ctx = this.createCtx(req, res)
        // 此时就可以直接给callback一个 ctx
        await this.callback(ctx)
        ctx.res.end(ctx.body)
      })
      .listen(...args)
  }
  // 把原生的 req 和 res 挂载到 ctx 上
  createCtx(req, res) {
    // 模拟 req 和 res
    const ctx = Object.create(this.context) // 生成 context 对象，里面挂载 body 和 url
    ctx.request = Object.create(this.request) // 把 request 挂载到 ctx 上
    ctx.response = Object.create(this.response) // 把 response 挂载到 ctx 上
    // 把原生的 req 和 res 都挂载到 request 和 response 以及 ctx 上
    ctx.req = ctx.request.req = req
    ctx.res = ctx.response.res = res
    return ctx
  }
}

const request = {
  get url() {
    return this.req.url
  }
}

const response = {
  get body() {
    return this._body
  },
  set body(val) {
    this._body = val
  }
}

const context = {
  get url() {
    return this.request.url
  },
  get body() {
    return this.response.body
  },
  set body(val) {
    this.response.body = val
  }
}
```

    这时，就可以通过 ctx 来获取 url 了:

```typescript
// server.js
import MiniKoa from './application.js'
const port = 3000
const app = new MiniKoa()
// 此时可以使用 ctx
app
 .use(async (ctx) => {
 ctx.body = 'hello world!'
 })
 .listen(port, () => {
 console.log(`server is running at http://localhost:${port}`)
 })
```

## Koa中间件及洋葱圈模型的理解与实现

    koa的中间件机制是一个洋葱圈模型，通过use()注册多个中间件放入数组中，然后从外层开始往内执行，遇到next()后进入下一个中间件，当所有中间件执行完后，开始返回，依次执行中间件中未执行的部分，如上图所示。

    在实现之前，先来了解一下中间件的原理，根据中间件的原理可知，要层层递进执行多个函数，比如下面的例子：

```typescript
// test.js

const add = (x, y) => x + y
const double = (z) => z * 2

const res1 = add(1, 2)
const res2 = double(res1)
console.log(res2)         // 6
```

    上面的例子中，把add()函数的结果传入double()中，把函数作为参数，这样最终就会先执行add()然后执行double()，下面把这种模式编写成一个通用的compose()函数：

```typescript
// test.js

const add = (x, y) => x + y

const double = (z) => z * 2

const compose = (middleware) => {
  return (...args) => {
    let res = middleware[0](...args)
    for (let i = 1; i < middleware.length; i++) {
      res = middleware[i](res)
    }
    return res
  }
}

const fn = compose([add, double])
const res = fn(1, 2)
console.log(res)
```

    上面的compose()函数还有一个缺点，它是一个同步的方法，并没有异步的等待，如果要使用异步，直接使用for循环是不行的，它不能等待异步执行完毕。

    koa 对外暴露了next()方法来实现异步等待，它是一个Promise，当执行到它时就执行下一个中间件。

```typescript
// test.js
async function fn1(next) {
  console.log('fn1')
  await next()
  console.log('end fn1')
}

async function fn2(next) {
  console.log('fn2')
  await delay()
  await next()
  console.log('end fn2')
}

async function fn3(next) {
  console.log('fn3')
}

function delay() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    }, 2000)
  })
}

function compose(middleware) {
  // console.log(middleware)
  // [ [AsyncFunction: fn1], [AsyncFunction: fn2], [AsyncFunction: fn3] ]
  return () => {
    // 先执行第一个函数
    return dispatch(0)

    function dispatch(i) {
      let fn = middleware[i]
      // 如何不存在直接返回 Promise
      if (!fn) return Promise.resolve()
      // step1: 返回一个 Promise，因此单纯变成一个 Promise 且 立即执行
      // step2: 往当前中间件传入一个next()方法，当这个中间件有执行 next 的时候才执行下一个中间件
      return Promise.resolve(fn(function next() {
          // 执行下一个中间件
          return dispatch(i + 1)
        })
      )
    }
  }
}

const middleware = [fn1, fn2, fn3]
const finalFn = compose(middleware)
finalFn()

// fn1
// fn2
// 等待两秒
// fn3
// end fn2
// end fn1
```

    上面已经实现一个了一个简单的中间件示例，接下来把它整合到 Application 类中：

```typescript
// application.js

import http from 'http'

export default class Application {
  constructor() {
    // this.callback = () => {}
    // 把 context、request 和 response 挂载到 Application 里面
    this.context = context
    this.request = request
    this.response = response
    this.middleware = []
  }

  use(callback) {
    // 创建一个 middleware 数组，通过 push 传入多个 callback
    // 然后通过 compose 控制整个 middleware 执行的顺序
    // 每个 callback 回调函数给两个参数 第一个是 context 第二个是 next
    this.middleware.push(callback)
    // this.callback = callback
    return this
  }

  // 直接把 compose 移植过来
  compose(middleware) {
    // 每个中间件需要一个 context
    return function (context) {
      return dispatch(0)

      function dispatch(i) {
        let fn = middleware[i]
        if (!fn) return Promise.resolve()

        // 中间件第一个参数是一个 context，第二个参数是 next()
        return Promise.resolve(
          fn(context, function next() {
            return dispatch(i + 1)
          })
        )
      }
    }
  }

  // 改造 listen
  listen(...args) {
    // http
    //   .createServer((req, res) => {
    //     this.callback(req, res)
    //   })
    //   .listen(...args)

    // 可能是一个 异步函数 因此需要 async
    http
      .createServer(async (req, res) => {
        const ctx = this.createCtx(req, res)
        // await this.callback(ctx)
        // 这里不能直接执行 callback 而是先获取经过 compose 处理后的中间件集合
        const fn = this.compose(this.middleware)
        await fn(ctx)
        ctx.res.end(ctx.body)
      })
      .listen(...args)
  }
  createCtx(req, res) {
    const ctx = Object.create(this.context)
    ctx.request = Object.create(this.request)
    ctx.response = Object.create(this.response)
    ctx.req = ctx.request.req = req
    ctx.res = ctx.response.res = res
    return ctx
  }
}

const request = {
  get url() {
    return this.req.url
  }
}

const response = {
  get body() {
    return this._body
  },
  set body(val) {
    this._body = val
  }
}

const context = {
  get url() {
    return this.request.url
  },
  get body() {
    return this.response.body
  },
  set body(val) {
    this.response.body = val
  }
}
```

    测试它是否好用:

```typescript
// server.js

import MiniKoa from './application.js'

const port = 3000

const app = new MiniKoa()

function delay() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve()
    }, 2000)
  })
}

app
  .use(async (ctx, next) => {
    ctx.body = '(fn1) '
    await next()
    ctx.body += '(end fn1) '
  })
  .use(async (ctx, next) => {
    ctx.body += '(fn2) '
    await delay()
    await next()
    ctx.body += '(end fn2) '
  })
  .use(async (ctx, next) => {
    ctx.body += '(fn3) '
  })
  .listen(port, () => {
    console.log(`server is running at http://localhost:${port}`)
  })

// 浏览器输出：(fn1) (fn2) (fn3) (end fn2) (end fn1)
```

## 总结

    到此为止，一个简单的 Koa 就实现了，但是这里还缺少了异常处理，更详细的实现方式请查看 Koa 源码，无非也只是一些工具函数以及一些功能点的细化，其基本原理大概就是如此了。其中的难点是中间件原理，通过这个例子彻底理解中间件原理后，以后再使用起这个框架来，就更加得心应手了。