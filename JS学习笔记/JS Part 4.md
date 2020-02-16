# you don't know JS 学习笔记

## 回调

### continuation

回调函数包裹或者说封装了程序的延续（continuation）。

### 小结

回调函数是 JavaScript 异步的基本单元。但是随着 JavaScript 越来越成熟，对于异步编程领域的发展，回调已经不够用了。

第一，大脑对于事情的计划方式是线性的、阻塞的、单线程的语义，但是回调表达异步流程的方式是非线性的、非顺序的，这使得正确推导这样的代码难度很大。难于理解的代码是坏代码，会导致坏 bug。

我们需要一种更同步、更顺序、更阻塞的的方式来表达异步，就像我们的大脑一样。

第二，也是更重要的一点，回调会受到控制反转的影响，因为回调暗中把控制权交给第三方（通常是不受你控制的第三方工具！）来调用你代码中的 continuation。这种控制转移导致一系列麻烦的信任问题，比如回调被调用的次数是否会超出预期。

可以发明一些特定逻辑来解决这些信任问题，但是其难度高于应有的水平，可能会产生更笨重、更难维护的代码，并且缺少足够的保护，其中的损害要直到你受到 bug 的影响才会被发现。

我们需要一个通用的方案来解决这些信任问题。不管我们创建多少回调，这一方案都应可以复用，且没有重复代码的开销。

## Promise

如果我们不把自己程序的continuation 传给第三方，而是希望第三方给我们提供了解其任务何时结束的能力，然后由我们自己的代码来决定下一步做什么，那将会怎样呢？

这种范式就称为 Promise。

### 完成事件

```javascript
function foo(x) {
  // 开始做点可能耗时的工作
  
  // 构造一个listener事件通知处理对象来返回
  return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){
  // 可以进行下一步了！
} );

evt.on( "failure", function(err){
  // 啊，foo(..)中出错了
} ); 
```

foo(..) 显式创建并返回了一个事件订阅对象，调用代码得到这个对象，并在其上注册了两个事件处理函数。相对于面向回调的代码，这里的反转是显而易见的，而且这也是有意为之。**这里没有把回调传给 foo(..)，而是返回一个名为 evt 的事件注册对象，由它来接受回调。**

一个很重要的好处是，可以把这个事件侦听对象提供给代码中多个独立的部分；在foo(..) 完成的时候，它们都可以独立地得到通知，以执行下一步：

```javascript
var evt = foo( 42 );
// 让bar(..)侦听foo(..)的完成
bar( evt );
// 并且让baz(..)侦听foo(..)的完成
baz( evt );
```

对控制反转的恢复实现了**更好的关注点分离**，其中 bar(..) 和 baz(..) 不需要牵扯到foo(..) 的调用细节。类似地，foo(..) 不需要知道或关注 bar(..) 和 baz(..) 是否存在，或者是否在等待 foo(..) 的完成通知。

从本质上说，**evt 对象就是分离的关注点之间一个中立的第三方协商机制。**

事件侦听对象 evt 就是 Promise 的一个模拟。

在基于 Promise 的方法中，前面的代码片段会让 foo(..) 创建并返回一个 Promise 实例，而且这个 Promise 会被传递到 bar(..) 和 baz(..)。

```javascript
function foo(x) {
  // 可是做一些可能耗时的工作
  // 构造并返回一个promise
  return new Promise( function(resolve,reject){
    // 最终调用resolve(..)或者reject(..)
    // 这是这个promise的决议回调
  } );
}

var p = foo( 42 );

bar( p );
baz( p ); 
```

new Promise( function(..){ .. } ) 模式通常称为 revealing constructor。**传入的函数会立即执行**（不会像 then(..) 中的回调一样异步延迟），它有两个参数，在本例中我们将其分别称为 resolve 和 reject。这些是 promise 的决议函数。resolve(..) 通常标识完成，而 reject(..) 则标识拒绝。

