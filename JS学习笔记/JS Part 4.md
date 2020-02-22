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

```javascript
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
```

2. finally

一个关键问题是：“那些被丢弃或忽略的 promise 会发生什么呢？”我们并不是从性能的角度提出这个问题的——通常最终它们会被垃圾回收——而是从行为的角度（副作用等）。Promise 不能被取消，也不应该被取消。

那么如果前面例子中的 foo() 保留了一些要用的资源，但是出现了超时，导致这个 promise被忽略，这又会怎样呢？在这种模式中，会有什么为超时后主动释放这些保留资源提供任何支持，或者取消任何可能产生的副作用吗？如果你想要的只是记录下 foo() 超时这个事实，又会如何呢？

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

### 消息的双向传递

```javascript
function *foo(x) {
  var y = x * (yield "Hello"); // <-- yield一个值！
  return y;
}

var it = foo( 6 );

var res = it.next(); // 第一个next()，并不传入任何东西
res.value; // "Hello"

res = it.next( 7 ); // 向等待的yield传入7
res.value; // 42 
```

yield .. 和 next(..) 这一对组合起来，在生成器的执行过程中构成了一个双向消息传递系统。

*我们并没有向第一个 next() 调用发送值，这是有意为之。只有暂停的 yield才能接受这样一个通过 next(..) 传递的值，而在生成器的起始处我们调用第一个 next() 时，还没有暂停的 yield 来接受这样一个值。规范和所有兼容浏览器都会默默丢弃传递给第一个 next() 的任何东西。*

第一个 next() 调用（没有参数的）基本上就是在提出一个问题：“生成器 *foo(..) 要给我的下一个值是什么”。谁来回答这个问题呢？第一个 yield "hello" 表达式。

最后一个it.next(7) 调用再次提出了这样的问题：生成器将要产生的下一个值是什么。但是，再没有 yield 语句来回答这个问题了，是不是？那么谁来回答呢？return 语句回答这个问题！

### 生成器产生值

#### 生成器与迭代器

假定你要产生一系列值，其中每个值都与前面一个有特定的关系。要实现这一点，需要一个有状态的生产者能够记住其生成的最后一个值。

可以实现一个直接使用函数闭包的版本：

```javascript
var gimmeSomething = (function(){
  var nextVal;
 
  return function(){
    if (nextVal === undefined) {
      nextVal = 1;
    }
    else {
      nextVal = (3 * nextVal) +6;
    }
    
    return nextVal;
  };
})();

gimmeSomething(); // 1
gimmeSomething(); // 9
gimmeSomething(); // 33
gimmeSomething(); // 105
```

实际上，这个任务是一个非常通用的设计模式，通常通过迭代器来解决。迭代器是一个定义良好的接口，用于从一个生产者一步步得到一系列值。JavaScript 迭代器的接口就是每次想要从生产者得到下一个值的时候调用 next()。

可以为我们的数字序列生成器实现标准的**迭代器接口**：

```javascript
var something = (function(){
  var nextVal;
  
  return {
    // for..of循环需要
    [Symbol.iterator]: function(){ return this; },

    // 标准迭代器接口方法
    next: function(){
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }
      
      return { done:false, value:nextVal };
    }
  };
})();

something.next().value; // 1
something.next().value; // 9
something.next().value; // 33
something.next().value; // 105 
```

[ .. ] 语法被称为计算属性名，这在对象术语定义中是指，指定一个表达式并用这个表达式的结果作为属性的名称。另外，Symbol.iterator 是 ES6 预定义的特殊Symbol 值之一。

ES6 还新增了一个 for..of 循环，这意味着可以通过原生循环语法自动迭代标准迭代器：

```javascript
for (var v of something) {
   console.log( v );
   // 不要死循环！
   if (v > 500) {
     break;
   }
}
// 1 9 33 105 321 969
```

当然，也可以手工在迭代器上循环，调用 next() 并检查 done:true 条件来确定何时停止循环：
```javascript
for (
  var ret;
  (ret = something.next()) && !ret.done;
) {
  console.log( ret.value );
  // 不要死循环！
  if (ret.value > 500) {
    break;
  }
}
// 1 9 33 105 321 969 
```

除了构造自己的迭代器，许多 JavaScript 的内建数据结构（从 ES6 开始），比如 array，也有默认的迭代器：

