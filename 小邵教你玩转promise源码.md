前言：大家好，我叫邵威儒，大家都喜欢喊我小邵，学的金融专业却凭借兴趣爱好入了程序猿的坑，从大学买的第一本vb和自学vb，我就与编程结下不解之缘，随后自学易语言写游戏辅助、交易软件，至今进入了前端领域，看到不少朋友都写文章分享，自己也弄一个玩玩，以下文章纯属个人理解，便于记录学习，肯定有理解错误或理解不到位的地方，意在站在前辈的肩膀，分享个人对技术的通俗理解，共同成长！

> 后续我会陆陆续续更新javascript方面，尽量把javascript这个学习路径体系都写一下  
> 包括前端所常用的es6、angular、react、vue、nodejs、koa、express、公众号等等  
> 都会从浅到深，从入门开始逐步写，希望能让大家有所收获，也希望大家关注我~

> 源码地址：https://github.com/iamswr/promiseAplus  
文章列表：https://juejin.im/user/5a84f871f265da4e82634f2d/posts

> Author: 邵威儒  
> Email: 166661688@qq.com  
> Wechat: 166661688  
> github: https://github.com/iamswr/
***
JavaScript作为单线程语言，其特点也是其缺陷，特点就是不用处理多线程引发的占用资源、冲突啪啦啪啦等，缺陷就是同一时间，只能做一件事情，那么会存在一个问题，网络传输是有延迟的，比如A发一条信息到B服务器，在B服务器还没返回信息给A时，那么A就会一直在等待接收信息，会造成页面的假死，那么该怎么办？俗话说得好，程序猿改变世界，于是乎出现了异步的概念，我会分以下几点，去讲述我对前端异步的理解:

1. callback
2. 闭包、高阶函数
3. promise
4. generator （ 文章地址：https://juejin.im/post/5b7512ec518825614064f36e ）
5. async / await  （ 文章地址：https://juejin.im/post/5b7512ec518825614064f36e ）

***
### 一.callback
回调函数被认为是一种高级函数，一种被作为参数传递给另一个函数(在这称作"otherFunction")的高级函数，回调函数会在otherFunction内被调用(或执行)。回调函数的本质是一种模式(一种解决常见问题的模式)，因此回调函数也被称为回调模式。

是不是看起来一头懵逼，到底什么是回调函数？有什么作用？我对回调函数理解，从真正意义上让我突然间恍然大悟的，就是当初研究jQuery底层源码的时候，我们看以下一段代码：
```
<img src='../a.jpg'></img>
<img src='../b.jpg'></img>
<img src='../c.jpg'></img>

$("img").attr("title",function(index,attr){
    console.log(index) // 依次返回0 1 2
});
```

在attr方法的第二个参数，传入了一个function，而该函数，会依次获取`$('img')`的DOM对象对应index和attr，我们可以在该function里，写我们需要的业务逻辑，那么这样有什么好处呢？

我的理解是，假如我要封装一个库，造一个轮子，那么要考虑到通用性和复用性并且提供一个途径，让使用者任意发挥想象写业务逻辑，并且把相关可能使用到的参数，都传给使用者。

假设，我们现在有一个需求，要写一个判断类型的方法，常见的方法有以下几种：

```
 - typeOf // 简单的数据类型判断，栈区
 - instanceof // 复杂的数据类型，堆区
 - constructor // 复杂的数据类型，主要是用在继承的改写指向的构造函数，很少用于判断类型
 - Object.prototype.toString.call() // 绝大多数库底层都是使用该方式，返回值如[object String]
```

首先，我们写一个isType方法
```
function isType(content,type,fn){
    // 类型判断
    let t = Object.prototype.toString
                              .call(content)
                              .replace(/\[object\s|\]/g,'')
                              
    // 判断完成后，执行传入的callback函数
    fn(type,t)
}
```

现在我们要判断一个值的类型，然后拿到这个类型，最终执行我们需要做的事情
```
isType('hello swr','String',function(type,t){ // 作为参数传入的函数，接收isType函数内的fn中type和t这两个参数
    console.log(type === t) // true
})
```

