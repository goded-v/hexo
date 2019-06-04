---
title: 手写Promise
date: 2019-06-04 11:09:24
tags:  
    - javascript            
---
手写Promise的实现

<!-- more -->

```javascript

/**
 *
 * @param executor 执行器
 * @constructor
 */
export const Promise = function (executor) {
    let me = this;
    // 加入成功时的value 和 失败时的reason
    me.value = null;
    me.reason = null;

    // 储存then中成功回调的函数
    me.onResolvedCallbacks = [];
    // 储存then中失败的回调函数
    me.onRejectedCallbacks = [];

    // 加入状态标识
    me.status = "pending";

    // 成功
    function resolve(value) {
        // 判断是否处于pending状态
        if (me.status === "pending") {
            me.value = value;
            // 执行后改变状态
            me.status = "resolved";
            // 成功之后遍历then中成功的所有回调函数
            me.onResolvedCallbacks.forEach(fn => fn());
        }
    }

    // 失败
    function reject(reason) {
        // 判断是否处于pending状态
        if(me.status === "pending"){
            me.reason = reason;
            me.reason = reason;
            // 执行后改变状态
            me.status = "rejected";
            // 成功之后遍历then中失败的所有回调函数
            me.onRejectedCallbacks.forEach(fn => fn());
        }
    }

    // 对异常进行处理
    try{
        executor(resolve, reject);
    }catch (e) {
        reject(e);
    }
}
/**
 * 将then方法添加到构造函数的原型上
 * @param onFulfilled 成功回调
 * @param onRejected  失败回调
 */
Promise.prototype.then = function (onFulfilled,onRejected) {
    // onFulfilled如果不是函数，就忽略onFulfilled，直接返回value
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    // onRejected如果不是函数，就忽略onRejected，直接扔出错误
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };

    // 达成链式
    let promise2 = new Promise((resolve, reject) => {
        let me = this;
        if(this.status === "resolved"){
            // 异步
            setTimeout(() => {
                try {
                    let x = onFulfilled(me.value);
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            },0);
        }
        if(this.status === "rejected"){
            setTimeout(() => {
                try {
                    let x = onRejected(me.reason);
                    resolvePromise(promise2, x , resolve, reject);
                } catch (e) {
                    reject(e);
                }
            },0);
        }
        // 如果异步执行则位于pending状态
        if (this.status === "pending") {
            // 保存回调函数
            this.onResolvedCallbacks.push(() => {
                setTimeout(()=>{
                    try{
                        let x = onFulfilled(me.value);
                        resolvePromise(promise2, x, resolve, reject);
                    }catch (e) {
                        reject(e);
                    }
                },0)
            })
            this.onRejectedCallbacks.push(() => {
                setTimeout(()=>{
                    try{
                        let x = onRejected(me.value);
                        resolvePromise(promise2, x, resolve, reject);
                    }catch (e) {
                        reject(e);
                    }
                },0)
            })
        }
    })
    return promise2;
}
/**
 * 解析then返回值与新Promise对象
 * @param {Object} promise2 新的Promise对象
 * @param {*} x 上一个then的返回值
 * @param {Function} resolve promise2的resolve
 * @param {Function} reject promise2的reject
 */
function resolvePromise(promise2, x, resolve, reject) {
    if (promise2 === x) {
        reject(new TypeError('Promise发生了循环引用'));
    }
    // 防止多次调用
    let called;
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
        //可能是个对象或是函数
        try{
            let then = x.then;// 取出then方法应用
            if (typeof then === 'function') {
                //then是function，那么执行Promise
                // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
                then.call(x, (y) => {
                    if (called) return;
                    called = true;
                    //递归调用，传入y若是Promise对象，继续循环
                    resolvePromise(promise2, y, resolve, reject);
                }, (r) => {
                    // 成功和失败只能调用一个
                    if(called) return;
                    called = true;
                    reject(r);
                });
            } else {
                resolve(x); // 直接成功即可
            }
        }catch (e) {
            // 也属于失败
            if (called) return;
            called = true;
            // 取then出错了那就不要在继续执行了
            reject(e);
        }
    } else {
        //否则是个普通值
        resolve(x);
    }
}

/**
 * resolve方法
 * @param val
 */
Promise.prototype.resolve = function (val) {
    return new Promise((resolve,reject)=>{
        reject(val)
    });
}

/**
 * reject方法
 * @param val
 * @returns {Promise}
 */
Promise.prototype.reject = function(val){
    return new Promise((resolve,reject)=>{
        reject(val)
    });
}

/**
 * race方法
 * @param promises
 * @returns {Promise}
 */
Promise.prototype.race  = function (promises) {
    return new Promise((resolve,reject)=>{
        for(let i=0;i<promises.length;i++){
            promises[i].then(resolve,reject)
        };
    })
}

/**
 * all方法
 * @param promises
 * @returns {Promise}
 */
Promise.prototype.all = function(promises){
    let arr = [];
    return new Promise((resolve,reject)=>{
        let i = 0;
        function processData(index,data){
            arr[index] = data;
            i++;
            if(i == promises.length){
                resolve(arr);
            };
        };
        for(let i=0;i<promises.length;i++){
            promises[i].then(data=>{
                processData(i,data);
            },reject);
        };
    });
}
/**
 * cath方法 相当于调用 then 方法, 但只传入 Rejected 状态的回调函数
 * @param promises
 * @returns {Promise}
 */
Promise.prototype.catch = function(onRejected){
    return this.then(undefined,onRejected)
}
/**
 * finally方法 用于指定不管 Promise 对象最后状态如何，都会执行的操作
 * @param onRejected
 * @returns {Promise}
 */
Promise.prototype.finally = function(val){
    return this.then(
        value  => Promise.resolve(val).then(() => value),
        reason => Promise.resolve(val).then(() => { throw reason })
    );

}

```