```javascript
var a = [1,3,5,7,9];

for (var v of a) {
  console.log( v );
}
// 1 3 5 7 9
```

for..of 循环向 a 请求它的迭代器，并自动使用这个迭代器迭代遍历 a 的值。

#### iterable

>iterable（可迭代），即指一个**包含可以在其值上迭代的迭代器**的**对象**。

从一个 iterable 中提取迭代器的方法是：iterable 必须支持一个函数，其名称是专门的 ES6 符号值 Symbol.iterator。调用这个函数时，它会返回一个迭代器。

前面代码片段中的 a 就是一个 iterable。for..of 循环自动调用它的 Symbol.iterator 函数来构建一个迭代器。

前面的代码中列出了定义的 something，你可能已经注意到了这一行：

    [Symbol.iterator]: function(){ return this; }

这段有点令人疑惑的代码是在将 something 的值（迭代器something 的接口）也构建成为一个 iterable。现在**它既是 iterable，也是迭代器**。然后我们把 something 传给 for..of 循环:

```javascript
for (var v of something) {
 ..
}
```

for..of 循环期望 something 是 iterable，于是它寻找并调用它的 Symbol.iterator 函数。我们将这个函数定义为就是简单的 return this，也就是把自身返回，而 for..of 循环并不知情。

#### 停止生成器

for..of 循环的“异常结束”（也就是“提前终止”），通常由 break、return 或者未捕获异常引起，会向生成器的迭代器发送一个信号使其终止。

严格地说，在循环正常结束之后，for..of 循环也会向迭代器发送这个信号。对于生成器来说，这本质上是没有意义的操作，因为生成器的迭代器需要先完成 for..of 循环才能结束。

尽管 for..of 循环会自动发送这个信号，但你可能会希望向一个迭代器手工发送这个信号。可以通过调用 return(..) 实现这一点。

如果在生成器内有 try..finally 语句，它将总是运行，即使生成器已经外部结束。如果需要清理资源的话（数据库连接等），这一点非常有用：

```javascript
function *something() {
  try {
    var nextVal;
    
    while (true) {
      if (nextVal === undefined) {
        nextVal = 1;
      }
      else {
        nextVal = (3 * nextVal) + 6;
      }
      
      yield nextVal;
    }
  }
  // 清理子句
  finally {
    console.log( "cleaning up!" );
  }
} 
```

之前的例子中，for..of 循环内的 break 会触发 finally 语句。但是，也可以在外部通过return(..) 手工终止生成器的迭代器实例：

```javascript
var it = something();

for (var v of it) {
  console.log( v );
  // 不要死循环！
  if (v > 500) {
    console.log(
    // 完成生成器的迭代器
    it.return( "Hello World" ).value
    );
  // 这里不需要break
  }
}
// 1 9 33 105 321 969
// 清理！
// Hello World 
```

调用 it.return(..) 之后，它会立即终止生成器，这当然会运行 finally 语句。另外，它还会把返回的 value 设置为传入 return(..) 的内容，这也就是 "Hello World" 被传出去的过程。

### 异步迭代生成器

回想一下回调方法：

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

如果想要通过生成器来表达同样的任务流程控制，可以这样实现：

```javascript
function foo(x,y) {
  ajax(
    "http://some.url.1/?x=" + x + "&y=" + y,
    function(err,data){
      if (err) {
        // 向*main()抛出一个错误
        it.throw( err );
      }
      else {
        // 用收到的data恢复*main()
        it.next( data );
      }
     }
  );
}

function *main() {
  try {
    var text = yield foo( 11, 31 ); 
    console.log( text );
  }
  catch (err) {
    console.error( err );
  }
}

var it = main();
// 这里启动！
it.next(); 
```

回头往前看一步，思考一下这意味着什么。我们在生成器内部有了看似完全同步的代码（除了 yield 关键字本身），但隐藏在背后的是，在 foo(..) 内的运行可以完全异步。

这是巨大的改进！对于我们前面陈述的回调无法以顺序同步的、符合我们大脑思考模式的方式表达异步这个问题，这是一个近乎完美的解决方案。

从本质上而言，我们把异步作为实现细节抽象了出去，使得我们可以以同步顺序的形式追踪流程控制：“发出一个 Ajax 请求，等它完成之后打印出响应结果。”并且，当然，我们只在这个流程控制中表达了两个步骤，而这种表达能力是可以无限扩展的，以便我们无论需要多少步骤都可以表达。

