# Promise

#### Promise对象的两个特点
1.对象的状态不受外界影响

    1)pending(进行中)
    2)fulfilled(已成功)
    3)rejected(以失败)

    只有异步操作结果才能决定当前处于哪一种状态，任何其他的操作都无法改变这个状态。这正是Promise名字的由来。

2.`Promise`对象的状态一旦发生改变，就不会再变，任何时候都可以得到这个结果。

    1)pending---->fulfilled
    2)pending---->rejected
    只要这两种情况一发生状态就会凝固

#### 缺点
1.无法被取消，一旦新建就会立即执行，无法中途取消。

2.必须设置回调函数，否则内部抛出的错误无法反应到外部。

3.当处于pending状态的时候无法判断当前是进行到什么状态。

## 基本用法
创造`Promise`实例
```javascript
let promise = new Promise(function(resolve,rejected){
  if(/*成功*/){
    resolve(value)
  }else{
    reject(error)
  }
})
```
`Promise`构造函数接受一个函数作为参数，函数的两个参数是两个函数，由`javascript`内部引擎提供。Promise实例生成之后，可以调用`then`方法分别指定`resolved`和`rejected`状态的回调函数。
``` javascript
promise.then(function(value){
  // success
},function(error){
  // failure
})
```
`then`方法的第二个参数可选。

`promise`新建后会立即执行，看`promise`执行顺序的一个例子
```javascript
let promise = new Promise(function(resolve,rejscted){
  console.log('我第一个执行')
  resolve('我第三个执行')
})
promise.then(function(value){
  console.log(value)
})
console.log('我第二个执行')
```
`resolve`和`reject`并不会终结`promise`的执行，他们会放在`Promise`本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。
```javascript
let promise = new Promise(function(){
  resolve('我第二个执行')
  console.log('我第一个执行')
})
promise.then(function(value){
  console.log(value)
})
```
在`Promise`里面`resolve`和`reject`调用之后他的任务就已经完成了，后续的操作就应该在`then`方法里面执行。为了防止意外一般在`resolve`和`reject`前面添加`return`