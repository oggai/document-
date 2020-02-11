# you don't know JS 学习笔记

## 第二部分 `this`和对象原型

### `this`

> **this既不指向函数自身也不指向函数的词法作用域.**  
>
> this实际上是在函数被调用时发生的绑定，它指向什么完全取决于**函数在哪里被调用。**

调用栈和调用位置
>调用栈就是为了到达当前执行位置所调用的所有函数。  
>我们关心的调用位置就在当前正在执行的函数的前一个调用中。

this的绑定规则  

* 默认绑定  

  在代码中，`fuction()`是直接使用不带任何修饰的函数引用进行调用的， 因此只能使用默认绑定，无法应用其他规则。

  ```javascript
  function foo() {
    "use strict";  // 函数体处于严格模式！

    console.log( this.a ); 
  }
  
  var a = 2;
  foo(); // TypeError: this is undefined
  ```
  
  ```javascript
  function foo() {
    console.log( this.a );
  }
  var a = 2;

  (function(){
    "use strict";

    foo(); // 2
  })();
  ```

  对于默认绑定来说，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是**函数体是否处于严格模式**。如果函数体处于严格模式，this 会被绑定到undefined，否则this 会被绑定到全局对象。

* 隐式绑定

  ```javascript
  function foo() {
    console.log( this.a );
  }
  var obj = {
    a: 2,

    foo: foo
  };
  
  obj.foo(); // 2
  ```

  当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 **`this` 绑定到这个上下文对象**。因为调用 `foo()` 时 `this` 被绑定到 `obj`，因此 `this.a` 和 `obj.a` 是一样的。

  这里有一个绑定丢失的问题，但函数被隐式赋值时（引用的其实时函数本身。相当于直接调用了`fuction()`），thisd的隐式绑定会丢失，this会被绑定到全局对象或undefined上，取决于是否是严格模式。例如：  

  ```javascript
  var bar = obj.foo;  
  
  doFoo( obj.foo );  
  
  setTimeout( obj.foo, 100 );
  ```

* 显示绑定: call()、apply()

  ```javascript
  function foo() {
    console.log( this.a );
  }
  
  var obj = {
    a:2
  };
  
  foo.call( obj ); // 2
  ```
  
  通过 `foo.call(..)`，我们可以在调用 `foo` 时强制把它的 `this` 绑定到 `obj` 上。  
  
  *如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作 `this` 的绑定对象，这个原始值会被转换成它的对象形式（也就是 ``new` String(..)`、``new` Boolean(..)` 或者``new` Number(..)`）。这通常被称为“装箱”。*

  可惜，显式绑定仍然无法解决我们之前提出的丢失绑定(隐式赋值)问题。

  一个解决办法是**硬绑定:**  

  包裹函数：

   ```javascript

  function foo() {
    console.log( this.a );
  };

  var obj = {
    a:2
  };

  var bar = function() {
    foo.call( obj );  // 将显示绑定放入函数内，
                      // 无论之后如何调用函数bar，它总会手动在 obj 上
                      // 调用 foo
  };

  bar(); // 2
  // 不管是隐式赋值，还是显示绑定
  setTimeout( bar, 100 ); // 2
  // 硬绑定的 bar 不可能再修改它的 this
  bar.call( window ); // 2
  ```

  内置方法`bind()`:

  ```javascript
  function foo(something) {
    console.log( this.a, something );
    return this.a + something;
  }
  
  var obj = {
    a:2
  };
  
  var bar = foo.bind( obj );  // bind(..) 会返回一个硬编码的新函数，
                              // 它会把参数设置为 this 的上下文并调用原始函数。
  var b = bar( 3 ); // 2 3
  
  console.log( b ); // 5
  ```

  *`bind(..)` 是一种 `polyfill` 代码（`polyfill` 就是我们常说的刮墙用的腻子，`polyfill` 代码主要用于旧浏览器的兼容，比如说在旧的浏览器中并没有内置 `bind` 函数，因此可以使用 `polyfill` 代码在旧浏览器中实现新的功能）*

  另外一种解决办法是**第三方API**：

  ```javascript
  function foo(el) {
    console.log( el, this.id );
  }
  
  var obj = {
    id: "awesome"
  };
  // 调用 foo(..) 时把 this 绑定到 obj
  [1, 2, 3].forEach( foo, obj );
  // 1 awesome 2 awesome 3 awesome
  ```

* `new`绑定

  构造函数：所有函数都可以用 `new` 来调用，这种函数调用被称为构造函数调用。这里有一个重要但是非常细微的区别：实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。  
  使用 `new` 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

  1. 创建（或者说构造）一个全新的对象。
  2. 这个新对象会被执行 [[ 原型 ]] 连接。
  3. 这个新对象会绑定到函数调用的 `this`。
  4. 如果函数没有返回其他对象，那么 `new` 表达式中的函数调用会自动返回这个新对象。

  ```javascript
  function foo(a) {
    this.a = a;
  }

  var bar = new foo(2);

  console.log( bar.a ); // 2
  ```

  使用 `new` 来调用 `foo(..)` 时，我们会构造一个新对象`bar`并把它绑定到 `foo(..)` 调用中的`this`上。

**绑定的优先级**  
这里只讨论new和显示绑定的优先级：