#### 同步错误处理

```javascript
try {
  var text = yield foo( 11, 31 );
  console.log( text );
}
catch (err) {
  console.error( err );
}
```

这是如何工作的呢？调用 foo(..) 是异步完成的，难道 try..catch 不是无法捕获异步错误？

我们已经看到 yield 是如何让赋值语句暂停来等待 foo(..) 完成，使得响应完成后可以被赋给 text。精彩的部分在于 yield 暂停也使得生成器能够捕获错误。通过这段前面列出的
代码把错误抛出到生成器中：

```javascript
if (err) {
  // 向*main()抛出一个错误
  it.throw( err );
} 
```

我们可以把错误抛入生成器中，不过如果是从生成器向外抛出错误呢？

```javascript
function *main() {
  var x = yield "Hello World";
  yield x.toLowerCase(); // 引发一个异常！
}

var it = main();

it.next().value; // Hello World
try {
  it.next( 42 );
}
catch (err) {
  console.error( err ); // TypeError
}
```

当然，也可以通过 throw .. 手工抛出一个错误，而不是通过触发异常。

### 生成器 + Promise

回想一下在运行 Ajax 例子中基于 Promise 的实现方法：

```javascript
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

foo( 11, 31 )
.then(
  function(text){
    console.log( text );
  },
  function(err){
    console.error( err );
  }
); 
```

支持 Promise 的 foo(..) 在发出 Ajax 调用之后返回了一个 promise。这暗示我们可以通过 foo(..) 构造一个 promise，然后通过生成器把它 yield 出来，然后迭代器控制代码就可以接收到这个 promise 了。

但迭代器应该对这个 promise 做些什么呢？

它应该侦听这个 promise 的决议（完成或拒绝），然后要么使用完成消息恢复生成器运行，要么向生成器抛出一个带有拒绝原因的错误。

```javascript
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

function *main() {
  try {
    var text = yield foo( 11, 31 );
    console.log( text );
  }
  catch (err) {
    console.error( err );
  }
}

var it = main();

var p = it.next().value;
// 等待promise p决议
p.then(
  function(text){
    it.next( text );
  },
  function(err){
    it.throw( err );
  }
); 
```

有什么方法可以实现重复（即循环）迭代控制，每次会生成一个Promise，等其决议后再继续呢？

#### 支持 Promise 的 Generator Runner

如何在运行 Ajax 的例子中使用 run(..) 和 *main() 呢？

```javascript
function *main() {
  // ..
}
// run(..) 返回一个 promise，一旦生成器完成，这个 promise 就会决议，或收到一个生成器没有处理的未捕获异常。
run( main ); 
```

这种运行 run(..) 的方式，它会自动异步运行你传给它的生成器，直到结束。

ES7：async 与 await?

```javascript
function foo(x,y) {
  return request(
    "http://some.url.1/?x=" + x + "&y=" + y
  );
}

async function main() {
  try {
    var text = await foo( 11, 31 );
    console.log( text );
  } 
  catch (err) {
    console.error( err );
  }
}
main(); 
```

main() 也不再被声明为生成器函数了，它现在是一类新的函数：async 函数。最后，我们不再 yield 出 Promise，而是用 await 等待它决议。**如果你 await 了一个 Promise，async 函数就会自动获知要做什么，它会暂停这个函数（就像生成器一样），直到 Promise 决议。**

#### 生成器中的 Promise 并发

想象这样一个场景：你需要从两个不同的来源获取数据，然后把响应组合在一起以形成第三个请求，最终把最后一条响应打印出来。你的第一直觉可能类似如下：

```javascript
function *foo() {
  var r1 = yield request( "http://some.url.1" );
  var r2 = yield request( "http://some.url.2" );
  var r3 = yield request(
    "http://some.url.3/?v=" + r1 + "," + r2
  );
  console.log( r3 );
}
// 使用前面定义的工具run(..)
run( foo ); 
```

在这段代码中，
它们是依次执行的；直到请求URL"http://some.url.1" 完成后才会通过 Ajax 获 取URL"http://some.url.2"。这两个请求是相互独立的，所以性能更高的方案应该是让它们同时运行。

最简单的方法：

