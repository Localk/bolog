>  传统的异步方式，需要使用回调函数，来处理后续的操作，在es6 及其以后的版本中，使用了很多解决回调产生的代码难维护的问题

# 一、promise

## 1、Promise 

promise 对象，是es6中的一种异步编程解决方案。promise 是一个对象，从其中可以获取异步操作的消息，可以说更像是一个容器，保存着未来才会结束的事件（也就是一个异步的操作）。promise 对象只有三种状态：进行中、结束、失败。那么它运行时，只能是 从进行中到失败，或者是从进行中到成功。使用promise对象，主要是通过同步的表达形式，来运行异步代码。

## 2、Promise 的使用

* 创建promise 对象
* 使用then 方法
* 使用 catch 方法
* 使用 finally 方法

一个简单的promise 对象，构造函数的参数，是一个待执行的函数，函数需要被传入两个回调函数，分别在当前函数体执行成功或失败后，有选择性的执行两个传入的回调函数

```javascript
let request = new Promise(function(success,fail){
    // code ，函数初始化时需要执行的操作，多数是一个异步操作，例如网络请求操作
    xhr.send()
    ...
    // 然后在异步操作有结果后，选择性的执行成功，或者失败的回调
    xhr.on请求成功事件 = function(){ // 伪代码
        success() ; // 执行成功的回调函数
    }
    xhr.on请求失败事件 = function(){ // 伪代码
        success() ; // 执行失败的回调函数
    }
}))
```

在 promise 对象实例化的时候，就会去执行需要执行的异步操作，如果需要在异步有结果后，再去执行对应的回调函数，还需要调用 promise 自身的 then 函数：

```javascript
request.then(function(){},function(){});
// then 中传入两个参数
// 如果异步结果是成功的，根据其内的逻辑，应该执行 then 中传入的第一个函数
// 如果异步结果是 失败的，根据期内的逻辑，应该执行 then 中传入的 第二个函数
```

当异步执行结果失败时，promise 对象调用 then 方法是，第一个参数传入的是null 或者 undefined，所以只会执行 then 函数中的第二个参数。这个过程可以使用 promise 对象的 catch 方法来代替：

```javascript
request.then(function(){
    // 异步成功成功的回调函数
}).catch(function(){
    // 异步失败的回调函数
})
```

在实际中，还可以再 then 或者 catch 函数中，再次返回一个 promise 对象，then 或者catch 会将回调函数生成的promise 对象，再次返回出来，所以可以使用调用链的方式，一直`then().then().then()`。

finally 方法，用来进行收尾的工作，也就是不管是 promise的状态是成功，还是失败，当执行完回调函数后，都会去寻找 finally 中寻找最后的回调函数来执行：

```javascript
request.finally(function(){
    // 最后 ， 且一定会执行的代码
})
```

## 3、Promise.all  和Promise.race

如果有一个同步任务，需要等待多个异步任务都执行完毕，才能执行，根据前面已知的方法来实现的话，依然会造成代码难以阅读和维护，所以，如果是需要等待多个异步任务的操作结果，使用`Promise.all` 方法

`Promise.all` 方法， 传入一个列表，该列表的元素，都是promise 对象。然后promise.all方法，会捕获执行列表中所有的执行结果，再包装成一个新的Promise 对象。

```javascript
let imgs = [1, 2, 3, 4, 5, 6];
let p_list = []; // 子任务列表，每个元素通过循环，变为一个 promise 对象
for (let key of imgs) {
    p_list.push(
        new Promise(function (suc, fail) {
            setTimeout(function () {
                console.log('异步操作:',key);
                suc(key);  // 子 promise 的返回值
            }, 3000)
        }))
}
// Promise.all 执行 子任务列表，返回一个新的promise 对象，所以 Promise.all 后面可以使用 then 方法
// 新的 promise 对象的返回值，是每个子Promise 返回值，组成的列表，即：r = 一个列表，
Promise.all(p_list)
    .then(r=>{
    console.log('all end : ',r);
})

// 最后打印出来，r 的值：[1, 2, 3, 4, 5, 6]
```

在Promise.all 中，返回值的顺序，是执行时，子任务列表的顺序，也就是说，等所有的子Promise 都执行完毕，如果没有失败，则返回子结果的列表，如果有失败，则在catch 中返回失败的那一个。