```javascript
function bar(fooPromise) {
  // 侦听foo(..)完成
  fooPromise.then(
    function(){
      // foo(..)已经完毕，所以执行bar(..)的任务
    },
    function(){
      // 啊，foo(..)中出错了！
    }
  );
}
// 对于baz(..)也是一样
```

Promise 决议并不一定要像前面将 Promise 作为未来值查看时一样会涉及发送消息。它也可以只作为一种流程控制信号，就像前面这段代码中的用法一样。另外一种实现方式是：

```javascript
function bar() {
 // foo(..)肯定已经完成，所以执行bar(..)的任务
}
function oopsBar() {
 // 啊，foo(..)中出错了，所以bar(..)没有运行
}
// 对于baz()和oopsBaz()也是一样
var p = foo( 42 );

p.then( bar, oopsBar );
p.then( baz, oopsBaz ); 
```

这里没有把 promise p 传给 bar(..) 和 baz(..)，而是使用 promise 控制 bar(..) 和 baz(..)何时执行，如果执行的话。最主要的区别在于错误处理部分。

在第一段代码的方法里，不论 foo(..) 成功与否，bar(..) 都会被调用。并且如果收到了foo(..) 失败的通知，它会亲自处理自己的回退逻辑。显然，baz(..) 也是如此。

在第二段代码中，bar(..) 只有在 foo(..) 成功时才会被调用，否则就会调用 oppsBar(..)。baz(..) 也是如此。

另外，两段代码都以使用 promise p 调用 then(..) 两次结束。这个事实说明了前面的观点，就是 Promise（一旦决议）一直保持其决议结果（完成或拒绝）不变，可以按照需要多次查看。

一旦 p 决议，不论是现在还是将来，下一个步骤总是相同的。

### 具有then方法的鸭子类型

在 Promise 领域，一个重要的细节是如何确定某个值是不是真正的 Promise。或者更直接地说，它是不是一个行为方式类似于 Promise 的值？

识别 Promise（或者行为类似于 Promise 的东西）就是定义某种称为 thenable 的东西，将其定义为任何具有 then(..) 方法的对象和函数。我们认为，任何这样的值就是Promise 一致的 thenable。 thenable值的鸭子类型检测就大致类似于：

```javascript
if (
  p !== null &&
  (
    typeof p === "object" ||
    typeof p === "function"
  ) &&
  typeof p.then === "function"
) {
  // 假定这是一个thenable!
}
else {
  // 不是thenable
} 
```

如果你试图使用恰好有 then(..) 函数的一个对象或函数值完成一个 Promise，但并不希望它被当作 Promise 或 thenable，那就有点麻烦了，因为它会自动被识别为 thenable，并被按照特定的规则处理。

### Promise 信任问题

把一个回调传入工具 foo(..) 时可能出现如下问题：
* 调用回调过早；
* 调用回调过晚（或不被调用）；
* 调用回调次数过少或过多；
* 未能传递所需的环境和参数；
* 吞掉可能出现的错误和异常。

Promise 的特性就是专门用来为这些问题提供一个有效的可复用的答案。

#### 调用过早

对一个 Promise 调用 then(..) 的时候，即使这个 Promise 已经决议，提供给then(..) 的回调也总会被异步调用。

#### 调用过晚

Promise 创建对象调用 resolve(..) 或 reject(..) 时，这个 Promise 的then(..) 注册的观察回调就会被自动调度。可以确信，这些被调度的回调在下一个异步事件点上一定会被触发。

#### 回调未调用

首先，没有任何东西（甚至 JavaScript 错误）能阻止 Promise 向你通知它的决议（如果它决议了的话）。如果你对一个 Promise 注册了一个完成回调和一个拒绝回调，那么 Promise在决议时总是会调用其中的一个。

但是，如果 Promise 本身永远不被决议呢？即使这样，Promise 也提供了解决方案，其使用
了一种称为竞态的高级抽象机制：

