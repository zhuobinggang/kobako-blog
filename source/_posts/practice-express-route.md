---
title: practice-express-route
date: 2018-06-04 22:31:48
tags:
---

最近尝试学习REST API的时候,拜读了一篇博客,里面的代码是用express写的.其中一段路由分发的代码完全颠覆了我的认识,让我重新思考怎样才是最优雅的路由实现方式.

安哥说要善于总结与反思,尝试着把所思所得记录下来.这也是开始用hexo的理由.

第一步是问谷歌,看了这个博客[Organizing your app routes with the Express 4 Router](https://www.terlici.com/2014/09/29/express-router.html),一眼看过去是以前用过的操作,但是还是耐着性子看了下去,把一些新的感悟总结一下:

1. app.use的是路由
2. 路由里面可以use新的路由,理论上是可以无限嵌套的
3. 为了保持入口文件简单,只在里面引用一个引用了所有路由的路由
4. 可以在router里使用middleware
5. 中间件跟路由处理方法的主要区别是路由参数里多一个next,使用上没多大区别(这也是express坑的地方)

现在大部分express应用都是用以上写法,所以我们可以看到这样的目录结构

```
controllers/
  animals.js
  cars.js
  index.js
models/
public/
tests/
app.js
package.json
```

在`app.js`里面用一句话引用路由`app.use(require('./controllers'))`

其实这样写也挺好的,但是之前觉得很秀的写法像这样:

```js
app.route('/tasks')
  .get(todoList.list_all_tasks)
  .post(todoList.create_a_task);


app.route('/tasks/:taskId')
  .get(todoList.read_a_task)
  .put(todoList.update_a_task)
  .delete(todoList.delete_a_task);
```

我猜如果我来实现的话会像是这样

```js
router = require('express').Router()

router.get('/tasks',todoList.list_all_tasks)
router.post('/tasks',todoList.create_a_task)

router.get('/tasks/:taskId',todoList.read_a_task)
router.put('/tasks/:taskId',todoList.update_a_task)
router.delete('/tasks/:taskId',todoList.delete_a_task)

app.use('/',router)
```

果然很多冗余,不过能用route方法减少冗余也`只限`REST API这种操作了,因为重复的主要是url

#### Summary
总结一下吧,用路由还是得像以前那样用,不过在设计REST API的时候,别忘了可以用`route`方法秀一波操作