```javascript
function *foo() {
  // 让两个请求"并行"
  var p1 = request( "http://some.url.1" );
  var p2 = request( "http://some.url.2" );
  // 等待两个promise都决议
  var r1 = yield p1;
  var r2 = yield p2;
  var r3 = yield request(
    "http://some.url.3/?v=" + r1 + "," + r2
  );
  console.log( r3 );
}
// 使用前面定义的工具run(..)
run( foo ); 
```

这种流程控制模型如果听起来有点熟悉的话，是因为这基本上和
Promise.all([ .. ]) 工具实现的 gate 模式相同。因此，也可以这样表达这种流程控制：

```javascript
function *foo() {
  // 让两个请求"并行"，并等待两个promise都决议
  var results = yield Promise.all( [
    request( "http://some.url.1" ),
    request( "http://some.url.2" )
  ] );
  var r1 = results[0];
  var r2 = results[1];
  var r3 = yield request(
    "http://some.url.3/?v=" + r1 + "," + r2
  );
  console.log( r3 );
}
// 使用前面定义的工具run(..)
run( foo ); 
```

使用生成器实现异步的方法的全部要点在于创建简单、顺序、看似同步的代码，将异步的细节尽可能隐藏起来。

这可能是一个更简洁的方案：

```javascript
// 注：普通函数，不是生成器
function bar(url1,url2) {
  return Promise.all( [
    request( url1 ),
    request( url2 )
  ] );
}

function *foo() {
  // 隐藏bar(..)内部基于Promise的并发细节
  var results = yield bar(
    "http://some.url.1",
    "http://some.url.2"
  );
  var r1 = results[0];
  var r2 = results[1];
  var r3 = yield request(
   "http://some.url.3/?v=" + r1 + "," + r2
  ); 
  console.log( r3 );
}
// 使用前面定义的工具run(..)
run( foo ); 
```

### 生成器委托

yield 委托的主要目的是代码组织，以达到与普通函数调用的对称。

想像一下有两个模块分别提供了方法 foo() 和 bar()，其中 bar() 调用了 foo()。一般来说，把两者分开实现的原因是该程序的适当的代码组织要求它们位于不同的函数中。比如，可能有些情况下是单独调用 foo()，另外一些地方则由 bar() 调用 foo()。

同样是出于这些原因，保持生成器分离有助于程序的可读性、可维护性和可调试性。在这一方面，yield * 是一个语法上的缩写，用于代替手工在 \*foo() 的步骤上迭代，不过是在*bar() 内部。

#### 消息委托

```javascript
function *foo() {
  console.log( "inside *foo():", yield "B" );
  console.log( "inside *foo():", yield "C" );
  return "D";
}

function *bar() { 
  console.log( "inside *bar():", yield "A" );
  // yield委托！
  console.log( "inside *bar():", yield *foo() );
  console.log( "inside *bar():", yield "E" );
  return "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A
console.log( "outside:", it.next( 1 ).value );
// inside *bar(): 1
// outside: B
console.log( "outside:", it.next( 2 ).value );
// inside *foo(): 2
// outside: C
console.log( "outside:", it.next( 3 ).value );
// inside *foo(): 3
// inside *bar(): D
// outside: E
console.log( "ooutside:", it.next( 4 ).value );
// inside *bar(): 4
// outside: F 
```

实际上，yield 委托甚至并不要求必须转到另一个生成器，它可以转到一个非生成器的一般 iterable。比如：

```javascript
function *bar() {
  console.log( "inside *bar():", yield "A" ); 
  // yield委托给非生成器！
  console.log( "inside *bar():", yield *[ "B", "C", "D" ] );
  console.log( "inside *bar():", yield "E" );
  return "F";
} 
```

和 yield 委托透明地双向传递消息的方式一样，错误和异常也是双向传递的：

```javascript
function *foo() {
  try {
    yield "B";
  }
  catch (err) {
    console.log( "error caught inside *foo():", err );
  }
  
  yield "C";
  
  throw "D";
} 

function *bar() {
  yield "A";
  try {
    yield *foo();
  }
  catch (err) {
    console.log( "error caught inside *bar():", err );
  }
  
  yield "E";
  
  yield *baz();
  // 注：不会到达这里！
  yield "G";
}

function *baz() {
  throw "F";
}

var it = bar();

console.log( "outside:", it.next().value );
// outside: A
console.log( "outside:", it.next( 1 ).value );
// outside: B
console.log( "outside:", it.throw( 2 ).value );
// error caught inside *foo(): 2
// outside: C
console.log( "outside:", it.next( 3 ).value );
// error caught inside *bar(): D
// outside: E
try {
  console.log( "outside:", it.next( 4 ).value );
}
catch (err) {
  console.log( "error caught outside:", err );
}
// error caught outside: F 
```