```javascript
// 用于超时一个Promise的工具
function timeoutPromise(delay) {
  return new Promise( function(resolve,reject){
      setTimeout( function(){
        reject( "Timeout!" );
      }, delay );
    } );
}
// 设置foo()超时
Promise.race( [
  foo(), // 试着开始foo()
  timeoutPromise( 3000 ) // 给它3秒钟
] )
.then(
  function(){
    // foo(..)及时完成！
  }, 
  function(err){
    // 或者foo()被拒绝，或者只是没能按时完成
    // 查看err来了解是哪种情况
  }
); 
```

很重要的一点是，我们可以保证一个 foo() 有一个输出信号，防止其永久挂住程序。

#### 调用次数过多或过少

回调被调用的正确次数应该是 1。

“过少”的情况就是调用 0 次，和前面解释过的“未被”调用是同一种情况。

“过多”的情况很容易解释。Promise 的定义方式使得它只能被决议一次。如果出于某种原因，Promise 创建代码试图调用 resolve(..) 或 reject(..) 多次，或者试图两者都调用，那么这个 Promise 将只会接受第一次决议，并默默地忽略任何后续调用。

由于 Promise 只能被决议一次，所以任何通过 then(..) 注册的（每个）回调就只会被调用一次。

#### 未能传递参数/环境值

如果你没有用任何值显式决议，那么这个值就是 undefined，这是 JavaScript 常见的处理方式。但不管这个值是什么，无论当前或未来，它都会被传给所有注册的（且适当的完成或拒绝）回调。

如果要传递多个值，你就必须要把它们封装在单个值中传递，比如通过一个数组或对象。

#### 吞掉错误或异常

如果拒绝一个 Promise 并给出一个理由（也就是一个出错消息），这个值就会被传给拒绝回调。

如果在 Promise 的创建过程中或在查看其决议结果过程中的任何时间点上出现了一个 JavaScript 异常错误，比如一个 TypeError 或ReferenceError，那这个异常就会被捕捉，并且会使这个 Promise 被拒绝。

```javascript
var p = new Promise( function(resolve,reject){
  foo.bar(); // foo未定义，所以会出错！
  resolve( 42 ); // 永远不会到达这里 :(
} );

p.then(
  function fulfilled(){
    // 永远不会到达这里 :(
  },
  function rejected(err){
    // err将会是一个TypeError异常对象来自foo.bar()这一行
  }
);
```

foo.bar() 中发生的 JavaScript 异常导致了 Promise 拒绝，你可以捕捉并对其作出响应,进而极大降低了竞态条件出现的可能。

如果 Promise 完成后在查看结果时（then(..) 注册的回调中）出现了 JavaScript 异常错误会怎样呢？

```javascript
var p = new Promise( function(resolve,reject){
  resolve( 42 );
} );

p.then(
  function fulfilled(msg){
    foo.bar();
    console.log( msg ); // 永远不会到达这里 :(
  },
  function rejected(err){ 
    // 永远也不会到达这里 :(
  }
);
```

这看起来像是 foo.bar() 产生的异常真的被吞掉了。别担心，实际上并不是这样。但是这里有一个深藏的问题，就是我们没有侦听到它。**p.then(..) 调用本身返回了另外一个 promise，正是这个 promise 将会因 TypeError 异常而被拒绝。**

为什么它不是简单地调用我们定义的错误处理函数呢？表面上的逻辑应该是这样啊。如果这样的话就违背了 Promise 的一条基本原则，即 Promise 一旦决议就不可再变。p 已经完成为值 42，所以之后查看 p 的决议时，并不能因为出错就把 p 再变为一个拒绝。

#### 是可信任的Promise吗

到 Promise 并没有完全摆脱回调。它们只是改变了传递回调的位置。我们并不是把回调传递给 foo(..)，而是从 foo(..) 得到某个东西（外观上看是一个真正的Promise），然后把回调传给这个东西。

