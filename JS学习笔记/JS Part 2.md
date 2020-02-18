# you don't know JS 学习笔记

- [you don't know JS 学习笔记](#you-dont-know-js-%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0)
  - [第二部分 `this`和对象原型](#%e7%ac%ac%e4%ba%8c%e9%83%a8%e5%88%86-this%e5%92%8c%e5%af%b9%e8%b1%a1%e5%8e%9f%e5%9e%8b)
    - [`this`](#this)
    - [对象](#%e5%af%b9%e8%b1%a1)
    - [混合对象“类”](#%e6%b7%b7%e5%90%88%e5%af%b9%e8%b1%a1%e7%b1%bb)
    - [原型](#%e5%8e%9f%e5%9e%8b)
    - [行为委托](#%e8%a1%8c%e4%b8%ba%e5%a7%94%e6%89%98)

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

  这里有一个绑定丢失的问题，当函数被隐式赋值时（引用的其实是函数本身。相当于直接调用了`fuction()`），this的隐式绑定会丢失，this会被绑定到全局对象或undefined上，取决于是否是严格模式。例如：  

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
  
  *如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作 `this` 的绑定对象，这个原始值会被转换成它的对象形式（也就是 `new String(..)` new`Boolean(..)` 或者`new Number(..)`）。这通常被称为“装箱”。*

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
                      // 无论之后如何调用函数bar，它总会手动在 obj 上调用 foo
                      
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
  一种非常常见的做法是使用 apply(..) 来“展开”一个数组，并当作参数传入一个函数。类似地，bind(..) 可以对参数进行柯里化（预先设置一些参数），这种方法有时非常有用：
  
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

两种定义形式：文字形式和构造形式。

```javascript
var myObj = {  // 文字形式
  key: value
// ...
};
```

```javascript
var myObj = new Object();  // 构造形式
myObj.key = value;  // 只能逐个添加属性
```

对象的简单基本类型：
* string
* number
* boolean
* null
* undefined
* object

对象的复杂基本类型（子类型）：
* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error

*在 JavaScript 中，它们实际上只是一些内置函数（首字母大写）。这些内置函数可以当作构造函数（由 new 产生的函数调用）来使用，从而可以构造一个对应子类型的新对象。*  
*在必要时语言会自动把字符串字面量转换成一个 String 对象，也就是说你并不需要显式创建一个对象。数字同理。*

对象的内容：
>对象的内容是由一些存储在特定命名位置的（任意类型的）值组成的，我们称之为属性。  
>在引擎内部，这些值的存储方式是多种多样的，一般并不会存在对象容器内部。存储在对象容器内部的是这些属性的名称，它们就像指针（从技术角度来说就是引用）一样，指向这些值真正的存储位置。

对象的访问

对象的访问用 . 操作符或者 [] 操作符。.a 语法通常被称为“属性访问”，["a"] 语法通常被称为“键访问”。[".."] 语法可以接受任意 UTF-8/Unicode 字符串作为属性名。

属性与方法的区别

*在其他语言中，属于对象（也被称为“类”）的函数通常被称为“方法”。*  

在JavaScript中，无论对象属性返回值是什么类型，每次访问对象的属性就是属性访问。如果属性访问返回的是一个函数，那它也并不是一个“方法”。属性访问返回的函数和其他函数没有任何区别（除了可能发生的隐式绑定 this）。

有些函数具有 this 引用，有时候这些 this 确实会指向调用位置的对象引用。但是这种用法从本质上来说并没有把一个函数变成一个“方法”，因为 this 是在运行时根据调用位置动态绑定的，所以函数和对象的关系最多也只能说是间接关系。

```javascript
function foo() {
  console.log( "foo" );
}

var someFoo = foo; // 对 foo 的变量引用

var myObject = {
  someFoo: foo
};

foo; // function foo(){..}
someFoo; // function foo(){..}
myObject.someFoo; // function foo(){..}
```
someFoo 和 myObject.someFoo 只是对于同一个函数的不同引用，并不能说明这个函数是特别的或者“属于”某个对象。  
即使你在对象的文字形式中声明一个函数表达式，这个函数也不会“属于”这个对象——它们只是对于相同函数对象的多个引用。

数组与对象

数组也是对象，所以虽然每个下标都是整数，你仍然可以给数组添加属性：
```javascript
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```
可以看到虽然添加了命名属性（无论是通过 . 语法还是 [] 语法），数组的 length 值并未发生变化。

如果你试图向数组添加一个属性，但是属性名“看起来”像一个数字，那它会变成一个数值下标（因此会修改数组的内容而不是添加一个属性）：
```javascript
var myArray = [ "foo", 42, "bar" ];

myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

对象的复制

浅复制和~~深复制~~：
>浅复制是指只拷贝值，对于函数、数组等只是引用。  
>深复制不提。

以 ES6 定义了 Object.assign(..) 方法来实现浅复制。  
Object.assign(..) 方法的第一个参数是目标对象，之后还可以跟一个或多个源对象。  
它会遍历一个或多个源对象的所有可枚举（enumerable，参见下面的代码）的自有键（owned key，很快会介绍）并把它们复制（使用 = 操作符赋值，即引用）到目标对象，最后返回目标对象。

属性描述符(也称数据描述符)

```javascript
var myObject = {
  a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// value: 2,
// writable: true,
// enumerable: true,
// configurable: true
// }
```

这个普通的对象属性对应的属性描述符可不仅仅只是一个 2。它还包含另外三个特性：writable（可写）、enumerable（可枚举）和 configurable（可配置）。

在创建普通属性时属性描述符会使用默认值，我们也可以使用 Object.defineProperty(..)来添加一个新属性或者修改一个已有属性（如果它是 configurable）并对特性进行设置。

```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
} );
myObject.a; // 2
```

1. writable  
   决定是否可以修改属性的值。*可以把 writable:false 看作是属性不可改变，相当于你定义了一个空操作 setter。*

2. Configurable  
   只要属性是可配置的，就可以使用 defineProperty(..) 方法来修改 属性描述符。  
   
   **configurable修改成false是单向操作，无法撤销！**  
   
   **即便属性是 configurable:false，我们还是可以修改属性的值，可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。**  
   
   除了无法修改，configurable:false 还会**禁止删除**这个属性。

3. Enumerable  
   这个描述符控制的是属性是否会出现在对象的属性枚举中，比如说for..in 循环。

对象/属性的不变性

很重要的一点是，**所有的方法创建的都是浅不变性**，也就是说，它们只会影响目标对象和它的直接属性。如果目标对象引用了其他对象（数组、对象、函数，等），其他对象的内容不受影响，仍然是可变的。

1. 对象常量  
结合 writable:false 和 configurable:false 就可以创建一个真正的常量属性（不可修改、
重定义或者删除）

2. 禁止扩展  
如果你想禁止一个对象添加新属性并且保留已有属性，可以使用Object.preventExtensions(..)

3. 密封  
Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用Object.preventExtensions(..) 并把所有现有属性标记为 configurable:false。所以，密封之后**不仅不能添加新属性，也不能重新配置或者删除任何现有属性**（虽然可以修改属性的值）。

4. 冻结  
Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用Object.seal(..) 并把所有“数据访问”属性标记为 writable:false，这样就**无法修改它们的值**。

[[Get]]

```javascript
var myObject = {
  a: 2
};

myObject.a; // 2
```

myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作（有点像函数调用：[[Get]]()）。对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值。然而，如果没有找到名称相同的属性，按照 [[Get]] 算法的定义会执行另外一种非常重要的行为(遍历[[Prototype]]原型链)。

[[Put]]

[[Put]] 被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性（这是最重要的因素）。  
如果已经存在这个属性，[[Put]] 算法大致会检查下面这些内容。
1. 属性是否是访问描述符？如果是并且存在setter 就调用 setter。
2. 属性的数据描述符中 writable 是否是 false ？如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

如果对象中不存在这个属性，[[Put]] 操作会更加复杂。

Getter和Setter
> ES5 中可以使用 getter 和 setter 部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。  
> getter 是一个隐藏函数，会在获取属性值时调用。setter 也是一个隐藏函数，会在设置属性值时调用。

*属性不一定包含值——它们可能是具备 getter/setter 的“访问描述符”。*

当你给一个属性定义 getter、setter 或者两者都有时，这个属性会被定义为“访问描述符”（和“数据描述符”相对）。

```javascript
var myObject = {
  // 给 a 定义一个 getter
  get a() {
  return 2;
}
};
Object.defineProperty(
  myObject, // 目标对象 
  "b", // 属性名
  { // 访问描述符
    // 给 b 设置一个 getter
    get: function(){ 
      return this.a * 2 
    },
    // 确保 b 会出现在对象的属性列表中
    enumerable: true
  }
);

myObject.a; // 2  这是对属性a的访问，会调用getter
myObject.b; // 4
```

不管是对象文字语法中的 get a() { .. }，还是 defineProperty(..) 中的显式定义，二者都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数，它的返回值会被当作属性访问的返回值。

```javascript
var myObject = {
// 给 a 定义一个 getter
  get a() {
    return this._a_;
  },
//给 a 定义一个 setter
  set a(val) {
    this._a_ = val * 2;
  }
};

myObject.a = 2; // setter
myObject.a; // 4   getter
```
实际上我们把赋值（[[Put]]）操作中的值 2 存储到了另一个变量_a_ 中。

属性的存在性

```javascript
var myObject = {
  a:2
};

("a" in myObject); // true
("b" in myObject); // false

myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

in 操作符会检查属性是否在对象及其 [[Prototype]] 原型链中。  
hasOwnProperty(..) 只会检查属性是否在 myObject 对象中，不会检查 [[Prototype]] 链。

in 操作符可以检查容器内是否有某个值，但是它实际上检查的是某
个**属性名是否存在**。对于数组来说这个区别非常重要，4 in [2, 4, 6] 的结果并不是你期待的 True，因为 [2, 4, 6] 这个数组中包含的属性名是 0、1、2，没有 4。

也可以通过另一种方式来区分属性是否可枚举：
```javascript
var myObject = { };

Object.defineProperty(
  myObject,
  "a",
  // 让 a 像普通属性一样可以枚举
  { enumerable: true, value: 2 }
);

Object.defineProperty(
  myObject,
  "b",
  // 让 b 不可枚举
  { enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

propertyIsEnumerable(..) 会检查给定的属性名**是否直接存在于对象中**（而不是在原型链上）并且满足 enumerable:true。  

Object.keys(..) 会返回一个数组，包含所有可枚举属性。  
Object.getOwnPropertyNames(..)会返回一个数组，包含所有属性，无论它们是否可枚举。

Object.keys(..)和 Object.getOwnPropertyNames(..) 都只会查找**对象直接包含的属性**。

对象的遍历

for..in 循环可以用来遍历对象的可枚举属性列表（包括 [[Prototype]] 链）。但是如何遍历属性的值呢？
```javascript
var myArray = [1, 2, 3];
for (var i = 0; i < myArray.length; i++) {
  console.log( myArray[i] );
}
// 1 2 3
```
这实际上并不是在遍历值，而是遍历下标来指向值，如 myArray[i]。  
那么如何直接遍历值而不是数组下标（或者对象属性）呢？幸好，ES6 增加了一种用来遍历数组的 for..of 循环语法（如果对象本身定义了迭代器的话也可以遍历对象）:

```javascript
var myArray = [ 1, 2, 3 ];
for (var v of myArray) {
console.log( v );
}
// 1
// 2
// 3
```

```javascript
var myObject = {
  a: 2,
  b: 3
};
// 用 for..of 遍历 myObject
for (var v of myObject) {
  console.log( v );
}
// 2
// 3
```

### 混合对象“类”

>在继承或者实例化时，JavaScript 的对象机制并不会自动执行复制行为。简单来说，JavaScript 中只有对象，并不存在可以被实例化的“类”。一个对象并不会被复制到其他对象，它们会被关联起来。 
> 
>由于在其他语言中类表现出来的都是**复制行为**，因此 JavaScript 开发者也想出了一个方法来**模拟类的复制行为**，这个方法就是混入。接下来我们会看到两种类型的混入：显式和隐式。

显示混入

* 显示混入mixin()

  ```javascript
  // 非常简单的 mixin(..) 例子 :
  function mixin( sourceObj, targetObj ) {
    for (var key in sourceObj) {
    // 只会在不存在的情况下复制
      if (!(key in targetObj)) {
        targetObj[key] = sourceObj[key];
      }
    }
    return targetObj;
  }
  
  var Vehicle = {
    engines: 1,
    
    ignition: function() {
      console.log( "Turning on my engine." );
    },
    
    drive: function() {
      this.ignition();
      console.log( "Steering and moving forward!" );
    }
  };
  
  var Car = mixin( Vehicle, {
    wheels: 4,
    
    drive: function() {
      Vehicle.drive.call( this );  // 显示多态
                                   // 在面向类的语言中，   inherited:drive()，我们称   之为相对多态
      console.log(
        "Rolling on all " + this.wheels + " wheels!"
      );
    }
  } );
  ```

  JavaScript（在 ES6 之前）并没有相对多态的机制。所以，由于   Car 和Vehicle 中都有 drive() 函数，为了指明调用对象，我们必须  使用绝对（而不是相对）引用。我们通过**名称显式指定**Vehicle 对  象并调用它的 drive() 函数。
  
  如果你向目标对象中显式混入超过一个对象，就可以部分模仿多重继承  行为，但是仍没有直接的方式来处理函数和属性的同名问题。
  
  但是在 JavaScript 中（由于屏蔽）使用显式伪多态会在所有需要使用  （伪）多态引用的地方创建一个函数关联，这会极大地增加维护成本。  此外，由于显式伪多态可以模拟多重继承，所以它会进一步增加代码的  复杂度和维护难度。

* 混合复制

  ```javascript
  // 另一种混入函数，可能有重写风险
  function mixin( sourceObj, targetObj ) {
    for (var key in sourceObj) {
      targetObj[key] = sourceObj[key];
    }
    return targetObj;
  }
  
  var Vehicle = {
  // ...
  };
  
  // 首先创建一个空对象并把 Vehicle 的内容复制进去
  var Car = mixin( Vehicle, { } );
  // 然后把新内容复制到 Car 中
  mixin( {
  wheels: 4,
  drive: function() {
  // ...
  }
  }, Car );
  ```
  
  这里我们是先进行复制然后对 Car 进行特殊化的话，就可以跳过存在性  检查。不过这种方法并不好用并且效率更低，所以不如第一种方法常  用。

* 寄生继承

  ```javascript
  //“传统的 JavaScript 类”Vehicle
  function Vehicle() {
    this.engines = 1;
  }
  
  Vehicle.prototype.ignition = function() {
    console.log( "Turning on my engine." );
  };
  
  Vehicle.prototype.drive = function() {
    this.ignition();
    console.log( "Steering and moving forward!" );
  };
  
  //“寄生类”Car
  function Car() {
    // 首先，car 是一个 Vehicle
    var car = new Vehicle();
    // 接着我们对 car 进行定制
    car.wheels = 4;
    // 保存到 Vehicle::drive() 的特殊引用
    var vehDrive = car.drive;
    // 重写 Vehicle::drive()
    car.drive = function() {
      vehDrive.call( this );
      console.log(
        "Rolling on all " + this.wheels + " wheels!"
      );
    
    return car;
  }
  
  var myCar = new Car();
  // 调用 new Car() 时会创建一个新对象并绑定到 Car 的 this   上。  
  // 但是因为我们没有使用这个对象而是返回了我们自己的 car 对象，  所以最初被创建的这个对象会被丢弃，因此可以不使用 new 关键字调  用 Car()。
  // 这样做得到的结果是一样的，但是可以避免创建并丢弃多余的对象。
  
  myCar.drive();
  // 发动引擎。
  // 手握方向盘！
  // 全速前进！
  ```

隐式混入

```javascript
var Something = {
  cool: function() {
    this.greeting = "Hello World";
    this.count = this.count ? this.count + 1 : 1;
  }
};

Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1

var Another = {
  cool: function() {
  // 隐式把 Something 混入 Another
    Something.cool.call( this );
  // 显示混入不同的地方是用了mixin函数进行复制操作，但也调用了源对象。
  }
};

Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1（count 不是共享状态）
```

通过在构造函数调用或者方法调用中使用 Something.cool.call( this )，我们实际上“借用”了函数 Something.cool() 并在 Another 的上下文中调用了它（通过 this 绑定）。最终的结果是 Something.cool() 中的赋值操作都会应用在 Another 对象上而不是Something 对象上。  
因此，我们把 Something 的行为“混入”到了 Another 中。

### 原型

再谈[[Put]]

`myObject.foo = "bar";`

如果**myObject 已经存在属性foo**，[[Put]] 算法大致会检查下面这些内容。
1. 属性是否是访问描述符？如果是并且存在setter 就调用 setter。
2. 属性的数据描述符中 writable 是否是 false ？如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

如果**foo不是直接存在于 myObject** 中，[[Prototype]] 链就会被遍历，类似 [[Get]] 操作。如果原型链上找不到 foo，foo 就会被直接添加到 myObject 上。

如果**foo 不直接存在于 myObject 中而是存在于原型链上层时**, myObject.foo = "bar" 会出现的三种情况。
1. 如果在 [[Prototype]] 链上层存在名为 foo 的普通数据访问属性并且可写（writable:true），那就会直接在 myObject 中添加一个名为 foo 的新属性，它是屏蔽属性。
2. 如果在 [[Prototype]] 链上层存在 foo，但是它被标记为只读（writable:false），那么无法修改已有属性或者在 myObject 上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在 [[Prototype]] 链上层存在 foo 并且它是一个 setter，那就一定会调用这个 setter。foo 不会被添加到（或者说屏蔽于）myObject，也不会重新定义 foo 这个 setter。

[[Prototype]]

```javascript
function Foo() {
// ...
}
Foo.prototype; // { }
```

所有的函数默认都会拥有一个名为 prototype 的公有并且不可枚举的属性，它会指向另一个对象。这个对象通常被称为**Foo 的原型**，因为我们通过名为Foo.prototype 的属性引用来访问它。

这个对象是在调用 new Foo()（参见第 2 章）时创建的，最后会被（有点武断地）关联到这个“Foo 点 prototype”对象上。
我们来验证一下：
```javascript
function Foo() {
// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true
```

调用 new Foo() 时会创建 a，其中的一步就是给 a 一个**内部的 [[Prototype]] 链接**，关联到 Foo.prototype 指向的那个对象。


下面这段代码使用的就是典型的“原型风格”：

```javascript
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function() {
  return this.name;
};

function Bar(name,label) {  // 声明 function Bar() { .. } 时，和其他函数一样，Bar 会有一个 .prototype 关联到默认的对象，
                            // 但是这个对象并不是我们想要的 Foo.prototype。因此我们创建了一个新对象并把它关联到我们希望的对象上，直接把
                            // 原始的关联对象抛弃掉。
  Foo.call( this, name );
  this.label = label;
}
// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
Bar.prototype = Object.create( Foo.prototype );
// 注意！现在没有 Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() {
  return this.label;
};

var a = new Bar( "a", "obj a" );

a.myName(); // "a"
a.myLabel(); // "obj a"
```
注意，下面这两种方式是常见的错误做法，实际上它们都存在一些问题：
```javascript
// 和你想要的机制不一样！
Bar.prototype = Foo.prototype;
// 基本上满足你的需求，但是可能会产生一些副作用 :(
Bar.prototype = new Foo();
```

Bar.prototype = Foo.prototype 并不会创建一个关联到 Bar.prototype 的新对象，它只是让 Bar.prototype 直接引用 Foo.prototype 对象。

Bar.prototype = new Foo() 的确会创建一个关联到 Bar.prototype 的新对象。但是它使用了 Foo(..) 的“构造函数调用”，如果函数 Foo 有一些副作用（比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等）的话，就会影响到 Bar() 的“后代”，后果不堪设想。

因此，要创建一个合适的关联对象，我们必须使用 Object.create(..) 而不是使用具有副作用的 Foo(..)。这样做唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉，不能直接修改已有的默认对象。

ES6 添加了辅助函数 `Object.setPrototypeOf(..)`，可以用标准并且可靠的方法来修改关联。  
我们来对比一下两种把 Bar.prototype 关联到 Foo.prototype 的方法：
```javascript
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.ptototype = Object.create( Foo.prototype );
// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

如果忽略掉 Object.create(..) 方法带来的轻微性能损失（抛弃的对象需要进行垃圾回收），它实际上比 ES6 及其之后的方法更短而且可读性更高。

检查“类”关系

假设有对象 a，如何寻找对象 a 委托的对象（如果存在的话）呢？（这种行为被称为内省或者反射）

第一种方法是站在“类”的角度来判断：

    a instanceof Foo; // true

instanceof 操作符的左操作数是一个普通的对象，右操作数是一个函数。instanceof 回答的问题是：在 a 的整条 [[Prototype]] 链中是否有指向 Foo.prototype 的对象？可惜，这个方法只能处理对象（a）和函数（带 .prototype 引用的 Foo）之间的关系。

下面是第二种判断 [[Prototype]] 反射的方法，它更加简洁：  
    Foo.prototype.isPrototypeOf( a ); // true

注意，在本例中，我们实际上并不关心（甚至不需要）Foo，我们只需要任何一个可以用来判断的对象（本例中是 Foo.prototype）就行。isPrototypeOf(..) 回答的问题是：在 a 的整条 [[Prototype]] 链中是否出现过 Foo.prototype ？

可以验证一下，这个对象引用是否和我们想的一样：  

    Object.getPrototypeOf( a ) === Foo.prototype; // true

绝大多数（不是所有！）浏览器也支持一种非标准的方法来访问内部 [[Prototype]] 属性：a.__proto__ === Foo.prototype; // true

对象关联

```javascript
var foo = {
  something: function() {
    console.log( "Tell me something good..." );
  }
};

var bar = Object.create( foo );

bar.something(); // Tell me something good...
```

Object.create(..) 会创建一个新对象（bar）并把它关联到我们指定的对象（foo），这样我们就可以充分发挥 [[Prototype]] 机制的威力（委托）并且避免不必要的麻烦（比如使用 new 的构造函数调用会生成 .prototype 和 .constructor 引用）。

**Object.create(null) 会 创 建 一 个 拥 有 空（ 或 者 说 null）[[Prototype]]链接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以instanceof 操作符（之前解释过）无法进行判断，因此总是会返回 false。这些特殊的空 [[Prototype]] 对象通常被称作“字典”，它们完全不会受到原型链的干扰，因此非常适合用来**存储数据**。**

Object.create()的polyfill代码


Object.create(..) 是在 ES5 中新增的函数，所以在 ES5 之前的环境中（比如旧 IE）如果要支持这个功能的话就需要使用一段简单的 polyfill 代码，它部分实现了 Object.create(..) 的功能：
```javascript
if (!Object.create) {
  Object.create = function(o) {
    function F(){}
    F.prototype = o;
    return new F();
  };
}
```

这段 polyfill 代码使用了一个一次性函数 F，我们通过改写它的 .prototype 属性使其指向想要关联的对象，然后再使用 new F() 来构造一个新对象进行关联。

当你给开发者设计软件时，假设要调用 myObject.cool()，如果 myObject 中不存在 cool()时这条语句也可以正常工作的话，那你的 API 设计就会变得很“神奇”，对于未来维护你软件的开发者来说这可能不太好理解。

但是你可以让你的 API 设计不那么“神奇”，同时仍然能发挥 [[Prototype]] 关联的威力：
```javascript
var anotherObject = {
  cool: function() {
    console.log( "cool!" );
  }
};

var myObject = Object.create( anotherObject );

myObject.doCool = function() {
  this.cool(); // 内部委托！
};

myObject.doCool(); // "cool!"
```

这里我们调用的 myObject.doCool() 是实际存在于 myObject 中的，这可以让我们的 API 设计更加清晰（不那么“神奇”）。

### 行为委托

类理论

假设我们需要在软件中建模一些类似的任务（“XYZ”、“ABC”等）。如果使用类，那设计方法可能是这样的：定义一个通用父（基）类，可以将其命名为
Task，在 Task 类中定义所有任务都有的行为。接着定义子类 XYZ 和 ABC，它们都继承自Task 并且会添加一些特殊的行为来处理对应的任务。

```javascript
下面是对应的伪代码：
class Task {
  id;
  // 构造函数 Task()
  Task(ID) { id = ID; }
  outputTask() { output( id ); }
}

class XYZ inherits Task {
  label;
  // 构造函数 XYZ()
  XYZ(ID,Label) { super( ID ); label = Label; }
  outputTask() { super(); output( label ); }
}

class ABC inherits Task {
  // ...
}
```

委托理论

首先你会定义一个名为 Task 的对象（和许多 JavaScript 开发者告诉你的不同，它既不是类也不是函数），它会包含所有任务都可以使用（写作使用，读作委托）的具体行为。接着，对于每个任务（“XYZ”、“ABC”）你都会定义一个对象来存储对应的数据和行为。你会把特定的任务对象都关联到 Task 功能对象上，让它们在需要的时候可以进行委托。

基本上你可以想象成，执行任务“XYZ”需要两个兄弟对象（XYZ 和 Task）协作完成。但是我们并不需要把这些行为放在一起，通过类的复制，我们可以把它们分别放在各自独立的对象中，需要时可以允许 XYZ 对象委托给 Task。

```javascript
Task = {
  setID: function(ID) { this.id = ID; },
  outputID: function() { console.log( this.id ); }
};
// 让 XYZ 委托 Task
XYZ = Object.create( Task );

XYZ.prepareTask = function(ID,Label) {
  this.setID( ID );
  this.label = Label;
};

XYZ.outputTaskDetails = function() {
  this.outputID();
  console.log( this.label );
};

// ABC = Object.create( Task );
// ABC ... = ...
```

相比于面向类（或者说面向对象），我会把这种编码风格称为“对象关联”（OLOO，objects linked to other objects）。

对象关联风格的代码还有一些不同之处。
1. 在上面的代码中，id 和 label 数据成员都是直接存储在 XYZ 上（而不是 Task）。通常来说，在 [[Prototype]] 委托中最好把状态保存在委托者（XYZ、ABC）而不是委托目标（Task）上。
   
2. 在类设计模式中，我们故意让父类（Task）和子类（XYZ）中都有 outputTask 方法，这样就可以利用重写（多态）的优势。在委托行为中则恰好相反：我们会尽量避免在[[Prototype]] 链的不同级别中使用相同的命名，否则就需要使用笨拙并且脆弱的语法来消除引用歧义。这个设计模式要求尽量少使用容易被重写的通用方法名，提倡使用更有描述性的方法名，尤其是要写清相应对象行为的类型。这样做实际上可以创建出更容易理解和维护的代码，因为方法名（不仅在定义的位置，而是贯穿整个代码）更加清晰（自文档）。

3. this.setID(ID)；XYZ 中的方法首先会寻找 XYZ 自身是否有 setID(..)，但是 XYZ 中并没有这个方法名，因此会通过 [[Prototype]] 委托关联到 Task 继续寻找，这时就可以找到setID(..) 方法。此外，由于调用位置触发了 this 的隐式绑定规则，因此虽然 setID(..) 方法在 Task 中，运行时 this 仍然会绑定到 XYZ，这正是我们想要的。在之后的代码中我们还会看到 this.outputID()，原理相同。

比较思维模型

下面是典型的（“原型”）面向对象风格：

```javascript
function Foo(who) {
  this.me = who;
}

Foo.prototype.identify = function() {
  return "I am " + this.me;
};

function Bar(who) {
  Foo.call( this, who );
}

Bar.prototype = Object.create( Foo.prototype );

Bar.prototype.speak = function() {
  alert( "Hello, " + this.identify() + "." );
};

var b1 = new Bar( "b1" );
var b2 = new Bar( "b2" );

b1.speak();
b2.speak();
```

下面我们看看如何使用对象关联风格来编写功能完全相同的代码：

```javascript
Foo = {
  init: function(who) {
    this.me = who;
  },
  
  identify: function() {
    return "I am " + this.me;
  }
};

Bar = Object.create( Foo );

Bar.speak = function() {
  alert( "Hello, " + this.identify() + "." );
};

var b1 = Object.create( Bar );
b1.init( "b1" );
var b2 = Object.create( Bar );
b2.init( "b2" );

b1.speak();
b2.speak();
```

非常重要的一点是，这段代码简洁了许多，我们只是把对象关联起来，并不需要那些既复杂又令人困惑的模仿类的行为（构造函数、原型以及 new）。

更好的语法

 ES6中我们可以在任意对象的字面形式中使用简洁方法声明（concise method declaration），所以对象关联风格的对象可以这样声明（和 class 的语法糖一样）：

```javascript
 var LoginController = {
  errors: [],
  getUser() { // 妈妈再也不用担心代码里有 function 了！
  // ...
  },
  getPassword() {
  // ...
  }
  // ...
};
// 使用更好的对象字面形式语法和简洁方法
var AuthController = {
errors: [],  // 内部委托
checkAuth() {
// ...
},
server(url,data) {
// ...
}
// ...
};
// 现在把 AuthController 关联到 LoginController
Object.setPrototypeOf( AuthController, LoginController );
```

但简洁方法有一个非常小但是非常重要的缺点。思考下面的代码：
```javascript
var Foo = {
  bar() { /*..*/ },
  baz: function baz() { /*..*/ }
};

去掉语法糖之后的代码如下所示：

var Foo = {
  bar: function() { /*..*/ },
  baz: function baz() { /*..*/ }
};
```
由于函数本身并没有名称标识符，所以bar()的缩写形式（function()..）实际上会变成一个匿名函数表达式并赋值给 bar 属性。相比之下，具名函数表达式（function baz()..）会额外给 .baz 属性附加一个词法名称标识符 baz。如果需要自我引用的话，那最好使用传统的具名函数表达式来定义对应的函数（baz: function baz(){..}），不要使用简洁方法。

内省（查找实例）

```javascript
function Foo() { /* .. */ }
Foo.prototype...

function Bar() { /* .. */ }
Bar.prototype = Object.create( Foo.prototype );

var b1 = new Bar( "b1" );
```

如果要使用 instanceof 和 .prototype 语义来检查本例中实体的关系，那必须这样做：

```javascript
// 让 Foo 和 Bar 互相关联
Bar.prototype instanceof Foo; // true

Object.getPrototypeOf( Bar.prototype ) === Foo.prototype; // true

Foo.prototype.isPrototypeOf( Bar.prototype ); // true

// 让 b1 关联到 Foo 和 Bar
b1 instanceof Foo; // true
b1 instanceof Bar; // true

Object.getPrototypeOf( b1 ) === Bar.prototype; // true

Foo.prototype.isPrototypeOf( b1 ); // true
Bar.prototype.isPrototypeOf( b1 ); // true
```

还有一种常见但是可能更加脆弱的内省模式，许多开发者认为它比 instanceof 更好。这种模式被称为“鸭子类型”。

```javascript
if (a1.something) {
  a1.something();
}
```

ES6 的 Promise 就是典型的“鸭子类型”。出于各种各样的原因，我们需要判断一个对象引用是否是 Promise，但是判断的方法是检查对象是否有 then() 方法。换句话说，如果对象有 then() 方法，ES6 的 Promise 就会认为这个对象是“可持续”（thenable）的，因此会期望它具有 Promise 的所有标准行为。

而对象关联风格代码，其内省更加简洁。我们先来回顾一下之前的 Foo/Bar/b1 对象关联例子（只包含关键代码）：

```javascript
var Foo = { /* .. */ };

var Bar = Object.create( Foo );
Bar...

var b1 = Object.create( Bar );

// 使用对象关联时，所有的对象都是通过 [[Prototype]] 委托互相关联，下面是内省的方法，非常简单：

// 让 Foo 和 Bar 互相关联
Foo.isPrototypeOf( Bar ); // true
Object.getPrototypeOf( Bar ) === Foo; // true

// 让 b1 关联到 Foo 和 Bar
Foo.isPrototypeOf( b1 ); // true
Bar.isPrototypeOf( b1 ); // true
Object.getPrototypeOf( b1 ) === Bar; // true
```
