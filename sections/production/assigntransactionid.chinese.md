# 在每一个log语句指定‘TransactionId’

<br/><br/>


### 一段解释

一个典型的日志是来自所有组件和请求的条目的仓库。当检测到一些可疑行或错误时，为了与其他属于同一特定流程的行（如用户“约翰”试图购买某物）相匹配，就会变得难以应付。特别在微服务环境下，当一个请求/交易可能跨越多个计算机，这变得更加重要和具有挑战性。解决这个问题，可以通过指定一个唯一的事务标识符给从相同的请求过来的所有条目，这样当检测到一行，可以复制这个id，并搜索包含这个transaction id的每一行。但是，在node中实现这个不是那么直截了当的，这是由于它的单线程被用来服务所有的请求 – 考虑使用一个库，它可以在请求层对数据进行分组 – 在下一张幻灯片查看示例代码。当调用其它微服务，使用HTTP头“x-transaction-id”传递transaction id去保持相同的上下文。

<br/><br/>


### 代码示例: 典型的nginx配置

```javascript
//当接收到一个新的要求，开始一个新的隔离的上下文和设置一个事务transaction id。下面的例子是使用NPM库continuation-local-storage去隔离请求
 
var createNamespace = require('continuation-local-storage').createNamespace;
var session = createNamespace('my session');
    router.get('/:id', (req, res, next) => {
    session.set('transactionId', 'some unique GUID');
    someService.getById(req.params.id);
    logger.info('Starting now to get something by Id');
}
//Now any other service or components can have access to the contextual, per-request, data
class someService {
    getById(id) {
        logger.info(“Starting now to get something by Id”);
        //other logic comes here
    }
}
//Logger can now append transaction-id to each entry, so that entries from the same request will have the same value
class logger{
    info (message)
    {console.log(`${message} ${session.get('transactionId')}`);}
}
```

<br/><br/>

### What Other Bloggers Say
摘自博客 [ARG! TEAM](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load):
> ...Although express.js has built in static file handling through some connect middleware, you should never use it. *Nginx can do a much better job of handling static files and can prevent requests for non-dynamic content from clogging our node processes*...