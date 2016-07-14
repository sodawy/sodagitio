---
title: koa笔记 co源码学习
date: 2016-07-11
type: code
tags:
- nodejs
- koa
- co
---

`此文假设你已经明白es6中generator用法, 可下方参考链接`

### 简介
- 作者[tj](https://github.com/tj), `The ultimate generator based flow-control goodness for nodejs (supports thunks, promises, etc)`
- `co`其实是`generator`的执行器. 而在`es7`中, 用`async/await`代替了`*fn/yield+co`

### 我简化后的co源码
``` javascript
"use strict";

function co(gen){
    let it = gen();

    return new Promise(function(resolve, reject){
        onfilfulled();

        function onfilfulled(res){
            let ret = it.next(res);
            next(ret);
        }

        function onreject(e){
            let ret = it.throw(e);
            next(ret);
        }

        function next(ret){
            if(ret.done){
                resolve(ret.value);
                return;
            }

            ret.value.then(onfilfulled, onreject)
        }
    });
}
```

### 源码解析
- `co`的参数是一个`generator`类型的`function`，返回值是`Promise`
- `co`内部会将传入的`generator`参数方法执行，并不断调用`next`方法，直到`.done`为`true`
- 每次`next()`结果中的`.value`(`yield` 右侧的表达式)会转化成`Promise`, 以下为源码中的转化过程
	``` javascript
		//co源码
		/**
		 * Convert a `yield`ed value into a promise.
		 *
		 * @param {Mixed} obj
		 * @return {Promise}
		 * @api private
		 */

		function toPromise(obj) {
		  if (!obj) return obj;
		  if (isPromise(obj)) return obj;
		  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj); //抹平了yield和yield*的差别
		  if ('function' == typeof obj) return thunkToPromise.call(this, obj);
		  if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
		  if (isObject(obj)) return objectToPromise.call(this, obj);
		  return obj;
		}
	```
- `toPromise`的结果(对应`简化源码25行`)，接上`.then(onfilfulled, onreject)`
- `ret.value`的`Promise`内的如果`resolve`, 进入`onfilfulled`, 传入结果`res`，成为接下来的`next(res)`调用时的参数, `res`于是成了`yield xxx`的返回值
	```javascript
		let haha = yield Promise.resolve('3')
		assert(haha, '3')
	```
- `promise`中如果抛出了异常或者`reject`, 进入`onreject`, 执行`it.throw`, 因此异常可在gen内的`try-catch语句破获到`
	```javascript
		try{
			yield Promise.reject(new Error('aError'))
		}catch(e){ assert(e.message, 'saError')}
	```




### 参考连接
- [coGithub源码](https://github.com/tj/co/blob/master/index.js)
- [es6入门](http://es6.ruanyifeng.com)
- [ecma-262](http://www.ecma-international.org/ecma-262/6.0/#sec-overview)


