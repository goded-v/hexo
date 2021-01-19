---
title: 最简单的Vue数据双向绑定（响应式系统）的实现原理
date: 2021-01-19 11:22:00
tags:  
    - vue           
---
最简单的Vue数据双向绑定（响应式系统）的实现原理

<!-- more -->


1、在生命周期的initState方法中将data，prop，method，computed，watch中的数据劫持， 通过observe方法与Object.defineProperty方法将相关对象转为换Observer对象。

2、然后在initRender方法中解析模板，通过Watcher对象，Dep对象与观察者模式将模板中的 指令与对象的数据建立依赖关系，使用全局对象Dep.target实现依赖收集。

3、当数据变化时，setter被调用，触发Object.defineProperty方法中的dep.notify方法， 遍历该数据依赖列表，执行其update方法通知Watcher进行视图更新。

vue是无法检测到对象属性的添加和删除，但是可以使用全局Vue.set方法（或vm.$set实例方法）。

vue无法检测利用索引设置数组，但是可以使用全局Vue.set方法（或vm.$set实例方法）。

无法检测直接修改数组长度，但是可以使用splice

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
