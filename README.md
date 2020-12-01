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

## then()

`then`方法定义在原型上，它的作用是为`Promise`实例状态改变添加回调函数。第一个回调函数是调用成功的回调函数，第二个参数是调用失败的回调函数。

`then`方法返回一个`Promise`对象其后还可以添加`then`方法。

## catch()

是用来指定发生错误的回调函数。`Promise`对象的错误具有冒泡性质，会一直向后传递直到被捕获。类似于`(try/catch)`的写法，所以总是建议用`catch`方法而不用`then`的第二个函数。如果不使用`catch`方法，`Promise`内部的错误不会传递到外层。

`catch`方法返回的还是一个`Promise`对象，后面仍然可以跟`then`方法。

## finally()

`finally()`方法用于不管`Promise`对象最后状态如何，都会执行的操作。类似于`then`的特例。`finally`方法的回调函数不接受任何参数。

## all()

`Promise.all()`方法用于将多个`Promise`实例，包装成一个新的`Promise`实例。
```javascript
let promise = Promise.all([[p1,p2,p3]])
```
p1,p2,p3都是`Promise`的实例，如果不是的话，js内部会调用`Promise.resolve()`方法，将参数转为`Promise`的实例再进行处理。

`Promise.all`方法的参数可以不是数组，但是必须是具有`Iterator`接口，且返回的每一个成员都是`Promise`实例。

#### 两种状态
1.只有p1,p2,p3三个的状态都变成`fulfilled,peomise`的状态才会变成`fulfilled`,并且p1,p2,p3的返回值作为一个数组传递给`promise`的回调函数

2.只要p1,p2,p3中的任意一个状态变成`rejected`那么`promise`的状态就会变成`rejected`,此时第一个被`rejected`的实例的返回值作为参数传递给`promise`的回调函数

如果p1,p2,p3格子定义了自己的`catch`方法，如果其中一个被`rejected`他们就会走自己的`catch`方法，不会再走`Promise.all()`的`catch`方法。
```javascript
let p1 = new Promise((resolve,reject) => {
  return resolve('OK')
}).then(resolve => resolve).catch(e => e)

let p2 = new Promise((resolve,reject) => {
  return new Error('这是一个报错')
}).then(resolve => resolve).catch(e => e)

let promise = Promise.all([p1,p2]).then(resolve => console.log(resolve)).catch(e => console.log(e))

// ['OK',Error:'这是一个报错']
```
首先p1会`resolve`,p2首先会`reject`,走自己的`catch`,返回一个`Promise`的实例。当该实例执行完`catch`方法后，状态也会变成`resolve`，导致`Promise.all()`方法参数里面的两个实例都是`resolve`因此会调用`then`方法里面指定的回调函数，而不会走`Promise.all()`的`catch`。

## race方法
`Promise.race()`方法同样是将多个`Promise`实例包装成一个新的`Promise`实例。
```javascript
let promise = Promise.reace([p1,p2,p3])
```
接受的参数规则和`Promise.all()`一样。

p1,p2,p3中只要有一个状态发生改变，`promise`的状态就会发生改变。

## allSettled()

这个方法和`all`方法类似。

区别：

1.`all`方法需要等待所有的`promise`实例都是`fulfiled`，而`allSettled`不用管是不是`fulfiled`只要实例参数的状态都改变之后,`allSettled`就会变成`fulfiled`。

2.`allSettled`返回的参数也是一个数组，数组中的每个对象都有一个`status`参数，代表实例参数实际的状态。
```javascript
const p1 = Promise.resolve(42);
const p2 = Promise.reject(-1);

const allSettledPromise = Promise.allSettled([p1, p2]);

allSettledPromise.then(function (results) {
  console.log(results);
});
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```
## any()
`any`和`race`只有一点不同。

```javascript
let promise = Promise.any([p1,p2,p3])
```
`any`方法不会因为一个因为p1,p2,p3其中一个状态变成`rejected`而改变结束`promise`的状态。只有，p1,p2,p3三个状态都变成`rejected`他才会变成`rejected`

`race`是跟随最快改变状态的实例状态改变`promise`的状态。

## resolve()
该方法是将对象转化成`Promise`对象。

`Promise.resolve()`方法类似于下面这样：
```javascript
Promise.resolve('返回一个Promise对象')
==
let promise = new Promise(resolve => resolve('返回一个Promise对象'))
```

#### `Promise.resolve()`方法的参数分成四种情况：
1.参数是一个`Promise`实例

如果参数已经是一个`Promise`对象，那么该方法将原封不动的讲参数返回。

2.参数是一个`thenable`对象

`thenable`对象是指具有`then`方法的对象，
```javascript
let thenable ={
  then:function(resolve,rejected){
    resolve('这是一个thenable对象')
  }
}
```
`Promise.resolve()`方法会立即返回`thenable`对象，并执行`thenable`对象的`then`方法指定的回调函数。

3.参数不是具有`then`方法的对象，或者根本不是对象
```javascript
const p = Promise.resolve('Hello');

p.then(function (s) {
  console.log(s)
});
// Hello
```
4.不带有任何参数

`Promise.resolve()`方法允许调用时不带参数，直接返回一个`resolved`状态的 `Promise` 对象。

所以，如果希望得到一个 `Promise` 对象，比较方便的方法就是直接调用`Promise.resolve()`方法。
```javascript
const p = Promise.resolve();

p.then(function () {
  // ...
});
```
需要注意的是，立即`resolve()`的 `Promise` 对象，是在本轮“事件循环”`（event loop）`的结束时执行，而不是在下一轮“事件循环”的开始时。
```javascript
setTimeout(function () {
  console.log('3');
}, 0);
Promise.resolve().then(function () {
  console.log('2');
});
console.log('1');
// 1
// 2
// 3
```
**上面代码中，`setTimeout(fn, 0)`在下一轮“事件循环”开始时执行，`Promise.resolve()`在本轮“事件循环”结束时执行，`console.log('1')`则是立即执行，因此最先输出。**

## rejecte()
该方法也是返回一个`Promise`对象
```javascript
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```
`Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。