但是，为什么这就比单纯使用回调更值得信任呢？如何能够确定返回的这个东西实际上就是一个可信任的 Promise 呢？

Promise 对这个问题已经有一个解决方案。包含在原生 ES6 Promise 实现中的解决方案就是 Promise.resolve(..)。

**如果向 Promise.resolve(..) 传递一个非 Promise、非 thenable 的立即值，就会得到一个用这个值填充的 promise。**

**而如果向 Promise.resolve(..) 传递一个真正的 Promise，就只会返回同一个 promise。**

**如果向 Promise.resolve(..) 传递了一个非 Promise 的thenable 值，前者就会试图展开这个值，而且展开过程会持续到提取出一个具体的非类Promise 的最终值。**

假设我们要调用一个工具 foo(..)，且并不确定得到的返回值是否是一个可信任的行为良好的 Promise，但我们可以知道它至少是一个 thenable。Promise.resolve(..) 提供了可信任的 Promise 封装工具，可以链接使用：

```javascript
// 不要只是这么做：
foo( 42 )
.then( function(v){
  console.log( v );
} );
// 而要这么做：
Promise.resolve( foo( 42 ) )
.then( function(v){
  console.log( v );
} ); 
```

#### 建立信任

Promise 这种模式通过可信任的语义**把回调作为参数传递（到then）**，使得这种行为更可靠更合理。通过把回调的控制反转反转回来，我们把控制权放在了一个可信任的系统（Promise）中，这种系统的设计目的就是为了使异步编码更清晰。

### Promise模式

* **Promise.all([..])**

在经典的编程术语中，门（gate）是这样一种机制要等待两个或更多并行 / 并发的任务都完成才能继续。它们的完成顺序并不重要，但是必须都要完成，门才能打开并让流程控制继续。

在 Promise API 中，这种模式被称为 all([ .. ])。

* **Promise.race([..])**

有时候你会想只响应“第一个跨过终点线的 Promise”，而抛弃其他 Promise。

这种模式传统上称为门闩，但在 Promise 中称为竞态。

1. 超时竞赛


我们之前看到过这个例子，其展示了如何使用 Promise.race([ .. ]) 表达 Promise 超时模式：

// foo()是一个支持Promise的函数
// 前面定义的timeoutPromise(..)返回一个promise，
// 这个promise会在指定延时之后拒绝
// 为foo()设定超时
Promise.race( [
 foo(), // 启动foo()
 timeoutPromise( 3000 ) // 给它3秒钟
] )
.then(
 function(){
 // foo(..)按时完成！
 },
 function(err){
 // 要么foo()被拒绝，要么只是没能够按时完成，
 // 因此要查看err了解具体原因
 }
); 

2. finally

一个关键问题是：“那些被丢弃或忽略的 promise 会发生什么呢？”我们并不是从性能的角度提出这个问题的——通常最终它们会被垃圾回收——而是从行为的角度（副作用等）。Promise 不能被取消，也不应该被取消

么如果前面例子中的 foo() 保留了一些要用的资源，但是出现了超时，导致这个 promise被忽略，这又会怎样呢？在这种模式中，会有什么为超时后主动释放这些保留资源提供任何支持，或者取消任何可能产生的副作用吗？如果你想要的只是记录下 foo() 超时这个事实，又会如何呢？

Promise 需要一个 finally(..) 回调注册，这个回调在 Promise 决议后总是会被调用，并且允许你执行任何必要的清理工作。

它看起来可能类似于：

```javascript
var p = Promise.resolve( 42 );

p.then( something )
.finally( cleanup )
.then( another )
.finally( cleanup ); 
```

在各种各样的 Promise 库中，finally(..) 还是会创建并返回一个新的Promise（以支持链接继续）。如果 cleanup(..) 函数要返回一个 Promise 的话，这个 promise 就会被连接到链中，这意味着这里还是会有前面讨论过的未处理拒绝问题。

