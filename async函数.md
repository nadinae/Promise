# async函数
`ssync`函数是`Generator`函数的语法糖。`async`函数对`Generator`函数的改进体现在以下四点。

###
+ 内置执行器

  `Generator` 函数的执行必须靠执行器，所以才有了co模块，而`async`函数自带执行器。也就是说，`async`函数的执行，与普通函数一模一样，只要一行。
  ```javascript
  asyncReadFile();
  ```
+ 更好的语义

  `async`和`await`，比起星号和`yield`，语义更清楚了。`async`表示函数里有异步操作，`await`表示紧跟在后面的表达式需要等待结果。

+ 更广的适用性

  co模块约定，`yield`命令后面只能是 `Thunk` 函数或 `Promise` 对象，而`async`函数的`await`命令后面，可以是 `Promise` 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 `resolved` 的 `Promise` 对象）。

+ 返回值是`Promise`

  `async`函数的返回值是 `Promise` 对象，这比 `Generator` 函数的返回值是 ``Iterator`` 对象方便多了。你可以用`then`方法指定下一步的操作。

**进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。**

## 语法

#### 返回`Promise`对象
`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。
```javascript
async function f(){
  return 'hello world'
}
f().then(v => console.log(v))
// hellow world
```
`async`函数内部抛出错误，会导致返回的`Promise`对象状态变成`reject`状态。抛出的对象会被`catch`方法回调函数接收到。
```javascript
async function f(){
  throw new Error('这是一个错误')
}
f().then(
  v => console.log('resolve',v),
  e => console.log('reject',e)
)
// reject Error:这是一个错误
```
