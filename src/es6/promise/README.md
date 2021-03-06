# Promise conclution
《JavaScript高级程序设计》的作者，Nicholas C.Zakas 说过，要想全面理解和掌握JavaScript，关键在于弄清楚它的历史、本质和局限性。学习Promise也是, 在了解它为什么而生之后,对他的核心和使用场景也更加顺其自然了.所以第一步先简单了解下使用Promise的目的.

## 为什么使用Promise
一个开发场景: 有两个请求A和B顺序执行, B请求需要用到A请求的返回结果才能开始执行, 看下面这段代码示例:
```jsx
ajax({
    url: "/getId",
    type: "GET",
    success: function(res) {
        ajax({
            url: "/getList",
            type: "GET",
            success: function(res) {
                // some code
            }
        })
    }
})
```
异步请求B嵌套在异步请求A的成功回调函数中.如果顺序执行的请求变多,异步嵌套的层数也会越多,像上面的代码就会像右层层缩进,这样的代码会变得越来越臃肿,阅读上也非常不直观.
如果用Promise改编这段代码会变为:
```jsx
new Promise(function(resolve, rejected){
    ajax({
        url: "/getId",
        type: "GET",
        success: function(res) {
            // 通过resolve把结果传递出去
            resolve(res)
        } 
    })
}).then(function(res){
    ajax({
        url: "/getList",
        type: "GET",
        success: function(res) {
            // some code
        }
    })
})

```
对比两段代码, 在使用Promise后,第二个异步请求不用再嵌套在第一个异步请求的回调中.如果有第三个异步请求,在末尾添加一个then,然后把回调放在then里,可以简写表示为new Promise(A).then(B).then(C).catch()。Promise的这种then的链式调用的方式,解决了顺序执行的异步函数间层层嵌套的问题。

## 什么是Promise
Promise是一个拥有then方法的对象,通过then的链式调用来控制异步操作与回调函数的流程,让程序的流程清晰可辨.Promise/A+是规范, 所有Promise都必须严格按照该规范实现.
*   state: 状态机
    *  三种状态: PENDING, FULFILLED, REJECTED
    *  状态变化方向: Promise的状态只有两种变化方向,只能从PENDING状态分别变更为FULFILLED或者REJECTED状态
    *  触发状态变更的时机: 异步操作成功时,异步操作失败时
*   value: 一旦状态变更为fulfilled或者rejected时存储的终值或者拒因
*   handlers: 存储成功或失败下关联的回调函数
```
function fulfill(result) {
    // state状态由PENDING修改为FULFILLED
    state = FULFILLED;
    value = result;
}
function reject(error) {
    // state状态由PENDING修改为FULFILLED
    state = REJECTED;
    value = error;
}
```

## 怎么用
### 创建一个Promise
```
function runAsyn() {
    return new Promise(function(resolve, reject) {
        if (/* success */) {
            resolve(value)
        } else {
            reject(reason)
        }
    })
}
```

*   resolve(value): 
    *  异步请求成功时调用 
    *  更改了state——Promise的状态从PENDING变为FULFILLED 
    *  将异步操作的结果value传递出去
    
*   reject(reason): 
    *  异步请求失败时调用 
    *  更改了state——Promise的状态从PENDING变为REJECTED 
    *  将异步操作报错的原因reason传递出去
    
```
注意: 
1.   new Promise()会立即执行，所以最好放到一个函数中去调用 
2.   new Promise(function(resolve, reject) {
        // 会把当前区域里的同步代码先执行完, 异步总是最后再执行
    })
```

### Promise#then

```
const promise = new Promise(function(resolve, rejected){
})
promise.then(onFulfilled, onRejected)
```
*   then的两个参数是可选的, 如果类型不是function,会直接被忽略掉
*   Promise是FULFILLED状态的时候:
    *  onFulfilled是function: onFulfilled会被立即调用, onFulfilled的第一个参数接收的是resolve吐出来的value
    *  onFulfilled不是function: promise2在FULFILLED状态下取得的value和promise1的一样
*   Promise是REJECTED状态的时候:
    *  onRejected是function: onRejected会被立即调用, onRejected的第一个参数接收的是reject吐出来的reason
    *  onRejected不是function: promise2在REJECTED状态下取得的reason和promise1的一样

```
注意: 
1.   如果then里没有定义onRejected函数,那么promise会把error向下传递给catch
2.   then的第二个参数onRejected即function(error){}可以补货异步操作的错误,Promise#catch(fuction(error))方法也可以捕获异步操作返回的错误,这两个方法的区别在于catch的写法还能捕获到then方法中的可能发生的错误,所以最好是把错误捕获放到catch里.
     new Promise(function(resolve, reject){
     }).then(funcion(value){
        // some code
     }).catch(function(error){
        // 错误处理
     })
```

## 4. 传送带
[Promise/A+规范](https://promisesaplus.com)
[Promise 核心源码](https://github.com/then/promise/blob/master/src/core.js)