我们可以构建一个静态辅助工具来支持查看（而不影响）Promise 的决议：

```javascript
// polyfill安全的guard检查
if (!Promise.observe) {
  Promise.observe = function(pr,cb) {
    // 观察pr的决议
    pr.then(
      function fulfilled (msg){
        // 安排异步回调（作为Job）
        Promise.resolve( msg ).then( cb );
      },
      
      function rejected(err){
        // 安排异步回调（作为Job）
        Promise.resolve( err ).then( cb );
      }
    );
    
    // 返回最初的promise
    return pr;
  };
} 
```

下面是如何在前面的超时例子中使用这个工具：

```javascript
Promise.race( [
  Promise.observe(
    foo(), // 试着运行foo()
    function cleanup(msg){
    // 在foo()之后清理，即使它没有在超时之前完成
    }
 ),

 timeoutPromise( 3000 ) // 给它3秒钟
] )
```

这个辅助工具 Promise.observe(..) 只是用来展示可以如何查看 Promise 的完成而不对其产生影响。

* none([ .. ])

这个模式类似于 all([ .. ])，不过完成和拒绝的情况互换了。所有的 Promise 都要被拒绝，即拒绝转化为完成值，反之亦然。

* any([ .. ])

这个模式与 all([ .. ]) 类似，但是会忽略拒绝，所以只需要完成一个而不是全部。

* first([ .. ])
  
这个模式类似于与 any([ .. ]) 的竞争，即只要第一个 Promise 完成，它就会忽略后续的任何拒绝和完成。

* last([ .. ])

这个模式类似于 first([ .. ])，但却是只有最后一个完成胜出。

### Promise的缺陷

#### 单一值

根据定义，Promise 只能有一个完成值或一个拒绝理由。在简单的例子中，这不是什么问题，但是在更复杂的场景中，你可能就会发现这是一种局限了。

一般的建议是构造一个值封装（比如一个对象或数组）来保持这样的多个信息。这个解决方案可以起作用，但要在 Promise 链中的每一步都进行封装和解封，就十分丑陋和笨重了。

有时候你可以把这一点当作提示你可以 / 应该把问题分解为两个或更多 Promise 的信号。设想你有一个工具 foo(..)，它可以异步产生两个值（x 和 y）：

```javascript
function getY(x) {
  return new Promise( function(resolve,reject){
    setTimeout( function(){
      resolve( (3 * x) - 1 );
    }, 100 );
  } );
}

function foo(bar,baz) {
  var x = bar * baz;

  return getY( x )
  .then( function(y){
    // 把两个值封装到容器中
    return [x,y];
  } );
} 

foo( 10, 20 )
.then( function(msgs){
  var x = msgs[0];
  var y = msgs[1];
  console.log( x, y ); // 200 599
} ); 
```

首先，我们重新组织一下 foo(..) 返回的内容，这样就不再需要把 x 和 y 封装到一个数组值中以通过 promise 传输。取而代之的是，我们可以把每个值封装到它自己的 promise：

```javascript
function foo(bar,baz) {
  var x = bar * baz;
  // 返回两个promise
  return [
    Promise.resolve( x ),
    getY( x )
  ];
}

Promise.all(
  foo( 10, 20 )
)
.then( function(msgs){
  var x = msgs[0];
  var y = msgs[1];
  console.log( x, y );
} ); 
```

var x = .. 和 var y = .. 赋值操作仍然是麻烦的开销。我们可以在辅助工具中采用某种函数技巧:

```javascript
function spread(fn) {
  return Function.apply.bind( fn, null );
}

Promise.all(
  foo( 10, 20 )
)
.then(
  spread( function(x,y){
    console.log( x, y ); // 200 599
  } )
) 
```

当然，你可以把这个函数戏法在线化，以避免额外的辅助工具：

```javascript
Promise.all(
  foo( 10, 20 )
)
.then( Function.apply.bind(
  function(x,y){
    console.log( x, y ); // 200 599
 }, 
  null
) ); 
```

