---
title: 手写call、apply、bind
date: 2019-06-04 11:09:24
tags:  
    - javascript            
---
手写call、apply、bind

<!-- more -->

```javascript
/**
 * 手写call
 * @param context
 * @returns {*}
 * @constructor
 */
Function.prototype.call = function (context) {
    context = context||window;
    context.fn = this;
    let arr = [...arguments].slice(1);// 取出参数
    let result = context.fn(...arr); // 执行fn
    delete context.fn;
    return result;
}
/**
 * 手写apply
 * @param context
 * @returns {*}
 * @constructor
 */
Function.prototype.apply = function(context) {
    context = context||window;
    context.fn = this;
    let args = arguments[1];
    let result;
    if(args){
        result = context.fn(...args);
    }else {
        result = context.fn();
    }
    delete context.fn;
    return result
}
/**
 * 手写bind
 * @param context
 * @returns {F}
 */
Function.prototype.bind = function(context) {
    if(typeof this !== "function") throw new TypeError("Error");
    let me = this;
    let args = [...arguments].slice(1);
    return function F() {// 因为bind转换后的函数可以作为构造函数使用，此时this应该指向构造出的实例，而bind函数绑定的第一个参数。
        // 判断是否被当做构造函数使用
        if (this instanceof F) {
            return me.apply(this, args.concat([...arguments]));
        }
        return me.apply(context, args.concat([...arguments]));
    }
}
```