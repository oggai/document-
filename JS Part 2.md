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
   只要属性是可配置的，就可以使用 defineProperty(..) 方法来修改   属性描述符。  
   
   **configurable修改成false是单向操作，无法撤销！**  
   
   **即便属性是 configurable:false，我们还是可以修改属性的值，可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。**  
   
   除了无法修改，configurable:false 还会**禁止删除**这个属性。

3. Enumerable  
   这个描述符控制的是属性是否会出现在对象的属性枚举中，比如说for..in 循环。

对象/属性的不变性

很重要的一点是，**所有的方法创建的都是浅不变形**，也就是说，它们只会影响目标对象和它的直接属性。如果目标对象引用了其他对象（数组、对象、函数，等），其他对象的内容不受影响，仍然是可变的。

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

161