![](https://user-gold-cdn.xitu.io/2018/7/25/164d019bffb02e2c?w=1788&h=552&f=png&s=140356)
首先执行了isType函数，然后执行isType函数内部的代码，isType内的fn(type,t)，这里的fn方法，实际就是我们传入的第三个参数，即里面只有一行console.log(type === t)的函数，而t则是isType函数内的t，可以稍微把整个流程体会一下，就差不多明白回调函数是干嘛的了~

那么问题就出现了，比如我们使用node.js的时候，进行文件读取操作时，想获取的值是一种嵌套依赖关系时，会出现什么问题呢？
```
目录结构：
- iamswr
  - A.txt
  - B.txt
  - C.txt
  
  其中
  A.txt文件里的内容为字符串B.txt
  B.txt文件里的内容为字符串C.txt
  C.txt文件里的内容为字符串'hello swr'
  
  那么当我们想获取到'hello swr'，会遇到什么问题呢？请看下面的代码
  let fs = require('fs')
  fs.readFile('A.txt','utf8',function(err,data){ // 此时回调函数data值为'B.txt'
      fs.readFile(data,'utf8',function(err,data){ // 此时回调函数data值为'C.txt'
          fs.readFile(data,'utf8',function(err,data){
            console.log(data) // 'hello swr'
          })
      })
  })
```
以上这个例子如果嵌套依赖层次更高一些，那代码变得十分难维护以及难阅读，我们在企业开发当中，经常会遇到想得到的数据，是通过嵌套依赖的关系，最终才获得需要的数据，陷入了回调地狱，而es6中，promise解决了这个让前端头疼的问题，后面我会详细讲promise，下面我们先了解一下闭包、高阶函数。


***
### 二.闭包、高阶函数
我个人理解，闭包实际上是一种函数，所以闭包技术也是函数技术的一种；闭包能做的事情函数几乎都能做，闭包有最大的两个用处，一个是可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。

#### 1.封闭作用域
在javascript中，如果一个对象不被引用了，那么这个对象会被GC回收，否则则一直保留在内存中，那么利用这个特点，配合闭包使用，有以下几个优点：封闭作用域、保存作用域、作用域链条。

不污染全局变量，当团队协作时，比如A大佬，封装了jQuery库，而jQuery库内是有大量变量，如果不使用闭包，则jQuery库内的变量会污染整个项目，甚至和其他团员的变量有冲突

```
外部无法获取闭包内的变量，封闭了作用域
(function(){
    var str = 'hello swr'
    console.log(str) // 'hello swr'
})()

console.log(str) // 报错

我们用原生js来写代码的时候，会存在一个问题，
比如有5个button标签
var btns = document.getElementsByTagName('button');
for(var i=0; i< btns.length; i++){   
   var btn = btns[i];        
   btn.onclick = function () {            
     alert('点击了第' + i + '个按钮');       
   }
}
无论我们点击哪个button，都是弹出'点击了第5个按钮'，
因为btn.onclick事件是异步触发的，当事件被触发时，
for循环早已经结束，此时变量I的值已经是5，
所有onclick事件函数从内到外查找变量i时，查找到的值总是5。

可以通过封闭作用域把每次循环的i值都封闭起来,
当时间函数顺着作用域链从内到外查找变量i时,
会先找到被封闭在闭包环境中的i,
如果有5个按钮, 则i的值就是0,1,2,3,4
var btns = document.getElementsByTagName('button');
for(var i=0; i< btns.length; i++){   
    (function (i) {        
       var btn = btns[i];        
       btn.onclick = function () {            
         alert('点击了第' + i + '个按钮');       
       }    
     })(i);
}

```


#### 2.作用域链
我们知道，在es6之前，只有函数是有作用域的说法，在es6出现了，则有了块级作用域的说法，比如
```
(function person(){
    var name = '邵威儒'
    console.log(name) // '邵威儒'
})()

console.log(name) // 报错
```
在函数外部，是访问不了内部的name，这就是作用域。
在es6出了一个新的概念，就是块级作用域
```
{
    let name = '邵威儒'
    console.log(name) // '邵威儒'
}

console.log(name) // 报错
```
效果和闭包一样

#### 3.保存作用域

函数嵌套函数，那么内部的那个函数将形成作用域闭包。简单的说，这种闭包能够达到的好处就是让指令能够绑定一些全局数据去运行，优点是全局数据隐藏化、 将数据绑定在指令上运行，让指令不再依赖全局数据。

```
function plus(num){
    ++num
    return function(){
        console.log(num)
    }
}

let toPlus = plus(5)
此时toPlus实际上为
function(){
    console.log(num)
}
而这个num实际上就是plus函数内作用域的num，此时我们无法从外部修改num，而且把plus函数内的数据隐藏化，将数据绑定在toPlus上运行。
```

#### 实际开发中遇到的问题

比如说，我们实际开发中会遇到一个问题，就是某个函数，要等多个异步执行完毕后才执行，这种情况怎么做呢？

一般会想到以下这个办法

```
let fs = require('fs')
let arr = []

fs.readFile('./a.txt','utf8',function(err,data){
    arr.push(data) // 假设data为'hello'
})

fs.readFile('./b.txt','utf8',function(err,data){
    arr.push(data) // 假设data为'swr'
})

console.log(arr) // 我们希望打印出来是['hello','swr']或['swr','hello']，但是打印出来的却是[]
这是为什么呢？
是因为javascript执行原理，是先执行同步，再执行异步的，而fs.readFile方法属于异步方法，所以还没执行完毕，就已经执行了console.log(arr)了
```
对于这种并非依赖嵌套获取，我们称为“同步”获取，此同步非异步同步的那个同步，特别是这种异步请求的数据，获取到的时间先后顺序不同，那我们该如何实现“同步”获取呢？
```
let fs = require('fs')

function after(times,callback){
    let arr = []
    return function(data){
        arr.push(data)
        if(--times === 0){
            callback(arr)
        }
    }
}

let fn = after(2,function(arr){
    console.log(arr) // 当fn执行两次后，则会执行该回调函数
})

fs.readFile('./a.txt','utf8',function(err,data){
    fn(data) // 假设data为'hello'
})

fs.readFile('./b.txt','uft8',function(err,data)=>{
    fn(data) // 假设data为'swr'
})

最终当2个fs.readFile读取完毕后，执行了fn()达到2次时，则会打印出['hello','swr']或者['swr','hello']
```

虽然以上的方式，实现了我们需要的需求，但是问题来了，难道我们每一次都要特意写一个after函数吗？其实还有一个概念，叫做发布订阅，订阅就类似你收藏了这个电台，而发布，则是这个电台向所有收藏了本电台的粉丝进行广播，看下面代码
```
let fs = require('fs')

let event = {
    arr:[], // 存需要执行的函数
    result:[], // 存结果
    on(fn){ // 订阅
        this.arr.push(fn)
    },
    emit(data){ // 发布
        this.result.push(data)
        this.arr.forEach(fn=>fn(this.result))
    }
}

event.on(function(data){
    if(data.length === 2){
        console.log(data) // ['hello','swr'] 或者 ['swr','hello']
    }
})

fs.readFile('./a.txt','utf8',(err,data)=>{
    event.emit(data) // data为'hello'
})

fs.readFile('./b.txt','utf8',(err,data)=>{
    event.emit(data) // data为'swr'
})

当两个fs.readFile读取完成，并且在其回调函数内执行了event.emit，最终会打印出['hello','swr'] 或者 ['swr','hello']
```
***
### 三.Promise
啰啰嗦嗦说了那么多，主要是想大家了解一下回调函数以及闭包，因为这概念和promise的紧密关联的，promise部分我主要是想和大家根据promiseAplus规范，逐步手写一个promise的底层实现方式。

首先，promise怎么理解？我在知乎上看到一篇比较通俗易懂的小故事，大家可以看看，https://zhuanlan.zhihu.com/p/19622332

> 早上，老爸说：“儿子，天气如何？”
每周一早上，老爸问儿子下午的天气情况，儿子可以到自家房子旁边小山上使用望远镜来观看。儿子在出发时许诺（Promise）老爸（会通知老爸天气情况）。 
此刻，老爸决定，如果天气不错，明天就出去捕鱼，否则就不去。而且如果儿子无法获得天气预报的话，也不去捕鱼。 
30分钟左右，儿子回来了，每周的结局都不一样。 

> 结局A：成功获得了（retrieved）天气预报，晴天 :)
儿子成功获取了天气预报，天空晴朗，阳光明媚！承诺（Promise）兑现了（resolved），于是老爸决定开始为周日的捕鱼做准备。