而相似的方法，Promise.race（），原理相似，但是返回的是执行最快的结果，其他的子任务，不进行等待。



# 二、generator

## 1、generator

generator 值得是一个生成器，也就是一个可迭代对象。其内封存了很多状态，这些状态不是一下就产生的，而是在需要生成的时候，再去调用生成器。所以，遍历生成器，可以获取到其内的每一个状态。定义一个生成器，使用function 后面加一个 * 来定义：

```javascript
// function + * 来定义一个生成器
// * 的位置，可以在function 和 函数名之间的任意位置
function* generators(){
    yield 1;
    yield 2;
}
```

## 2、yield 表达式 和 next 方法

yield 表达式，用来生成一个状态。对 一个生成器 产生的作用，类似于一个return，区别是 yield 不会结束函数的执行。yield 会在函数内部形成一个一个的断点，第一次执行函数时，就返回一个生成器。当生成器再次调用时（通过next 方法，这个方法是生成器的原型方法），是从函数顶部开始执行，执行到 yield 表达式时，返回yield 表达式的结果，并且在生成器内部打一个断点，标识当前执行结束的位置。再次调用生成器时（依然是通过 next 来调用），则从上次断点的位置开始继续向下执行，直到遇到一个 yield 表达式，或者 return，则将表达式的值返回，然后再次打一个断点。

```javascript
// a 是直接调用生成器函数，生成的生成器对象
let a = generators();

// 调用生成器对象， 获取第一次状态
// state_1 = {value:1,done:false}
// value 是返回的生成器值，done 代表生成器是否迭代完毕
let state_1 = a.next() ; 

// 可以一直调用 生成器 的 next 方法，直到生成器迭代完毕
let s = a.next();
...
let s = a.next();
// 如果生成器迭代完毕，再次调用next方法时，获取的返回值：{value: undefined, done: true}

```

yield 和 return 的区别 ： 用来返回生成器的一个状态，但不会终结生成器的执行。而return 会终止生成器函数。return之后，再次调用生成器的next 方法， 获取到的结果永远是迭代完成：`{value: undefined, done: true}` 。所以，yield 不能放在 return 之后。

next() 函数，用来执行生成器，从而获取生成器的状态。**在不传参数的情况下，next 函数调用时，将上一次 生成器的返回值，当做参数，传入函数体内部 **，如果是next 函数接收一个参数，用来在调用生成器时，将传入的参数，当做上一个生成器yield 的执行结果，替换原本产生的结果：

```javascript
function * gg (x){
    let y = yield x+1;
    yield y;
}
let g = gg(1);
g.next(); // g: y 未定义，因为未赋值，就返回了 , yield 的返回值 = 2
g.next(3); // g : y = 3,next 传入的参数，代替上一次yield 执行结果，继续操作，给y赋值，所以y = 3
```

# 三、async / await

async 是对generator的再一次语法糖封装，帮我们实现了生成器的调用，使语句更贴近同步代码的表达方式。

使用 async 标识的函数，会返回promise 对象，所以 该函数内部，可以添加任何的异步操作代码。可以将 async 函数，看做是多个异步操作，封装的 promise 对象，而await 表达式，就是then的语法糖。

```javascript
// promise 定义的异步操作
var p = new Promise(function(suc){
    setTimeout(function(){ // setTimeout 模仿一个异步操作
        suc(123)
    },3000)
})
// then 执行回调函数，也就是p实例化时，传入的suc 函数，then 回调函数的参数，作为 p 对象的返回值
p.then(function(num){console.log('end:',num)})  


// 整改成 async 方式
// 1. 先定义异步操作,异步操作有个返回值，作为回调函数的参数
function asyncFunction(){
    setTimeout(function(){
        return 123
    },3000)
}
async function main(){
    let num = await asyncFunction() ; // 返回 123
    nsole.log('end:',num); // 上文中，then 中的执行代码
}
```

如果想使用 await 来执行一个异步操作，那么其调用函数，必须使用 async 来声明。

await 能返回一个 promise 对象，也能返回一个值。如果await 返回的是 promise 对象，那么还可以继续给 await 的返回值使用 then 函数。











