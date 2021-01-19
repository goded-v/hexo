---
title: 最简单的Vue数据双向绑定（响应式系统）的实现原理
date: 2019-06-04 11:09:24
tags:  
    - vue           
---
最简单的Vue数据双向绑定（响应式系统）的实现原理

<!-- more -->

```javascript
// observe方法遍历并包装对象属性
function observe(target) {
    // 若target是一个对象，则遍历它
    if(target && typeof target === "object") {
        Object.keys(target).forEach(key => {
            // defineReactive方法会给目标属性装上“监听器”
            defineReactive(target, key, target[key])
        })
    }
}
function defineReactive(target, key , val) {
    const dep = new Dep();
    // 属性值也可能是object类型，这种情况下需要调用observe进行递归遍历
    observe(val);
    Object.defineProperty(target, key, {
        // 可枚举
        enumerable: true,
        // 可配置
        configurable: true,
        get: function () {
            return val
        },
        set: function (value) {
            dep.notify()
        }
    })
}
class Dep {
    constructor() {
        this.subs = [];
    }
    addSub(sub) {
        this.subs.push(sub);
    }
    notify() {
        this.subs.forEach(sub => {
            sub.update();
        })
    }
}
```