> 结局B：同样成功获得了天气预报，雨天:(
儿子成功获得了天气预报，只不过是乌云密布，要下雨。承诺（Promise）兑现了（resolved），只是老爸决定呆在家里，因为天气很糟糕。

> 结局C：没法获得天气预报:-/
出了问题，儿子没法得知天气预报，因为雾很大，就算站在小山上也无法看清。儿子没办法对象他离开时许下的诺言， promise was rejected！老爸决定留下来，这并不值得冒险。

#### Promise的一些特性

首先我们要了解PromiseA+规范 https://promisesaplus.com/

* promise是有兼容性问题的，node环境下默认支持，还可以下载相应插件来解决兼容性问题
* promise是有三种状态的，等待态pending / 成功态resolved / 失败态rejected
* promise的状态是可以转换的，可以从pending -> resolved 或 pending -> rejected，但是resolved不能转换为rejected/pending，rejected不能转换为resolved/pending，简而言之即状态只会更改一次

```
// Promise构造函数的第一个参数为executor
let promise = new Promise(function(resolve,reject){
    console.log('我是会被立即执行的哟')
})

// promise的实例都有then方法
promise.then(()=>{ // 成功的回调
    
},()=>{ // 失败的回调
    
})
```
* executor默认在new的时候会自动执行
* 每个promise的实例都有then方法
* then方法中，有两个参数，分别是成功的回调函数和失败的回调函数
```
// 默认时为pending态，既不会走成功的回调也不会走失败的回调
promise.then(()=>{
    console.log('success1')
},()=>{
    console.log('error1')
})

console.log('2')

在这段代码中，只会打印出'2'，因为promise一直处于pending态，不会走then后的回调函数
```
```
let promise = new Promise(function(resolve,reject){
    console.log('1')
    resolve() // 更改pending状态为resolved
})

promise.then(()=>{
    console.log('success1')
},()=>{
    console.log('error1')
})

console.log('2')

此时输出顺序为'1' -> '2' -> 'success1'
```
* then方法是异步的，属于微任务，从上面的例子可以看出，先执行完同步代码，再执行异步代码
```
let promise = new Promise(function(resolve,reject){
    console.log('1')
    setTimeout(()=>{ // 异步行为
        resolve() // 更改状态为成功
    },1000)
})

promise.then(()=>{
    console.log("success1")
})

promise.then(()=>{
    console.log('success2')
})

console.log("2")

此时输出顺序为'1' -> '2' -> 'success1' -> 'success2'
```
* 同一个promise的实例可以then多次,成功时会调用所有的成功方法，失败时会调用所有的失败方法
* new Promise中可以支持异步行为
```
let promise = new Promise(function(resolve,reject){
    throw new Error('出错了') // 抛出错误
})

promise.then(()=>{
    console.log('success1')
},()=>{
    console.log('error1')
})

此时输出为 'error1'
```
* 如果发现错误，就会进入失败态

#### 实现一个Promise

下面代码部分和源码实现部分要结合来看
```
// ----- 代码部分
// 1.executor默认在new的时候会自动执行
// 成功和失败的视乎可以传递参数
let promise = new Promise((resolve,reject)=>{ // 6.resolve、reject函数对应源码实现部分的resolve、reject函数
    resolve('hello swr') // 11.执行resolve
})

// 7.Promise的实例都有then方法
promise.then((data)=>{ // 8.成功的回调函数
    
},(err)=>{ // 9.失败的回调函数
    
})
```
```
// ----- 源码实现部分
// 2.声明一个Promise构造函数
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined // 12.因为value和reason值需要在Promise实例方法then中使用，所以把这两个值，赋给new出来的实例
    function resolve(value){ // 3.声明一个resolve函数
        self.value = value // 13.当调用了resolve并且传参数时，则把这value值赋予self.value
    }
    
    function reject(reason){ // 4.声明一个reject函数
        self.reason = reason // 13.当调用了reject并且传参数时，则把这reason值赋予self.reason
    }
    executor(resolve,reject) // 5.把resolve、reject函数传到executor
}

// 因为Promise的实例都有then方法，那么意味着then方法是在Promise的原型对象中的方法
// 10.对应上面成功的回调函数onFulfilled以及失败的回调函数onRejected
Promise.prototype.then = function(onFulfilled,onRejected){
    
}

module.exports = Promise // 把Promise暴露出去
```
此时，我们会发现，如何去判断调用resolve还是reject呢？
这个时候我们在内部应该维护一个状态，而我们之前说过了Promise有三种状态，分别为pending、resolved、rejected，那么我们接着看下面的代码。
```
// ----- 代码部分
let promise = new Promise((resolve,reject)=>{
    resolve('hello swr') // 5.暂时忽略此行
    resolve('看看同时执行resolve和reject会发生什么？')  // 5.此行执行resovle
    reject('看看同时执行resolve和reject会发生什么？') // 5.此行执行reject
})

promise.then((data)=>{
    console.log('success:' + data) // 5.当调用了resolve函数，则输出success:hello swr
},(err)=>{
    
})
```
```
// ----- 源码实现部分
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined 
    self.status = 'pending' // 1.在内部维护一个status状态
    function resolve(value){
        self.value = value 
        self.status = 'resolved' // 2.当调用了resolve时，更改状态为resolved
    }
    
    function reject(reason){
        self.reason = reason 
        self.status = 'rejected' // 2.当调用了reject时，更改状态为rejected
    }
    executor(resolve,reject)
}

Promise.prototype.then = function(onFulfilled,onRejected){
    let self = this
    // 3.当我们在then中，执行了成功或者失败的回调函数时，首先要判断目前处于什么状态
    if(self.status === 'resolved'){
        onFulfilled(self.value) // 4.当调用了resolve函数后，会执行成功的回调函数，并且把resolve中传递的值，传递给成功的回调函数
    }
    
    if(self.status === 'rejected'){
        onRejected(self.reason) // 4.当调用了reject函数后，会执行成功的回调函数，并且把reject中传递的值，传递给失败的回调函数
    }
}

module.exports = Promise
```
当我们在上面5中同时执行resolve和reject，会发现都能够执行，那么就违背了状态只能更改一次的原则了，下面我们来解决这个问题。
```
// ----- 代码部分
let promise = new Promise((resolve,reject)=>{
    resolve('看看同时执行resolve和reject会发生什么？') // 1. 此时执行resolve和reject
    reject('看看同时执行resolve和reject会发生什么？') // 3.此时即使调用reject，因为resolve已经调用了一次，从pending更改为resolve，所以在第一次调用后，多次调用也不会生效
    
    // 4.以上resolve、reject暂时忽略掉，我们考虑一个情况，当promise抛出错误时，怎么去处理呢？
    throw new Error('出错啦')
})

promise.then((data)=>{
    console.log('success:' + data)
},(err)=>{
    
})
```
```
// ----- 源码实现部分
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined 
    self.status = 'pending'
    function resolve(value){
        if(self.status === 'pending'){ // 2.此时新增一个状态判断，当状态为pending的时候才能执行
            self.value = value 
            self.status = 'resolved'
        }
    }
    
    function reject(reason){
        if(self.status === 'pending'){ // 2.此时新增一个状态判断，当状态为pending的时候才能执行
            self.reason = reason 
            self.status = 'rejected'
        }
    }
    
    // 5.当我们在执行executor时，内部抛出错误的时候，可以利用try catch来处理这个问题
    try{
        executor(resolve,reject)
    }catch(error){
        reject(error)
    }
}

Promise.prototype.then = function(onFulfilled,onRejected){
    let self = this
    if(self.status === 'resolved'){
        onFulfilled(self.value) 
    }
    
    if(self.status === 'rejected'){
        onRejected(self.reason) 
    }
}

module.exports = Promise
```
这样我们就解决了多次调用，只认第一次的更改状态，并且当抛出错误时，使用try catch来处理，那么接下来，我们想一下，目前我们都是new一个Promise，然后调用then，这整个流程，仿佛没任何问题，但是，现在问题出现了，如果此时resolve或者reject是处于setTimeout(()=>{resolve()},3000)中，即处于异步中，当我们new一个Promise时，不会马上执行异步代码，而是直接执行了promise.then这个函数，而此时因为self.status的状态依然是处于pending，所以不会执行resolve或者reject，当同步代码执行完毕后，执行异步代码时，更改了状态为resolved或者rejected时，此时then方法已经执行完毕了，不会再次执行then的方法，那么此时我们该如何处理？

还存在一个问题，就是上面所说的，同一个promise的实例可以then多次,成功时会调用所有的成功方法，失败时会调用所有的失败方法，那这个又该如何处理呢？

可以利用我们前面所说的发布订阅的思路来解决，现在我们看下面代码。
```
// ----- 代码部分
let promise = new Promise((resolve,reject)=>{
    setTimeout(()=>{ // 1.此时resolve处于异步
        resolve('hello swr')
    },3000)
})

promise.then((data)=>{ // 多个then
    console.log('success1:' + data)
},(err)=>{
    
})
promise.then((data)=>{ // 多个then
    console.log('success2:' + data)
},(err)=>{
    
})
```
```
// ----- 源码实现部分
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined 
    self.status = 'pending'
    self.onResolvedCallbacks = [] // 2.可能new Promise中会有异步的操作，此时我们把异步操作时，执行的then函数的成功回调，统一保存在该数组中
    self.onRejectedCallbacks = [] // 2.可能new Promise中会有异步的操作，此时我们把异步操作时，执行的then函数的失败回调，统一保存在该数组中
    function resolve(value){
        if(self.status === 'pending'){ 
            self.value = value 
            self.status = 'resolved'
            // 4.当调用resolve时，把该数组中存放的成功回调都执行一遍，如果是异步，则会把成功的回调都存到该数组里了，如果是异步，则没存到。
            self.onResolvedCallbacks.forEach(fn=>fn())
        }
    }
    
    function reject(reason){
        if(self.status === 'pending'){ 
            self.reason = reason 
            self.status = 'rejected'
            // 4.当调用reject时，把该数组中存放的失败回调都执行一遍，如果是异步，则会把成功的回调都存到该数组里了，如果是异步，则没存到。
            self.onRejectedCallbacks.forEach(fn=>fn())
        }
    }
    
    try{
        executor(resolve,reject)
    }catch(error){
        reject(error)
    }
}

Promise.prototype.then = function(onFulfilled,onRejected){
    let self = this
    if(self.status === 'resolved'){
        onFulfilled(self.value) 
    }
    
    if(self.status === 'rejected'){
        onRejected(self.reason) 
    }
    
    // 3.当new Promise中有resolve、reject处于异步中，执行then的时候，状态为pending，
    if(self.status === 'pending'){
        self.onResolvedCallbacks.push(()=>{
            onFulfilled(self.value)
        }) // 3. 把成功的回调函数，存到该数组中，这样写的好处，就是把参数传进去，不需要将来遍历onResolvedCallbacks时，再传参
        self.onRejectedCallbacks.push(()=>{
            onRejected(self.reason)
        }) // 3. 把失败的回调函数，存到该数组中，这样写的好处，就是把参数传进去，不需要将来遍历onRejectedCallbacks时，再传参
    }
}

module.exports = Promise
```
到此为止，我们简版的Promise实现得差不多了，小伙伴们可以对着代码敲一下，感受一下，体会一下。
***
##### Promise的链式调用
其实Promise的核心在于链式调用，Promise主要是解决2个问题：
* 回调地狱
* 并发异步io操作，同一时间内把这个结果拿到，即比如有两个异步io操作，当这2个获取完毕后，才执行相应的代码，比如前面所说的after函数，发布订阅、Promise.all等。

首先，比如回调地狱怎么解决呢？那么我们来看下面的代码，并且改为promise。
```
// 回调函数
let fs = require('fs')
fs.readFile('./a.txt','utf8',(err,data)=>{ // 往fs.readFile方法传递了第三个为函数的参数
    if(err){
        console.log(err)
        return
    }
    console.log(data)
})

// 改写为Promise
let fs = require('fs')
function read(filePath,encoding){
    return new Promise((resolve,reject)=>{
        fs.readFile(filePath,encoding,(err,data)=>{
            if(err) reject(err)
            resolve()
        })
    })
}

read('./a.txt','utf8').then((data)=>{ // 在这里则不再需要传回调函数进去，而是采用then来达到链式调用
    console.log(data)
},(err)=>{
    console.log(err)
})

// 这样看好像Promise也没什么优势，那么接下来我们对比一下
// 假设有3个文件
// - 1.txt    文本内容为'2.txt'
// - 2.txt    文本内容为'3.txt'
// - 3.txt    文本内容为'hello swr'

// 用回调函数
fs.readFile('./1.txt','utf8',(err,data)=>{
    fs.readFile(data,'utf8',(err,data)=>{
        fs.readFile(data,'utf8',(err,data)=>{
            console.log(data) // hello swr
        })
    })
})

// 用Promise
read('./1.txt','utf8')
.then((data)=>{
    // 1.如果一个promise执行完后，返回的还是一个promise，
    //   会把这个promise的执行结果会传递给下一次then中
    return read(data,'utf8')
})
.then((data)=>{
    return read(data,'utf8')
})
.then((data)=>{
    // 2.如果在then中返回的不是一个promise，
    //   而是一个普通值，会将这个普通值作为下次then的成功的结果
    return data.split('').reverse().join('')
})
.then((data)=>{
    console.log(data) // rws olleh
    // 3.如果当前then中失败了，会走下一个then的失败回调
    throw new Error('出错')
})
.then(null,(err)=>{
    console.log(err) // Error:出错   报错了
    // 4.如果在then中不返回值，虽然没有显式返回，
    //   但是默认是返回undefined，是属于普通值，依然会把这个普通值传到
    //   下一个then的成功回调中
})
.then((data)=>{
    console.log(data) // undefined
})
```
从上面可以看得出，改写为Promise的代码，更好阅读和维护，从用Promise方式可以得出结论:
* 1.如果一个promise执行完后，返回的还是一个promise，会把这个promise的执行结果会传递给下一次then中
* 2.如果在then中返回的不是一个promise，而是一个普通值，会将这个普通值作为下次then的成功的结果
* 3.如果当前then中失败了，会走下一个then的失败回调
* 4.如果在then中不返回值，虽然没有显式返回，但是默认是返回undefined，是属于普通值，依然会把这个普通值传到下一个then的成功回调中
```
// 如果在then中抛出错误，会怎样呢？
// 情景一，会被下一个then中的失败回调捕获
read('./1.txt','utf8')
.then((data)=>{
    throw new Error('出错了')
})
.then(null,(err)=>{
    console.log(err) // Error:出错了   报错
})

// 情景二，如果没有被失败的回调捕获，抛出错误最终会变成异常
read('./1.txt','utf8')
.then((data)=>{
    throw new Error('出错了')
})

// 情景三，如果没有被失败的回调捕获，那么最终会被catch捕获到
read('./1.txt','utf8')
.then((data)=>{
    throw new Error('出错了')
})
.then((data)=>{
    
})
.catch((err)=>{
    console.log(err) // Error:出错了   报错
})

// 情景四，如果被失败的回调捕获了，那么不会被catch捕获到
read('./1.txt','utf8')
.then((data)=>{
    throw new Error('出错了')
})
.then(null,(err)=>{
    console.log(err) // Error:出错了   报错
})
.catch((err)=>{
    console.log(err)  // 不会执行到这里
})
```
* 5.catch是错误没有处理的情况下才会执行
* 6.then中可以不写东西

#### 穿插一个与jquery的链式调用区别
jquery的链式调用，是通过其内部执行完后return this，返回自身这个对象，达到链式调用的目的，那为什么Promise不采用这种方式呢？

我们可以看以下代码，感受一下。
```
let promise = new Promise((resolve,reject)=>{
    resolve() // 执行resolve，使状态从pending变为resolved
})

let promise2 = promise.then(()=>{
    throw new Error() // 抛出错误
    return this // 返回自身
})

// 那么我在promise2中，调then，那么它会执行失败的回调吗？答案是不会的。
// 因为我们不可能让状态既成功又失败的
// promise成功了，如果返回this，那不能走向失败
promise2.then(()=>{
    console.log('来到这里了') 
},()=>{
    console.log('会来到这里吗？')
})

// 此时then中返回自身后，promise2其实就是promise，而我们想达到
// 的是把当前的then返回后，传到下一个then中，但是我们这样返回this，
// 其实会变得很矛盾，因为状态已经从pending变为resolved，不可能又从resolved变成rejected的
// 所以得出结论，返回的必须是一个新的promise，因为promise成功后不能再走失败
// 只能创建一个新的promise再执行业务逻辑，返回同一个promise的话，就不能既成功又失败
```

##### 实现Promise链式调用
```
// ----- 代码部分
let promise = new Promise((resolve,reject)=>{
    resolve()
})

// 2.返回的值为promise2 为什么这样规定呢？这是promiseA+规范规定的，我们要遵循
let promise2 = promise.then((data)=>{
    return x // 1.then中的返回值x可能是普通值也可能是promise，并且传给下一个then
}).then((data)=>{
    console.log(data) // x的值
})
```
```
// ----- 源码实现部分
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined 
    self.status = 'pending'
    self.onResolvedCallbacks = []  
    self.onRejectedCallbacks = []
    function resolve(value){
        if(self.status === 'pending'){ 
            self.value = value 
            self.status = 'resolved'
            self.onResolvedCallbacks.forEach(fn=>fn())
        }
    }
    
    function reject(reason){
        if(self.status === 'pending'){ 
            self.reason = reason 
            self.status = 'rejected'
            self.onRejectedCallbacks.forEach(fn=>fn())
        }
    }
    
    try{
        executor(resolve,reject)
    }catch(error){
        reject(error)
    }
}

Promise.prototype.then = function(onFulfilled,onRejected){
    let self = this
    let promise2 // 3.上面讲promise链式调用时，已经说了返回的是一个新的promise对象，那么我们声明一个新的promise
    
    // 4.那么我们new一个新的promise，并且把以下代码放到promise中
    let promise2 = new Promise((resolve,reject)=>{
        if(self.status === 'resolved'){
            // 7.当执行成功回调的时候，可能会出现异常，那么就把这个异常作为promise2的错误的结果
            try{
                let x = onFulfilled(self.value) // 6.这里的x，就是上面then中执行完返回的结果，我们在这里声明一个x用来接收
                // 8.根据promiseA+规范，我们应该提供一个函数来处理promise2
                //   我个人的理解是，then中不管是成功回调还是失败回调，其返回
                //   值，有可能是promise，也有可能是普通值，也有可能是抛出错误
                //   那么我们就需要一个函数来处理这几种不同的情况
                //   这个函数我们声明为resolvePromise吧
                resolvePromise(promise2,x,resolve,reject)
                // 9. 这里的promise2就是当前的promise2，x则是执行then中成功回调后返回的结果，如果是成功则调promise2的resolve，失败则调reject
            }catch(e){
                reject(e) // 注意：这里的reject是这个promise2的reject
            }
        }
        
        if(self.status === 'rejected'){
            // 同6-7步
            try{
                let x = onRejected(self.reason) 
                // 同8-9
                resolvePromise(promise2,x,resolve,reject)
            }catch(e){
                reject(e)
            }
        }
    
        if(self.status === 'pending'){
            self.onResolvedCallbacks.push(()=>{
                // 同6-7步
                try{
                    let x =  onFulfilled(self.value)
                    // 同8-9
                    resolvePromise(promise2,x,resolve,reject)
                }catch(e){
                    reject(e)
                }
            }) 
            self.onRejectedCallbacks.push(()=>{
                // 同6-7步
                try{
                    let x = onRejected(self.reason)
                    // 同8-9
                    resolvePromise(promise2,x,resolve,reject)
                }catch(e){
                    reject(e)
                }
            })
        }
    })
    return promise2 // 5.在jquery中是return this，但是在promise中，则是返回一个新的promise对象
}

module.exports = Promise
```
#### 写一个resolvePromise函数
接下来我们写一下resolvePromise这个函数，整个Promise最核心的部分就是在这里
```
// ----- 代码部分
let promise = new Promise((resolve,reject)=>{
    resolve()
})

let promise2 = promise.then((data)=>{
    return x 
}).then((data)=>{
    console.log(data) 
})

// 2.我们在resolvePromise函数中，在原生情况下，如果传参的时候，promise2和x是同一个对象会发生什么呢？
let promise = new Promise((resolve,reject)=>{
    resolve()
})

let promise2 = promise.then(()=>{
    return promise2 
    // 2.1报错 UnhandledPromiseRejectionWarning: TypeError: Chaining cycle detected for promise #<Promise>
    // 报错的意思是，陷入了死循环，那怎么理解呢？
    // promise2的成功或失败是要取决于promise中then的返回结果，而返回的却是promi2自己
    // 这样就陷入死循环了，promise2是依赖于promise的then返回的结果，
    // 而then返回的结果是promise2，而then中的promise2，既不是成功也不是失败，不能自己等于自己
})

// 7.当取一个对象上的属性，可能存在报异常的情况，怎么理解呢？
// 因为这个方法有可能不是自己写的，可能别人搞恶作剧乱写的，看以下代码。
let obj = {}
// 给obj对象定义一个then方法，当我们去obj对象中调用then方法时
// 就会执行里面的get，而get则是抛出异常
Object.defineProperty(obj,'then',{
    get(){
        throw new Error()
    }
})

// 10.为什么要用call呢？解决了什么问题？看一下以下代码
首先我们执行
promise.then(()=>{
    console.log(this) // 此时this是指向该promise的，对象的方法中this是指向这个对象的
})

但是我们在下面通过let then = promise.then，来判断是否promise，是否会异常
当我们执行then时，里面的this还是会指向这个promise吗？答案是不一定的，
因为此时then，如果在全局下执行，指向的可能就是window了，所以为了让this的
指向正确，我们需要通过
then.call(promise)，来把then的this指向promise
```
```
// ----- 源码实现部分
function Promise(executor){
    let self = this
    self.value = undefined
    self.reason = undefined 
    self.status = 'pending'
    self.onResolvedCallbacks = []  
    self.onRejectedCallbacks = []
    function resolve(value){
        if(self.status === 'pending'){ 
            self.value = value 
            self.status = 'resolved'
            self.onResolvedCallbacks.forEach(fn=>fn())
        }
    }
    
    function reject(reason){
        if(self.status === 'pending'){ 
            self.reason = reason 
            self.status = 'rejected'
            self.onRejectedCallbacks.forEach(fn=>fn())
        }
    }
    
    try{
        executor(resolve,reject)
    }catch(error){
        reject(error)
    }
}

// 1.声明一个resolvePromise函数
// 这个函数非常核心，所有的promise都遵循这个规范，所有的promise可以通用，
/**
 * 
 * @param {*} promise2 then的返回值，返回新的promise
 * @param {*} x then中成功函数或者失败函数的返回值
 * @param {*} resolve promise2的resolve
 * @param {*} reject promise2的reject
 */
function resolvePromise(promise2,x,resolve,reject){
    // 3.从2中我们可以得出，自己不能等于自己
    // 当promise2和x是同一个对象的时候，则走reject
    if(promise2 === x){
        return reject(new TypeError('Chaining cycle detected for promise'))
    }
    // 4.因为then中的返回值可以为promise，当x为对象或者函数，才有可能返回的是promise
    let called
    if(x !== null && (typeof x === 'object' || typeof x === 'function')){
        // 8.从第7步，可以看出为什么会存在抛出异常的可能，所以使用try catch处理
        try{
            // 6.因为当x为promise的话，是存在then方法的
            // 但是我们取一个对象上的属性，也有可能出现异常，我们可以看一下第7步
            let then = x.then 
            
            // 9.我们为什么在这里用call呢？解决了什么问题呢？可以看上面的第10步
            // x可能还是个promise，那么就让这个promise执行
            // 但是还是存在一个恶作剧的情况，就是{then:{}}
            // 此时需要新增一个判断then是否函数
            if(typeof === 'function'){
                then.call(x,(y)=>{ // y是返回promise后的成功结果
                    // 一开始我们在这里写的是resolve(y)，但是考虑到一点
                    // 这个y，有可能还是一个promise，
                    // 也就是说resolve(new Promise(...))
                    // 所以涉及到递归，我们把resolve(y)改成以下
                    
                    // 12.限制既调resolve，也调reject
                    if(called) return
                    called = true
                    
                    resolvePromise(promise2,y,resolve,reject)
                    // 这样的话，代码会一直递归，取到最后一层promise
                    
                    // 11.这里有一种情况，就是不能既调成功也调失败，只能挑一次，
                    // 但是我们前面不是处理过这个情况了吗？
                    // 理论上是这样的，但是我们前面也说了，resolvePromise这个函数
                    // 是所有promise通用的，也可以是别人写的promise，如果别人
                    // 的promise可能既会调resolve也会调reject，那么就会出问题了，所以我们接下来要
                    // 做一下限制，这个我们写在第12步
                    
                },(err)=>{ // err是返回promise后的失败结果
                    if(called) return
                    called = true
                    reject(err)
                })
            }else{
                resolve(x) // 如果then不是函数的话，那么则是普通对象，直接走resolve成功
            }
        }catch(e){ // 当出现异常则直接走reject失败
            if(called) return
            called = true
            reject(e)
        }
    }else{ // 5.x为一个常量，则是走resolve成功
        resolve(x)
    }
}

Promise.prototype.then = function(onFulfilled,onRejected){
    // onFulfilled、onRejected是可选参数
    onFulfilled = typeof onFulfilled === 'function'?onFulfilled:val=>val;
    onRejected = typeof onRejected === 'function'?onRejected: err=>{throw err}
    let self = this
    let promise2 
    let promise2 = new Promise((resolve,reject)=>{
        if(self.status === 'resolved'){
            // 13.根据promiseA+规范，onFulfilled或onRejected必须
            // 被调用不是当前的上下文，then方法是异步的
            setTimeout(()=>{
                try{
                    let x = onFulfilled(self.value)
                    resolvePromise(promise2,x,resolve,reject)
                }catch(e){
                    reject(e) 
                }
            },0)
        }
        
        if(self.status === 'rejected'){
            // 同13
            setTimeout(()=>{
                try{
                    let x = onRejected(self.reason) 
                    resolvePromise(promise2,x,resolve,reject)
                }catch(e){
                    reject(e)
                }
            },0)
        }
    
        if(self.status === 'pending'){
            self.onResolvedCallbacks.push(()=>{
                // 同13
                setTimeout(()=>{
                    try{
                        let x =  onFulfilled(self.value)
                        resolvePromise(promise2,x,resolve,reject)
                    }catch(e){
                        reject(e)
                    }
                },0)
            }) 
            self.onRejectedCallbacks.push(()=>{
                // 同13
                setTimeout(()=>{
                    try{
                        let x = onRejected(self.reason)
                        resolvePromise(promise2,x,resolve,reject)
                    }catch(e){
                        reject(e)
                    }
                },0)
            })
        }
    })
    return promise2
}

// 14.到目前为止，根据promiseA+规范的代码写得差不多了，我们可以通过测试代码来测试我们是否写得正确，下面我们写一段测试代码

Promise.defer = Promise.deferred = function(){
    let dfd = {}
    dfd.promise = new Promise((resolve,reject)=>{
        dfd.resolve = resolve
        dfd.reject = reject
    })
    return dfd
}

// 14.接下来我们要安装一个插件，npm install promises-aplus-test -g

module.exports = Promise
```
```
// 完整代码 也顺便带大家理顺一下
function Promise(executor) {
    let self = this;
    self.value = undefined;  // 成功的值
    self.reason = undefined;  // 失败的值
    self.status = 'pending'; // 目前promise的状态pending
    self.onResolvedCallbacks = []; // 可能new Promise的时候会存在异步操作，把成功和失败的回调保存起来
    self.onRejectedCallbacks = [];
    function resolve(value) { // 把状态更改为成功
        if (self.status === 'pending') { // 只有在pending的状态才能转为成功态
            self.value = value;
            self.status = 'resolved';
            self.onResolvedCallbacks.forEach(fn => fn()); // 把new Promise时异步操作，存在的成功回调保存起来
        }
    }
    function reject(reason) {  // 把状态更改为失败
        if (self.status === 'pending') { // 只有在pending的状态才能转为失败态
            self.reason = reason;
            self.status = 'rejected';
            self.onRejectedCallbacks.forEach(fn => fn()); // 把new Promise时异步操作，存在的失败回调保存起来
        }
    }
    try {
        // 在new Promise的时候，立即执行的函数，称为执行器
        executor(resolve, reject);
    } catch (e) { // 如果执行executor抛出错误，则会走失败reject
        reject(e);
    }
}

// 这个函数为核心，所有的promise都遵循这个规范
// 主要是处理then中返回的值x和promise2的关系
function resolvePromise(promise2,x,resolve,reject){
    // 当promise2和then返回的值x为同一个对象时，变成了自己等自己，会陷入死循环
    if(promise2 === x){
        return reject(new TypeError('Chaining cycle'));
    }
    let called;
    // x可能是一个promise也可能是一个普通值
    if(x!==null && (typeof x=== 'object' || typeof x === 'function')){
        try{
            let then = x.then; 
            if(typeof then === 'function'){
                then.call(x,y=>{ 
                    if(called) return; 
                    called = true;
                    resolvePromise(promise2,y,resolve,reject);
                },err=>{ 
                    if(called) return;
                    called = true;
                    reject(err);
                });
            }else{
                resolve(x);
            }
        }catch(e){
            if(called) return;
            called = true;
            reject(e);
        }
    }else{ 
        resolve(x);
    }
}

// then调用的时候，都是属于异步，是一个微任务
// 微任务会比宏任务先执行
// onFulfilled为成功的回调，onRejected为失败的回调
Promise.prototype.then = function (onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function'?onFulfilled:val=>val;
    onRejected = typeof onRejected === 'function'?onRejected: err=>{throw err}
    let self = this;
    let promise2;
    // 上面讲了，promise和jquery的区别，promise不能单纯返回自身，
    // 而是每次都是返回一个新的promise，才可以实现链式调用，
    // 因为同一个promise的pending resolve reject只能更改一次
    promise2 = new Promise((resolve, reject) => {
        if (self.status === 'resolved') {
            // 为什么要加setTimeout？
            // 首先是promiseA+规范要求的
            // 其次是大家写的代码，有的是同步，有的是异步
            // 所以为了更加统一，就使用为setTimeout变为异步了，保持一致性
            setTimeout(()=>{
                try { // 上面executor虽然使用try catch捕捉错误
                      // 但是在异步中，不一定能够捕捉，所以在这里
                      // 用try catch捕捉
                    let x = onFulfilled(self.value);
                    // 在then中，返回值可能是一个promise，所以
                    // 需要resolvePromise对返回值进行判断
                    resolvePromise(promise2,x,resolve,reject);
                } catch (e) {
                    reject(e);
                }
            },0)
        }
        if (self.status === 'rejected') {
            setTimeout(()=>{
                try {
                    let x = onRejected(self.reason);
                    resolvePromise(promise2,x,resolve,reject);
                } catch (e) {
                    reject(e);
                }
            },0)
        }
        if (self.status === 'pending') {
            self.onResolvedCallbacks.push(() => {
                setTimeout(()=>{
                    try {
                        let x = onFulfilled(self.value);
                        resolvePromise(promise2,x,resolve,reject);
                    } catch (e) {
                        reject(e);
                    }
                },0)
            });
            self.onRejectedCallbacks.push(() => {
                setTimeout(()=>{
                    try {
                        let x = onRejected(self.reason);
                        resolvePromise(promise2,x,resolve,reject);
                    } catch (e) {
                        reject(e);
                    }
                },0)
            });
        }
    });
    return promise2
}
Promise.defer = Promise.deferred = function(){
    let dfd = {};
    dfd.promise = new Promise((resolve,reject)=>{
        dfd.resolve = resolve;
        dfd.reject = reject;
    })
    return dfd;
}
module.exports = Promise;
```
```
执行promises-aplus-tests promise.js
```

![](https://user-gold-cdn.xitu.io/2018/8/11/16528bafb4f82b88?w=1672&h=1404&f=png&s=425769)

![](https://user-gold-cdn.xitu.io/2018/8/11/16528bb3f8c390f6?w=1672&h=1404&f=png&s=359252)

到此为止，我们已经写了一个符合promiseA+规范的promise了，大家可以好好多看几次。
***
#### 接下来，我们完善一下这个promise，写一下常用的promise方法

* Promise.reject
* Promise.resolve
* catch
* Promise.all
* Promise.race

```
Promise.reject = function(reason){
    return new Promise((resolve,reject)=>{
        reject(reason);
    })
}

Promise.resolve = function(value){
    return new Promise((resolve,reject)=>{
        resolve(value);
    })
}

Promise.prototype.catch = function(onRejected){
    return this.then(null,onRejected);
};

Promise.all = function(promises){
    return new Promise((resolve,reject)=>{
        let arr = [];
        let i = 0;
        function processData(index,data){
            arr[index] = data;
            if(++i == promises.length){
                resolve(arr);
            }
        }
        for(let i = 0;i<promises.length;i++){
            promises[i].then(data=>{
                processData(i,data);
            },reject);
        }
    })
}

Promise.race = function(promises){
    return new Promise((resolve,reject)=>{
        for(let i = 0;i<promises.length;i++){
            promises[i].then(resolve,reject);
        }
    })
}
```
***
#### Promise的常用方法如何实现呢？
##### Promise.resolve / Promise.reject
```
// 原生的Promise.resolve使用
Promise.resolve('hello swr').then((data)=>{ // 直接把成功的值传递给下一个then
    console.log(data) // hello swr
})

// 那么Promise.resolve内部是怎么实现的呢？
Promise.resolve = function(value){
    return new Promise((resolve,reject)=>{ // 在内部new一个Promise对象
       resolve(value) 
    })
}

// 同理，Promise.reject内部也是类似实现的
Promise.reject = function(reason){
    return new Promise((resolve,reject)=>{
        reject(reason)
    })
}
```

##### catch是怎样实现呢？
```
// 原生Promise的catch使用
Promise.reject('hello swr').catch((e)=>{
    console.log(e) // hello swr
})

// 上面这段代码相当于下面这段代码
Promise.reject('hello swr').then(null,(e)=>{ // then里直接走了失败的回调
    console.log(e) // hello swr
})

// 内部实现
Promise.prototype.catch = function(onRejected){
    return this.then(null,onRejected) // 相当于then里的成功回调只传个null
}


```

##### Promise.all，这个方法非常重要，同时执行多个异步，并且返回一个新的promise，成功的值是一个数组，该数组成员的顺序是传参给Promise.all的顺序
```
// 原生Promise.all的使用
// 假设1.txt内容为hello 2.txt内容为swr
let fs = require('fs')
function read(filePath,encoding){
    return new Promise((resolve,reject)=>{ 
        fs.readFile(filePath,encoding,(err,data)=>{
            if(err) reject(err)
            resolve(data)
        })
    })
}

Promise.all([read('./1.txt','utf8'),read('./2.txt','utf8')]).then((data)=>{
    console.log(data) // 全部读取成功后返回 ['hello','swr']
                      // 需要注意的是，当其中某个失败的话，则会走失败的回调函数
})

// 内部实现
Promise.all = function(promises){ // promises 是一个数组
    return new Promise((resolve,reject)=>{
        let arr = []
        let i = 0
        function processData(index,data){
            arr[index] = data
            // 5.我们能用arr.length === promises.length来判断请求是否全部完成吗？
            // 答案是不行的，假设arr[2] = 'hello swr'
            // 那么打印这个arr，将是[empty × 2, "hello swr"]，
            // 此时数组长度也是为3，而数组arr[0] arr[1]则为空
            // 那么换成以下的办法
            if(++i === promises.length){ // 6.利用i自增来判断是否都成功执行
                resolve(arr) // 此时arr 为['hello','swr']
            }
        }
        
        for(let i = 0;i < promises.length;i++){ // 1.在此处遍历执行
            promises[i].then((data)=>{ // 2.data是成功后返回的结果
                processData(i,data) // 4.因为Promise.all最终返回的是一个数组成员按照顺序排序的数组
                                    // 而且异步执行，返回并不一定按照顺序
                                    // 所以需要传当前的i
            },reject) // 3.如果其中有一个失败的话，则调用reject
        }
    })
}
```
##### Promise.race 该方法是同时执行多个异步，然后哪个快，就用哪个的结果，race的意思是赛跑
```
// 原生Promise.race的使用
// 一个成功就走成功的回调，一个失败就走失败的回调
Promise.race([read('./1.txt','utf8'),read('./2.txt','utf8')]).then((data)=>{
    console.log(data) // 可能返回 'hello' 也可能返回 'swr' 看哪个返回快就用哪个作为结果
})

// 内部实现
Promise.race = function(promises){ // promises 是一个数组
    return new Promise((resolve,reject)=>{
        for(let i = 0;i < promises.length;i++){ 
            promises[i].then(resolve,reject) // 和上面Promise.all有点类似
        }
    })
}
```

***
#### Promise.defer = Promise.deferred 这个语法糖怎么理解呢？

这个语法糖可以简化一些操作，比如
```
let fs = require('fs')
// 写法一：
function read(filePath,encoding){
    // 这里的new Promise依然是传递了一个executor回调函数
    // 我们该怎样减少回调函数嵌套呢？
    return new Promise((resolve,reject)=>{ 
        fs.readFile(filePath,encoding,(err,data)=>{
            if(err) reject(err)
            resolve(data)
        })
    })
}

// 写法二：
// 这样的写法减少了一层回调函数的嵌套
function read(filePath,encoding){
    let dfd = Promise.defer()
    fs.readFile(filePath,encoding,(err,data)=>{
        if(err) dfd.reject(err)
        dfd.resolve(data)
    })
    return dfd.promise
}

read('./1.txt','utf8').then((data)=>{
    console.log(data)
})


```
***
结尾：第一次写，都是想到哪写到哪，请大家多多谅解~ 也希望对大家有所帮助，promise的源码实现，我最大的收获并不是怎么实现promise，而是编程思维，大家可以多多往深里想一想，也希望大家可以和我进行交流，共同进步