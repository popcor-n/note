### 首先
*一点点 setTimeout：*
setTimeout超时调用，相比于setInterval间歇调用来说，在功能上可能有些不足。但是在实际使用中我们会发现，setTimeout的使用率是高于setInterval的，
就算需要用到间歇调用，我们还是会使用setTimeout来模拟实现。那么我们应该了解这种情况的原因以及怎么去实现代替setInterval的功能
### 原因
> “在开发环境下，很少使用间歇调用（setInterval），原因是后一个间歇调用很可能在前一个间歇调用结束前启动”。
在间歇调用的过程中，setInterval的做法是，在开始执行前面一个方法之前，就把下一个方法放进了事件队列里面。那么如果第一个方法的执行事件超过了设定的时间间隔
就有可能实现后一个任务在前一个结束之前就开始启动了。
### 想法
既然 setInterval可能会影响实际开发，要做的当然就是实现用其他方法实现相同或相似功能以供使用。setTimeout作为间歇调用的代替品很容易被想到了。但是在使用之前
应该更好的了解一下setTimeout本身 以及 使用setTimeout反映出来的问题。这样才能在使用中更加得心应手

```
console.log ('1');
setTimeout ( function() {console.log ('2')}, 0);
console.log ('3');
```
尽管设置间隔时间为0，但是由于js的运行机制，结果是 132。如果在复习的时候看到这种例子能直接想到js是单线程的，那就最好不过了。
### js单线程
耳熟不一定能详，Js引擎是单线程事件驱动的。js引擎在不断地等待着任务队列的任务并加以执行，但是任何时刻，他只能做一件事，这也是我们遇见很多次的阻塞式执行。
与原理相悖的是，我们在绝大多数情况下并不喜欢这种所谓阻塞式的执行。所以有了异步执行来解决眼下的问题。异步提供了非阻塞的方法，是解决耗时严重或者时间不确定
的逻辑的必然之选。
异步说法有些空洞，具体的实现多任务的处理采用了 任务队列 这一结构来处理。其中分为同步任务和异步任务。
* 同步任务：在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。


  * 1 输出
  如：console.log()
  * 2 变量的声明，等
  * 3 同步函数：如果在函数返回的时候，调用者就能够拿到预期的返回值或者看到预期的效果，那么这个函数就是同步的。

* 异步任务

  * 1 setTimeout和setInterval
  * 2 DOM事件
  * 3 Promise
  * 4 process.nextTick，等
  异步函数：如果在函数返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。
  除此之外，任务队列又分为macro-task（宏任务）与micro-task（微任务），在ES5标准中，它们被分别称为task与job。
* 宏任务

  * I/O
  * setTimeout
  * setInterval
  * setImmdiate
  * requestAnimationFrame

* 微任务

  * process.nextTick
  * Promise
  * Promise.then
  * MutationObserver
  每次事件循环中，先执行宏任务队列的一个任务，然后会执行微任务直至微任务队列被清空。
  话说回来，刚才的setTimeout的执行过程到底是什么样的？
  在执行时，会把setTimeout的代码屏蔽掉，直到下一次事件循环时判断是否到了指定的间隔时间，如果没到就继续刚才的过程，反之执行他。
  ### 问题
  在复习了基础知识之后 ，怎么去实现用setTimeout来解决间歇调用呢？
  一个例子，他是错的：
  ```
    for (var i = 0; i < 4; i++) {
      setTimeout(function () {
          console.log(i);
      }, 1000);
   }
  ```
我们的想法是在循环体内，每次把事件放进任务队列中，并且在指定的事件间隔来执行。看起来没错，但是答案是隔了一秒后直接输出4444 那么问题出在哪里了？
在浏览器中，是不是每个异步事件都会直接放进异步队列里面呢？答案是否定的。浏览器有定时器模块（timer），具体的规定和在正常情况下使用setTimeout如果把间隔
时间调整到0，是什么情况呢？
>HTML5 标准规定了setTimeout()的第二个参数的最小值，即最短间隔，不得低于4毫秒。如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。
> 即使setTimeout设置的执行时间为0毫秒，也按4毫秒算。


然而再循环中 4次循环花费的时间代价非常小，所以其中的操作根本就没有放进异步队列，在执行过程中相当于外部访问了循环体内的i，出发了闭包而导致了4444的情况
### 解决


```
  //立即执行：
  for (var i = 0; i < 4; i++) {
      (function (j) {
          setTimeout(function () {
              console.log(j);
          }, 1000 * i)
      })(i);
  }
  //使用let会省去一些麻烦
  for ( let i = 0; i < 4; i++) {
          setTimeout(function () {
              console.log(i);
          }, 1000 * i)
  }

  //扔在外面处理，按值传递参数
  var output = function (i) {
      setTimeout(function () {
          console.log(i);

      }, 1000 * i)
  }
  for (let i = 0; i < 4; i++) {
      output(i);
  }

  //或者使用promise更高级的做法：
  const tasks = [];

  const output = (i) => new Promise((resolve) => {
      setTimeout(() => {
          console.log(i);
          resolve();
      }, 1000 * i);

  });

  //生成全部的异步操作
  for (var i = 0; i < 4; i++) {
      tasks.push(output(i));
  }
  //同步操作完成后，输出最后的i
  Promise.all(tasks).then(() => {
      setTimeout(() => {
          console.log(i);
      }, 1000)
  })

```
最后想说的是，setTimeout的间隔时间并不准确，是否及时执行取决于 JavaScript 线程是否拥挤。
运行过程是，发现了setTimeout后，不会立刻放到异步队列，继续执行同步任务，在timer规定时间之后，放进异步队列。同步队列没有要继续执行的任务后才会开始读取
异步队列的任务，放进运行栈中才会开始执行。这中间的时间间隔不准确性是很大的