但 ES6 给出了一个更好的答案：解构。数组解构赋值形式看起来是
这样的：

```javascript
Promise.all(
  foo( 10, 20 )
)
.then( function(msgs){
  var [x,y] = msgs;
  console.log( x, y ); // 200 599
} );
```

不过最好的是，ES6 提供了数组参数解构形式：
```javascript
Promise.all(
  foo( 10, 20 )
)
.then( function([x,y]){
  console.log( x, y ); // 200 599
} );
```

现在，我们符合了“每个 Promise 一个值”的理念，并且又将重复样板代码量保持在了最小！

#### 单决议

设想这样一个场景：你可能要启动一系列异步步骤以响应某种可能多次发生的激励（就像是事件），比如按钮点击。这样可能不会按照你的期望工作：

```javascript
// click(..)把"click"事件绑定到一个DOM元素
// request(..)是前面定义的支持Promise的Ajax
var p = new Promise( function(resolve,reject){
  click( "#mybtn", resolve );
} );

p.then( function(evt){
  var btnID = evt.currentTarget.id;
  return request( "http://some.url.1/?id=" + btnID );
} )
.then( function(text){
  console.log( text );
} );
```

只有在你的应用只需要响应按钮点击一次的情况下，这种方式才能工作。如果这个按钮被点击了第二次的话，promise p 已经决议，因此第二个 resolve(..) 调用就会被忽略。因此，你可能需要转化这个范例，为每个事件的发生创建一整个新的 Promise 链：

```javascript
click( "#mybtn", function(evt){
  var btnID = evt.currentTarget.id;
  
  request( "http://some.url.1/?id=" + btnID )
  .then( function(text){
    console.log( text );
  } );
} );
```

这种方法可以工作，因为针对这个按钮上的每个 "click" 事件都会启动一整个新的 Promise序列。

由于需要在事件处理函数中定义整个 Promise 链，这很丑陋。除此之外，这个设计在某种程度上破坏了关注点与功能分离（SoC）的思想。

#### 惯性

要在你自己的代码中开始使用 Promise 的话，一个具体的障碍是，现存的所有代码都还不理解 Promise。如果你已经有大量的基于回调的代码，那么保持编码风格不变要简单得多。

Promise 提供了一种不同的范式，因此，编码方式的改变程度从某处的个别差异到某种情况下的截然不同都有可能。你需要刻意的改变，因为 Promise 不会从目前的编码方式中自然而然地衍生出来。

考虑如下的类似基于回调的场景：

```javascript
function foo(x,y,cb) {
  ajax(
    "http://some.url.1/?x=" + x + "&y=" + y,
    cb
  );
}

foo( 11, 31, function(err,text) {
  if (err) {
    console.error( err );
  }
  else {
    console.log( text );
  }
} );
```

能够很快明显看出要把这段基于回调的代码转化为基于 Promise 的代码应该从哪些步骤开始吗？

我们绝对需要一个支持 Promise 而不是基于回调的 Ajax 工具，可以称之为request(..)。你可以实现自己的版本，就像我们所做的一样。但是，如果不得不为每个基于回调的工具手工定义支持 Promise 的封装，这样的开销会让你不太可能选择支持 Promise的重构。

Promise 没有为这个局限性直接提供答案。多数 Promise 库确实提供辅助工具。

```javascript
var request = Promise.wrap( ajax );

request( "http://some.url.1/" )
.then( .. )
.. 
```

Promise.wrap(..) 并不产出 Promise。它产出的是一个将产生 Promise 的函数。在某种意义上，产生 Promise 的函数可以看作是一个 Promise 工厂。我提议将其命名为“promisory”（“Promise”+“factory”）。**把需要回调的函数封装为支持 Promise 的函数，这个动作有时被称为“提升”或“Promise工厂化”。**

## 生成器
255