```javascript
function foo(something) {
  this.a = something;
}

var obj1 = {};
var bar = foo.bind( obj1 );

bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar(3);
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

出乎意料！ **`bar` 被硬绑定到 `obj1` 上，但是 `new bar(3)` 并没有像我们预计的那样把`obj1.a`修改为 3**。相反，**`new` 修改了硬绑定（到 `obj1` 的）调用 `bar(..)` 中的 `this`**。因为使用了`new` 绑定，我们得到了一个名字为 `baz` 的新对象，并且 `baz.a` 的值是 3。

`bind`函数会判断硬绑定函数是否是被 `new` 调用，如果是的话就会使用函数内新创建的 `this` 替换硬绑定的 `this`。

那么，为什么要在 new 中使用硬绑定函数呢？  
之所以要在 new 中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用new 进行初始化时就可以只传入其余的参数。bind(..) 的功能之一就是可以**把除了第一个参数（第一个参数用于绑定 this）之外的其他参数都传给下层的函数**（这种技术称为“部分应用”，是“**柯里化**”的一种）。举例来说：

```javascript
function foo(p1,p2) {
  this.val = p1 + p2;
}
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么
// 反正使用 new 时 this 会被修改
var bar = foo.bind( null, "p1" );
var baz = new bar( "p2" );

baz.val; // p1p2
```

因此this的判定规则是：

1. 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象。
`var bar = new foo()`
2. 函数是否通过 call、apply（显式绑定）或者硬绑定调用？如果是的话，this 绑定的是指定的对象。  
`var bar = foo.call(obj2)`
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上下文对象。  
`var bar = obj1.foo()`
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。  
`var bar = foo()`

绑定规则的例外是：

* null

  如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或  者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。  
  那么什么情况下你会传入 null 呢？  
  一种非常常见的做法是使用 apply(..) 来“展开”一个数组，并当作参数传入一个  函数。类似地，bind(..) 可以对参数进行柯里化（预先设置一些参数），这种方  法有时非常有用：
  
  ```javascript
  function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
  }
  // 把数组“展开”成参数
  foo.apply( null, [2, 3] ); // a:2, b:3
  // 使用 bind(..) 进行柯里化
  var bar = foo.bind( null, 2 );
  bar( 3 ); // a:2, b:3
  ```
  
  解决方法是：
  
  传入一个特殊的对象，把 this 绑定到这个对象不会对你的程序产生任何副作用。  我们可以创建一个“DMZ”（demilitarizedzone，非军事区）对象——它就是一个空  的非委托的对象。如果我们在忽略 this 绑定时总是传入一个 DMZ 对象，那就什  么都不用担心了，因为任何对于 this 的使用都会被限制在这个空对象中，不会对  全局对象产生任何影响。  
  在 JavaScript 中创建一个空对象最简单的方法都是 Object.create(null)。  Object.create(null) 和 {} 很像，但是并不会创建Object.prototype 这个委  托，所以它比 {}“更空”。
  
  ```javascript
  function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
  }
  // 我们的 DMZ 空对象
  var ø = Object.create( null );
  // 把数组展开成参数
  foo.apply( ø, [2, 3] ); // a:2, b:3
  // 使用 bind(..) 进行柯里化
  var bar = foo.bind( ø, 2 );
  bar( 3 ); // a:2, b:3
  ```

* 间接引用（隐式赋值）

* 软绑定`softBind()`
  
  ```javascript
  function foo() {
    console.log("name: " + this.name);
  }
  var obj = { name: "obj" },
      obj2 = { name: "obj2" },
      obj3 = { name: "obj3" };
  
  var fooOBJ = foo.softBind( obj );
  
  fooOBJ(); // name: obj
  
  obj2.foo = foo.softBind(obj);
  obj2.foo(); // name: obj2 <---- 看！！！
  
  fooOBJ.call( obj3 ); // name: obj3 <---- 看！
  setTimeout( obj2.foo, 10 );
  // name: obj <---- 应用了软绑定
  ```

  软绑定解决了用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this的问题。  
  softBind(..) 的其他原理和 ES5 内置的 bind(..) 类似。它会对指定的函数进行封装，首先检查调用时的 this，如果 this 绑定到全局对象或者 undefined，那就把指定的默认对象 obj 绑定到 this，否则不会修改 this。此外，这段代码还支持可选的柯里化（详情请查看之前和 bind(..) 相关的介绍）。

箭头函数 `=>`：

```javascript
function foo() {
  // 返回一个箭头函数
  return (a) => {
  //this 继承自 foo()
    console.log( this.a );
  };
}

var obj1 = {
  a:2
};

var obj2 = {
  a:3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是 3 ！
```

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。由于 foo() 的this 绑定到 obj1，bar（引用箭头函数）的 this 也会绑定到 obj1，箭头函数的绑定无法被修改。（new 也不行！）

与箭头函数类似的`self = this`：

```javascript
function foo() {
  var self = this; // lexical capture of this
  
  setTimeout( function(){
    console.log( self.a );
  }, 100 );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

**ES6 中的箭头函数并不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this，具体来说，箭头函数会继承外层函数调用的 this 绑定（无论 this 绑定到什么）。这其实和 ES6 之前代码中的 self = this 机制一样。**

### 对象
