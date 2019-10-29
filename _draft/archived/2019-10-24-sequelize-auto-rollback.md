---
title: Koa 项目中 Sequelize 查询如何进行自动回滚
tags: [sequelize, mysql, koa, nodejs]
---
# Table of Contents

在 Koa 项目中使用 Sequelize, 如果不加任何处理，每个 controller 都必须用下面的代码来处理 Sequelize 查询的回滚。
```javascript
const transaction = await sequelize.transaction()

try {
  // ... query using transaction
} catch (error) {
  console.error(error)
  await transaction.rollback()
}

await transaction.commit()
```

明明有一个最后包住的错误处理，还是每次都写这种垃圾代码。

已经在 config.onerror 的配置里加入了错误堆栈的记录，现在还需要加入事务的自动回滚。问题就在于这个是个异步操作。sequelize 返回的是一个 Promise. 直接加入能够执行。但执行顺序上似乎不太理想。

这是第一次卡住的地方。怎么样在一个请求的末端再加入异步操作？觉得自己是被这个思路限制住了。回顾一下 koa 的调用链，这不就是一个 middleware 的事吗？

    // middleware after error handling
    module.exports = async function (ctx, next) {
      console.log('this is middleware working');
      await next();
      console.log('this is middleware logging after error handling');
    }

还是太年轻了，错误处理之后，请求就结束，返回响应了。

看了一下 koa 的代码，应该是这一行下面的 `onFinished` 的作用。

    // https://github.com/koajs/koa/blob/master/lib/application.js#L150
    class Application extends Emitter {
      // ...
      handleRequest(ctx, fnMiddleware) {
        const res = ctx.res;
        res.statusCode = 404;
        const onerror = err => ctx.onerror(err);
        const handleResponse = () => respond(ctx);
        onFinished(res, onerror);
        return fnMiddleware(ctx).then(handleResponse).catch(onerror);
      }
      // ...
    }

更多的需要更加深入的学习 koa 代码了。

回到原来的问题。

async 函数的传染性麻烦就麻烦在这里，你是将一个回调注入到第三方仓库的代码中。如果是自己的代码，再不济全链路 async 就是了，但这里是绝对不行的。

这里也不可能用回调的方法，因为回调的调用方不是你，你也没办法再定义一个。再改可能就要对 emitter 相关的东西做 hack 了。

只有退而取其次了。一开始的方案也不是不可以接受的。很多场景下把一些业务放到背景然后先响应甚至都是必要的，比如批量操作。只要注意做好日志就行了。

