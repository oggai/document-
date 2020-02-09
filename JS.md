# you don't know JS 学习笔记

## 第一部分 作用域和闭包

### 作用域

>作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。

引擎对变量的查找有两种方式：LHS 和 RHS，如果查找的目的是对变量进行赋值，那么就会使用 LHS 查询；如果目的是获取变量的值，就会使用 RHS 查询。规则是一级一级向上级作用域查找。

两种异常：

ReferenceError 同作用域判别失败相关，不成功的 RHS 引用会导致抛出 ReferenceError 异常。不成功的 LHS 引用会导致自动隐式地创建一个全局变量（非严格模式下），该变量使用 LHS 引用的目标作为标识符，或者抛出 ReferenceError 异常（严格模式下）。

而 TypeError 则代表作用域判别成功了，但是对结果的操作是非法或不合理的。

### 词法作用域

>词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会**保持作用域不变(即静态)**（大部分情况下是这样的）。

*作用域查找会在找到第一个匹配的标识符时停止。内部的标识符“遮蔽”了外部的标识符。*

*全局变量会自动成为全局对象（比如浏览器中的 window 对象）的属性，因此有时需要访问全局变量，则可以不直接通过全局对象的词法名称，而是间接地通过对全局对象属性的引用来对其进行访问。如``window.a``*

- **eval() & with() “欺骗”词法作用域**

```JavaScript
function foo(str, a) {
  eval( str );   // 欺骗！
  console.log( a, b ); 
}
var b = 2;
foo( "var b = 3;", 1 ); // 1, 3
```

``eval(..)`` 调用中的 ``var b = 3;`` ,这段代码会被当作本来就在那里一样来处理，此时b在函数内部声明，遮蔽了外部作用域的b。

``eval(..)`` 通常被用来执行动态创建的代码，``setTimeout(..)`` 、``setInterval(..)`` 和``new Function(..)`` 函数同样可以将字符串当成参数，字符串会被当成**动态执行**的代码。

```JavaScript
function foo(obj) {
  with (obj) {
  a = 2;
  }  
}
var o1 = {
  a: 3
};
var o2 = {
  b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2——不好，a 被泄漏到全局作用域上了！
```

with 声明实际上是根据你传递给它的**对象**凭空创建了一个全新的词法作用域，将对象的属性当作作用域中的标识符来处理。可以这样理解，当我们传递 o1 给 with 时，with 所声明的作用域是 o1，而这个作用域中含有一个同 o1.a 属性相符的标识符。但当我们将 o2 作为作用域时，其中并没有 a 标识符，因此进行了正常的 LHS 标识符查找,o2 的作用域、foo(..) 的作用域和全局作用域中都没有找到标识符 a，因此当 a＝2 执行时，自动创建了一个全局变量（因为是非严格模式）。

尽管 with 块可以将一个对象处理为词法作用域，但是这个块内部正常的 var声明并不会被限制在这个块的作用域中，而是被添加到 with 所处的函数作用域中。

### 函数作用域和块作用域

- 函数作用域

>最小暴露原则：这个原则是指在软件设计中，应该最小限度地暴露必要内容，而将其他内容都“隐藏”起来，比如**某个模块或对象的 API 设计**。

```JavaScript
function foo() { // 函数声明
  var a = 3;
  console.log( a ); // 3
}
foo();
```

```JavaScript
(function foo(){ // 函数表达式
  var a = 3;
  console.log( a ); // 3
})();
```

函数声明和函数表达式之间最重要的区别是它们的名称标识符将会绑定在何处。比较一下前面两个代码片段。第一个片段中 foo 被绑定在所在作用域中，可以直接通过foo() 来调用它。第二个片段中 foo 被绑定在函数表达式自身的函数中而不是所在作用域中。foo 变量名被隐藏在自身中意味着不会非必要地污染外部作用域。

- LIFE函数

>IIFE，代表立即执行函数表达式（Immediately Invoked Function Expression）。

```JavaScript
(function IIFE() {
    var a = 3;
    console.log( a ); // 3
})();
```

```JavaScript
var a = 2;

(function IIFE( global ) { // 良好的代码风格
  var a = 3;
  console.log( a ); // 3
  console.log( global.a ); // 2
})( window ); //进阶用法：传递参数
```

```JavaScript
var a = 2;

(function IIFE( def ) {
  def( window );
})(function def( global ) {  // 将函数当作参数传递进去
  var a = 3;
  console.log( a ); // 3
  console.log( global.a ); // 2
});
```

- 块作用域

>块作用域是一个用来对之前的最小授权原则进行扩展的工具，将代码从在函数中隐藏信息扩展为**在块中隐藏信息**。

with 从对象中创建出的作用域仅在 with 声明中而非外部作用域中有效。

try/catch 的 catch 分句会创建一个块作用域，其中声明的变量仅在 catch 内部有效。

let 关键字可以将变量绑定到所在的任意作用域中（通常是 { .. } 内部）。换句话说，let为其声明的变量隐式地绑定在了所在的块作用域。也可直接用{  }来进行显示绑定。*let 声明附属于一个新的作用域而不是当前的函数作用域（也不属于全局作用域）。*

```JavaScript
let (a = 2) {  //let的显式声明，但let 声明并不包含在 ES6 中。
  console.log( a ); // 2
}
console.log( a ); // ReferenceError
```

const同样可以用来创建块作用域变量，但其值是固定的（常量）。之后任何试图修改值的操作都会引起错误。

### 提升

