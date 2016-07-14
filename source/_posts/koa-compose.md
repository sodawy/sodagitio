---
title: koa笔记 koa中间件原理
date: 2016-07-13
type: code
tags:
- nodejs
- koa
- koa-compose
---

### 运行例子和图示
#### 一个请求从req进来到res回去, 经过中间件的顺序
![img](http://oa8gshrm5.bkt.clouddn.com/koa-mids.png)

#### 中间件代码执行的示意图
![img](http://oa8gshrm5.bkt.clouddn.com/koa-middleware.gif)

### koa中关于中间件的组织
1. 业务中调用`app.use(*fn)`, 对应源码中`this.middleware.push(fn);`
2. 有请求来时, 源码中执行`co.wrap(compose(this.middleware)).call(ctx)`, 就会如上图执行中间件
3. `co`之前[讲过了](https://sodawy.github.io/2016/07/11/koa-co/),是`generator`的执行器, `compose`是[koa-compose](https://github.com/koajs/compose)组件, 中间件执行原理就在里面

### koa-compose源码解析
1. `compose`算是中间件的执行器
2. 传入中间件数组, 就按上图执行
3. 源码:
    ```javascript
        /**
         * Compose `middleware` returning
         * a fully valid middleware comprised
         * of all those which are passed.
         *
         * @param {Array} middleware
         * @return {Function}
         * @api public
         */
        
        function compose(middleware){
          return function *(next){
            if (!next) next = noop();
        
            var i = middleware.length;
        
            while (i--) {
              next = middleware[i].call(this, next); //中间件数组从后向前分别执行一次, 返回的gen对象传给前一个中间件
            }
        
            return yield *next; //最后yield*第一个中间件, 从第一个中间件开始执行co的next循环
          }
        }
        
        /**
         * Noop.
         *
         * @api private
         */
        
        function *noop(){}
    ```
4. 调用例子
    ```javascript
        function *a(next){
            console.log('a1');
            yield next;
            console.log('a2');
        }
        
        function *b(next){
            console.log('b1');
            yield next;
            console.log('b2');
        }
        
        co(compose([a, b])); 
        
        //对应的执行顺序
        //genB = b(noop);
        //genA = a(genB);
        //yield* genA //进入a
        //console.log('a1');
        //遇到a中的yield next, 这个next是genB, 进入b
        //console.log('b1');
        //遇到b中的yiled next, 以为是最后一个中间件, 空函数直接执行
        //console.log('b2'), b执行完, 回到上一个中间件yield next位置继续执行
        //console.log('a2');
    ```
    
### 赞!叹!
这几行代码就做了这么nb的事!! 服!
    