#### 异步委托

```javascript
function *foo() {
  var r2 = yield request( "http://some.url.2" );
  var r3 = yield request( "http://some.url.3/?v=" + r2 );
  return r3;
}

function *bar() {
  var r1 = yield request( "http://some.url.1" );
  var r3 = yield *foo();
  console.log( r3 );
}

run( bar ); 
```

#### 递归委托

```javascript
function *foo(val) {
  if (val > 1) {
    // 生成器递归
    val = yield *foo( val - 1 );
  }
  return yield request( "http://some.url/?v=" + val );
}

function *bar() {
  var r1 = yield *foo( 3 );
  console.log( r1 ); 
}

run( bar ); 
```

### 生成器并发

给出这样一个场景：其中两个不同并发 Ajax 响应处理函数需要彼此协调，以确保数据交流不会出现竞态条件。我们把响应插入到 res 数组中，就像这样：

```javascript
function response(data) {
  if (data.url == "http://some.url.1") {
    res[0] = data;
  }
  else if (data.url == "http://some.url.2") {
    res[1] = data;
  }
} 
```

但是这种场景下如何使用多个并发生成器呢？

```javascript
// request(..)是一个支持Promise的Ajax工具

var res = [];
function *reqData(url) {
  res.push(
  yield request( url )
  );
} 
```

这里不需要手工为 res[0] 和 res[1] 赋值排序，而是使用合作式的排序，使得 res.push(..) 把值按照预期以可预测的顺序正确安置。这样，表达的逻辑给人感觉应该更清晰一点。

但是，实践中我们如何安排这些交互呢？首先，使用 Promise 手工实现：

```javascript
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it1.next();
var p2 = it2.next();

p1
.then( function(data){
  it1.next( data );
  return p2; 
 } )
.then( function(data){
  it2.next( data );
} ); 
```

坦白地说，这种方式的手工程度非常高，并且它也不能真正地让生成器自己来协调，而那才是真正的威力所在。让我们换一种方法试试：

```javascript
// request(..)是一个支持Promise的Ajax工具
var res = [];

function *reqData(url) {
  var data = yield request( url );
  // 控制转移
  yield;
  
  res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );

var p1 = it.next();
var p2 = it.next();

p1.then( function(data){
 it1.next( data );
} );

p2.then( function(data){
 it2.next( data );
} ); 

Promise.all( [p1,p2] )
.then( function(){
  it1.next();
  it2.next();
} ); 
```

在前面的代码中，第二个实例直到第一个实例完全结束才得到数据。但在这里，两个实例都是各自的响应一回来就取得了数据，然后每个实例再次 yield，用于控制传递的目的。然后我们在 Promise.all([ .. ]) 处理函数中选择它们的恢复顺序。

### 形实转换程序

>在通用计算机科学领域，有一个早期的前 JavaScript 概念，称为形实转换程序（thunk）。JavaScript 中的 thunk 是指一个用于调用另外一个函数的函数，没有任何参数。

换句话说，你用一个函数定义封装函数调用，包括需要的任何参数，来定义这个调用的执行，那么这个封装函数就是一个形实转换程序。之后在执行这个 thunk 时，最终就是调用了原始的函数。

```javascript
function foo(x,y) {
  return x + y;
}
function fooThunk() {
  return foo( 3, 4 );
}
// 将来
console.log( fooThunk() ); // 7 
```

所以，同步的 thunk 是非常简单的。但如果是异步的 thunk 呢？我们可以把这个狭窄的thunk 定义扩展到包含让它接收一个回调。

```javascript
function foo(x,y,cb) {
  setTimeout( function(){
    cb( x + y );
  }, 1000 );
}

function fooThunk(cb) {
  foo( 3, 4, cb );
}
// 将来
fooThunk( function(sum){
  console.log( sum ); // 7
} ); 
```

fooThunk(..) 只需要一个参数 cb(..)，因为它已经有预先指定的值 3 和 4（分别作为 x 和 y）可以传给 foo(..)。thunk 就耐心地等待它完成工作所需的最后一部分：那个回调。

我们发明一个工具来做这部分封装工作:

