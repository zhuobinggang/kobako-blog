---
title: requireJs实践
date: 2018-06-05 21:11:24
tags:
---

做软件工程课设闷死了,学点新东西自娱自乐

其实之前在公司闲着的时候已经看过一些文档了,但是果然没有实际自己搭建过是很难理解并且使用的.

<!--more-->


首先最简单的用法就是

```
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

这样一个引用会自动设置好requireJs的baseUrl

> baseUrl: 是requreJs寻找js文件的根目录

#### 官方推荐的目录结构以及用法

```
www/
 index.html
 js/
  app/
    sub.js
  lib/
    jquery.js
    canvas.js
 app.js
 require.js
```

可以看到lib里面是引入的外部库,app里面是自己写的js

`index.html`里面是`<script data-main="js/app.js" src="js/require.js"></script>`

然后是`app.js`

```js
requirejs.config({
    //By default load any module IDs from js/lib
    baseUrl: 'js/lib',
    //except, if the module ID starts with "app",
    //load it from the js/app directory. paths
    //config is relative to the baseUrl, and
    //never includes a ".js" extension since
    //the paths config could be for a directory.
    paths: {
        app: '../app'
    }
});

// Start the main app logic.
requirejs(['jquery', 'canvas', 'app/sub'],
function   ($,        canvas,   sub) {
    //jQuery, canvas and the app/sub module are all
    //loaded and can be used here now.
});
```

可以看到在paths里面写了app的path,这样在引用的时候,以app开头的引用就会自动引用这个前缀

#### 如何引用传统库

在`requirejs.config`里使用shim属性

主要原理是...把传统引用放到requireJs引用之前就好了

使用里面的exports属性定义引用的全局变量

```js
requirejs.config({
    //Remember: only use shim config for non-AMD scripts,
    //scripts that do not already call define(). The shim
    //config will not work correctly if used on AMD scripts,
    //in particular, the exports and init config will not
    //be triggered, and the deps config will be confusing
    //for those cases.
    shim: {
        'backbone': {
            //These script dependencies should be loaded before loading
            //backbone.js
            deps: ['underscore', 'jquery'],
            //Once loaded, use the global 'Backbone' as the
            //module value.
            exports: 'Backbone'
        },
        'underscore': {
            exports: '_'
        },
        'foo': {
            deps: ['bar'],
            exports: 'Foo',
            init: function (bar) {
                //Using a function allows you to call noConflict for
                //libraries that support it, and do other cleanup.
                //However, plugins for those libraries may still want
                //a global. "this" for the function will be the global
                //object. The dependencies will be passed in as
                //function arguments. If this function returns a value,
                //then that value is used as the module export value
                //instead of the object found via the 'exports' string.
                //Note: jQuery registers as an AMD module via define(),
                //so this will not work for jQuery. See notes section
                //below for an approach for jQuery.
                return this.Foo.noConflict();
            }
        }
    }
});

//Then, later in a separate file, call it 'MyModel.js', a module is
//defined, specifying 'backbone' as a dependency. RequireJS will use
//the shim config to properly load 'backbone' and give a local
//reference to this module. The global Backbone will still exist on
//the page too.
define(['backbone'], function (Backbone) {
  return Backbone.Model.extend({});
});
```