>JavaScript 实际上会将``var a = 2;``看成两个声明：``var a;`` 和 ``a = 2;``。第一个定义声明是在编译阶段进行的。第二个赋值声明会被留在原地等待执行阶段。变量和函数声明从它们在代码中出现的位置被“移动”到了最上面。这个过程就叫作提升。

*只有声明本身会被提升，而赋值或其他运行逻辑会留在原地。*

**函数声明会首先被提升，然后才是变量。**

```JavaScript
foo(); // 不是 ReferenceError, 而是 TypeError!

var foo = function bar() {
// ...
};
```

```JavaScript
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
// ...
};
这个代码片段经过提升后，实际上会被理解为以下形式：
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {  // 可见函数bar的作用域其实是其函数内部
var bar = ...self...
// ...
}
```

### 作用域闭包

>当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

例如

```JavaScript
function foo() {
  var a = 2;
  
  function bar() {
    console.log( a );
  }
  
  return bar;
}

var baz = foo();

baz(); // 2 —— 朋友，这就是闭包的效果。
```

在``foo()``执行后，通常会期待``foo()``的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不再使用的内存空间。由于看上去``foo()``的内容不会再被使用，所以很自然地会考虑对其进行回收。

而闭包的“神奇”之处正是可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回收。谁在使用这个内部作用域？原来是``bar()``本身在使用。

``bar()``依然持有对该作用域的引用，而这个引用就叫作闭包。

再看几个例子：

```JavaScript
function foo() {
  var a = 2;
  
  function baz() {
    console.log( a ); // 2
  }
  
  bar( baz );
}

function bar(fn) {
  fn(); // 妈妈快看呀，这就是闭包！
}
```

```JavaScript
var fn;

function foo() {
  var a = 2;
  
  function baz() {
    console.log( a );
  }
  
  fn = baz; // 将 baz 分配给全局变量
}

function bar() {
  fn(); // 妈妈快看呀，这就是闭包！
}

foo();
bar(); // 2
```

实际的例子：

```JavaScript
function wait(message) {
  
  setTimeout( function timer() {
    console.log( message );
  }, 1000 );
}

wait( "Hello, closure!" );
```

``wait(..)``执行 1000 毫秒后，它的内部作用域并不会消失，``timer`` 函数依然保有``wait(..)``作用域的闭包。

```JavaScript
function setupBot(name, selector) {
  $( selector ).click( function activator() {
    console.log( "Activating: " + name );
  } );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

``setupBot(..)``执行之后并不会被回收，``actibator()``仍然保有其作用域的闭包，当``click``事件发生后，``actibator()``会执行。

如果将函数（访问它们各自的词法作用域）当作第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。**在定时器、事件监听器、Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包！**

再谈LIFE：

```javascript
var a = 2;

(function IIFE() {
  
  console.log( a );

})();
```

虽然这段代码可以正常工作，但严格来讲它并不是闭包。为什么？因为函数（示例代码中的 ``IIFE``）并不是在它本身的词法作用域以外执行的。它在定义时所在的作用域中执行（而外部作用域，也就是全局作用域也持有 a）。a 是通过普通的词法作用域查找而非闭包被发现的。

尽管 IIFE 本身并不是观察闭包的恰当例子，但它的确创建了闭包，并且也是最常用来创建可以被封闭起来的闭包的工具。

- **循环与回调**

```javascript
for (var i=1; i<=5; i++) {
  setTimeout( function timer() {
    console.log( i );
  }, i*1000 );
}
```

*注意：延迟函数的回调会在循环结束时才执行。*

这段代码在运行时会以每秒一次的频率输出五次 6。尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个 i，所有函数共享一个 i 的引用。

**解决问题：**

```javascript
for (var i=1; i<=5; i++) {
  (function(j) {
    setTimeout( function timer() {
      console.log( j );
    }, j*1000 );
  })( i );
}
```

在迭代内使用 IIFE 会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。

**更为厉害的是：**

```javascript
for (var i=1; i<=5; i++) {
  let j = i; // 是的，闭包的块作用域！
  
  setTimeout( function timer() {
    console.log( j );
  }, j*1000 );
}
```

for 循环头部的 let 声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

**改进后：**

```javascript
for (let i=1; i<=5; i++) {
  setTimeout( function timer() {
    console.log( i );
  }, i*1000 );
}
```

- **模块**

>模块模式需要具备两个必要条件。
>
>1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
>2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

```javascript
function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];
  
  function doSomething() {
    console.log( something );
  }
  
  function doAnother() {
    console.log( another.join( " ! " ) );
  }
  
  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
}

var foo = CoolModule(); //可以调用多次的模块创建器

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

``CoolModule()``返回一个用对象字面量语法``{ key: value, ... }``来表示的对象。可以将这个对象类型的返回值看作本质上是模块的公共 API。

```javascript
var foo = (function CoolModule() {
  var something = "cool";
  var another = [1, 2, 3];
  
  function doSomething() {
    console.log( something );
  }
  
  function doAnother() {
    console.log( another.join( " ! " ) );
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  };
})();  //单例模块

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

未来的模块机制：

```javascript
export hello;  

import hello from "bar";

module foo from "foo";
```

``import`` 可以将一个模块中的**一个或多个 API** 导入到当前作用域中，并分别绑定在一个变量上（在我们的例子里是 hello）。

``module``会将**整个模块的 API** 导入并绑定到一个变量上（在我们的例子里是 foo 和 bar）。

``export``会将当前模块中的一个标识符（变量、函数）导出为公共 API。
