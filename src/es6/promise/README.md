# Promise conclution
《JavaScript高级程序设计》的作者，Nicholas C.Zakas 说过，要想全面理解和掌握JavaScript，关键在于弄清楚它的历史、本质和局限性。学习Promise也是, 在了解它为什么而生之后,对他的核心和使用场景也更加顺其自然了.所以第一步先简单了解下使用Promise的目的.

## 1. 为什么使用Promise
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

## 2. 什么是Promise
Promise是一个拥有then方法的对象,通过then的链式调用来控制异步操作与回调函数的流程.
*   state: 状态机
    *  三种状态: PENDING, FULFILLED, REJECTED
    *  状态变化方向: Promise的状态只有两种变化方向,只能从PENDING状态分别变更为FULFILLED或者REJECTED状态
    *  触发状态变更的时机: 
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
*   value: 一旦状态变更为fulfilled或者rejected时存储的终值或者拒因
*   handlers: 存储成功或失败下关联的回调函数