```javascript
function thunkify(fn) {
  var args = [].slice.call( arguments, 1 );
  return function(cb) {
    args.push( cb );
    return fn.apply( null, args );
 };
}

var fooThunk = thunkify( foo, 3, 4 );
// 将来
fooThunk( function(sum) {
  console.log( sum ); // 7
} ); 
```

前面 thunkify(..) 的实现接收 foo(..) 函数引用以及它需要的任意参数，并返回 thunk 本身（fooThunk(..)）。

但是，这并不是 JavaScript 中使用 thunk 的典型方案。典型的方法——如果不令人迷惑的话——并不是 thunkify(..) 构造 thunk 本身，而是thunkify(..) 工具产生一个生成 thunk 的函数。

```javascript
function thunkify(fn) {
  return function() {
    var args = [].slice.call( arguments );
    return function(cb) {
      args.push( cb );
      return fn.apply( null, args );
    };
  };
} 
```

此处主要的区别在于多出来的 return function() { .. } 这一层。以下是用法上的区别：

```javascript
var whatIsThis = thunkify( foo );
var fooThunk = whatIsThis( 3, 4 ); 
// 将来
fooThunk( function(sum) {
  console.log( sum ); // 7
} ); 
```

这段代码暗藏的一个大问题是：whatIsThis 调用的是什么。并不是这个 thunk，而是某个从 foo(..) 调用产生 thunk 的东西。这有点类似于 thunk 的“工厂”。可以说thunkify(..) 生成一个 thunkory，然后 thunkory 生成 thunk。

```javascript
var fooThunkory = thunkify( foo );

var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );
// 将来
fooThunk1( function(sum) {
  console.log( sum ); // 7
} );
fooThunk2( function(sum) {
  console.log( sum ); // 11
} ); 
```

那么所有这些关于 thunk 的内容与生成器有什么关系呢？

可以把 thunk 和 promise 大体上对比一下：它们的特性并不相同，所以并不能直接互换。Promise 要比裸 thunk 功能更强、更值得信任。但从另外一个角度来说，它们都可以被看作是对一个值的请求，回答可能是异步的。

前面我们定义了一个工具用于 promise 化一个函数，我们称之为
Promise.wrap(..)，也可以将其称为 promisify(..) ！这个 Promise 封装工具并不产生Promise，它生成的是 promisory，而 promisory 则接着产生 Promise。这和现在讨论的thunkory 和 thunk 是完全对称的。

为了说明这种对称性，我们要首先把前面的 foo(..) 例子修改一下，改成使用 error-first 风格的回调：

```javascript
function foo(x,y,cb) {
  setTimeout( function(){
    // 假定cb(..)是error-first风格的
    cb( null, x + y );
  }, 1000 );
} 
```

现在我们对比一下 thunkify(..) 和 promisify(..)（即第 3 章中的 Promise.wrap(..)）的使用：

```javascript
// 对称：构造问题提问者
var fooThunkory = thunkify( foo );
var fooPromisory = promisify( foo );
// 对称：提问
var fooThunk = fooThunkory( 3, 4 );
var fooPromise = fooPromisory( 3, 4 );
// 得到答案
fooThunk( function(err,sum){
  if (err) {
    console.error( err );
  }
  else {
    console.log( sum ); // 7 
  }
} );
// 得到promise答案
fooPromise
.then(
  function(sum){
    console.log( sum ); // 7
  },
  function(err){
    console.error( err );
 }
); 
```

hunkory 和 promisory 本质上都是在提出一个请求（要求一个值），分别由 thunk fooThunk和 promise fooPromise 表示对这个请求的未来的答复。

了解了这个视角之后，就可以看出，yield 出 Promise 以获得异步性的生成器，也可以为异步性而 yield thunk。我们所需要的只是一个更智能的 run(..) 工具（就像前面的一样），不但能够寻找和链接 yield 出来的 Promise，还能够向 yield 出来的 thunk 提供回调。

于是，request(..) 可能是以下两者之一：

```javascript
// promisory request(..) 
var request = Promise.wrap( ajax );
// vs.
// thunkory request(..)
var request = thunkify( ajax ); 
```

从更大的角度来说，thunk 本身基本上没有任何可信任性和可组合性保证，而这些是Promise 的设计目标所在。单独使用 thunk 作为 Pormise 的替代在这个特定的生成器异步模式里是可行的，但是与 Promise 具备的优势相比，这应该并不是一种理想方案。