# you don't know JS 学习笔记

- [you don't know JS 学习笔记](#you-dont-know-js-%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0)
  - [第1章 类型](#%e7%ac%ac1%e7%ab%a0-%e7%b1%bb%e5%9e%8b)
  - [第2章 值](#%e7%ac%ac2%e7%ab%a0-%e5%80%bc)
  - [第3章 原生函数](#%e7%ac%ac3%e7%ab%a0-%e5%8e%9f%e7%94%9f%e5%87%bd%e6%95%b0)
  - [第4章 强制类型转换](#%e7%ac%ac4%e7%ab%a0-%e5%bc%ba%e5%88%b6%e7%b1%bb%e5%9e%8b%e8%bd%ac%e6%8d%a2)
  - [第5章 语法](#%e7%ac%ac5%e7%ab%a0-%e8%af%ad%e6%b3%95)

## 第1章 类型

>类型是值的内部特征，它定义了值的行为，以使其区别于其他值。

*换句话说，如果语言引擎和开发人员对 42（数字）和 "42"（字符串）采取不同的处理方式，那就说明它们是不同的类型，一个是number，一个是 string。通常我们对数字 42 进行数学运算，而对字符串 "42" 进行字符串操作，比如输出到页面。它们是不同的类型。*

内置类型

* 空值（null）
* 未定义（undefined）
* 布尔值（ boolean）
* 数字（number）
* 字符串（string）
* 对象（object）
* 符号（symbol，ES6 中新增）

*除对象之外，其他统称为“基本类型”。函数和数组是对象的子类型。*

我们可以用 typeof 运算符来查看值的类型，它**返回的是类型的字符串值**。但是，  
    
    typeof null === "object"; // true

我们需要使用复合条件来检测 null 值的类型：

    var a = null;
    (!a && typeof a === "object"); // true

null 是基本类型中唯一的一个“假值”类型，typeof对它的返回值为 "object"。


undefined & undeclared

```javascript
var a;
// 已在作用域中声明但还没有赋值的变量，是 undefined 的。相反，还没有在作用域中声明过的变量，是 undeclared 的。
a; // undefined
b; // ReferenceError: b is not defined
```

```javascript
var a;

typeof a; // "undefined"
typeof b; // "undefined"
```

对于 undeclared（或者 not defined）变量，typeof 照样返回"undefined"。请注意虽然 b 是一个 undeclared 变量，但 typeof b 并没有报错。这是因为 typeof 有一个特殊的安全防范机制。

如何在程序中检查全局变量 DEBUG 才不会出现 ReferenceError 错误。这时 typeof 的安全防范机制就成了我们的好帮手：
```javascript
// 这样会抛出错误
if (DEBUG) {
  console.log( "Debugging is starting" );
}
// 这样是安全的
if (typeof DEBUG !== "undefined") {
  console.log( "Debugging is starting" );
}
```

这不仅对用户定义的变量（比如 DEBUG）有用，对内建的 API 也有帮助：
```javascript
if (typeof atob === "undefined") {
  atob = function() { /*..*/ };
}
```

如果要为某个缺失的功能写 polyfill代码，一般不会用 var atob 来声明变量 atob。如果在 if 语句中使用 var atob，声明会被提升到作用域（即当前脚本或函数的作用域）的最顶层，即使 if 条件不成立也是如此（因为 atob 全局变量已经存在）。

还有一种不用通过 typeof 的安全防范机制的方法，就是检查所有全局变量是否是全局对象的属性，浏览器中的全局对象是 window。所以前面的例子也可以这样来实现：

```javascript
if (window.DEBUG) {
 // ..
}
if (!window.atob) {
 // ..
}
```

与 undeclared 变量不同，访问不存在的对象属性（甚至是在全局对象 window 上）不会产生ReferenceError 错误。

## 第2章 值

数组

有时需要将**类数组**(一组通过数字索引的值)转换为真正的数组，这一般通过数组工具函数（如 indexOf(..)、concat(..)、forEach(..) 等）来实现。

例如，一些 DOM 查询操作会返回 DOM 元素列表，它们并非真正意义上的数组，但十分类似。另一个例子是通过 arguments 对象（类数组）将函数的参数当作列表来访问。

工具函数 slice(..) 经常被用于这类转换：

```javascript
function foo() {
  var arr = Array.prototype.slice.call( arguments );
  arr.push( "bam" );
  console.log( arr );
}
foo( "bar", "baz" ); // ["bar","baz","bam"]
```

如上所示，slice() 返回参数列表（上例中是一个类数组）的一个数组复本。

用 ES6 中的内置工具函数 Array.from(..) 也能实现同样的功能：

```javascript
...
var arr = Array.from( arguments );
...
```

字符串

字符串和数组的确很相似，它们都是类数组，都有 length 属性以及 indexOf(..)（从 ES5开始数组支持此方法）和 concat(..) 方法。但这并不意味着它们都是“字符数组”，比如：

```javascript
a[1] = "O";
b[1] = "O";
a; // "foo"
b; // ["f","O","o"]
```

JavaScript 中字符串是不可变的，而数组是可变的。字符串不可变是指字符串的成员函数不会改变其原始值，而是创建并返回一个新的字符串。而数组的成员函数都是在其原始值上进行操作。

```javascript
c = a.toUpperCase();
a === c; // false

a; // "foo"
c; // "FOO"

b.push( "!" );
b; // ["f","O","o","!"]
```

许多数组函数用来处理字符串很方便。虽然字符串没有这些函数，但可以通过“借用”**数组的非变更方法**来处理字符串：
```javascript
a.join; // undefined
a.map; // undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
  return v.toUpperCase() + ".";
} ).join( "" );

c; // "f-o-o"
d; // "F.O.O."
```

另一个不同点在于字符串反转（JavaScript 面试常见问题）。数组有一个字符串没有的可变更成员函数 reverse()：
```javascript
a.reverse; // undefined
b.reverse(); // ["!","o","O","f"]
b; // ["f","O","o","!"]
```
我们无法“借用”数组的可变更成员函数，因为字符串是不可变的。一个变通（破解）的办法是先将字符串转换为数组，待处理完后再将结果转换回字符串：

```javascript
var c = a
    // 将a的值转换为字符数组
    .split( "" )
    // 将数组中的字符进行倒转
    .reverse()
    // 将数组中的字符拼接回字符串
    .join( "" );

c; // "oof"
```

数字

JavaScript 只有一种数值类型：number（数字），包括“整数”和**带小数的十进制数**。此处“整数”之所以加引号是因为和其他语言不同，JavaScript 没有真正意义上的整数。**“整数”就是没有小数的十进制数**。所以 42.0 即等同于“整数”42。

由于数字值可以使用 Number 对象进行封装，因此数字值可以调用 Number.prototype 中的方法。例如，tofixed(..) 方法可指定小数部分的显示位数。对于 . 运算符需要给予特别注意，因为它是一个有效的数字字符，会被优先识别为数字常量的一部分，然后才是对象属性访问运算符。

```javascript
// 无效语法：
42.toFixed( 3 ); // SyntaxError
// 下面的语法都有效：
(42).toFixed( 3 ); // "42.000"
0.42.toFixed( 3 ); // "0.420"
42..toFixed( 3 ); // "42.000"
42 .toFixed(3); // "42.000" 注意其中的空格
```

42.tofixed(3) 是无效语法，因为 . 被视为常量 42. 的一部分（如前所述），所以没有 . 属性访问运算符来调用 tofixed 方法。

数字常量还可以用其他格式来表示，如二进制、八进制和十六进制。建
议尽量使用小写的 0x、0o 和0b 。

二进制浮点数最大的问题（不仅 JavaScript，所有遵循 IEEE 754 规范的语言都是如此），是会出现如下情况：

    0.1 + 0.2 === 0.3; // false

从数学角度来说，上面的条件判断应该为 true，可结果为什么是 false 呢？简单来说，二进制浮点数中的 0.1 和 0.2 并不是十分精确，它们相加的结果并非刚好等于0.3，而是一个比较接近的数字 0.30000000000000004，所以条件判断结果为 false。

是设置一个误差范围值，通常称为“机器精度”（machine epsilon），对JavaScript 的数字来说，这个值通常是 2^-52 (2220446049250313e-16)。从 ES6 开始，该值定义在 Number.EPSILON 中，我们可以直接拿来用，也可以为 ES6 之前
的版本写 polyfill：

```javascript
if (!Number.EPSILON) {
 Number.EPSILON = Math.pow(2,-52);
}
```

可以使用 Number.EPSILON 来比较两个数字是否相等（在指定的误差范围内）：
```javascript
function numbersCloseEnoughToEqual(n1,n2) {
  return Math.abs( n1 - n2 ) < Number.EPSILON;
}

var a = 0.1 + 0.2;
var b = 0.3;

numbersCloseEnoughToEqual( a, b ); // true
numbersCloseEnoughToEqual( 0.0000001, 0.0000002 ); // false
```

数字的呈现方式决定了“整数”的安全值范围远远小于 Number.MAX_VALUE。能够被“安全”呈现的最大整数是 2^53 - 1，即 9007199254740991，在 ES6 中被定义为Number.MAX_SAFE_INTEGER。最小整数是 -9007199254740991，在 ES6 中被定义为 Number.MIN_SAFE_INTEGER。

要检测一个值是否是整数，可以使用 ES6 中的 Number.isInteger(..) 方法;要检测一个值是否是安全的整数，可以使用 ES6 中的 Number.isSafeInteger(..) 方法。

虽然整数最大能够达到 53 位，但是有些数字操作（如数位操作）只适用于 32 位数字，所以这些操作中数字的安全范围就要小很多，变成从 Math.pow(-2,31)（-2147483648，约－21 亿）到 Math.pow(2,31) - 1（2147483647，约 21 亿）。

**`a | 0`** 可以将变量 a 中的数值转换为 32 位有符号整数，因为数位运算符 | 只适用于 32 位整数（它只关心 32 位以内的值，其他的数位将被忽略）。因此**与 0 进行操作即可截取 a 中的 32 位数位**。

特殊数值

undefined & null

undefined 类型只有一个值，即 undefined。null 类型也只有一个值，即 null。它们的名称既是类型也是值。undefined 指从未赋值，null 指曾赋过值，但是目前没有值。

undefined 是一个内置标识符，它的值为 undefined，通过 void 运算符即可得到该值。表达式 void ___ 没有返回值，因此返回结果是 undefined。void 并不改变表达式的结果，只是让表达式不返回值：
    
    var a = 42;
    
    console.log( void a, a ); // undefined 42


NaN

如果数学运算的操作数不是数字类型（或者无法解析为常规的十进制或十六进制数字），就无法返回一个有效的数字，这种情况下返回值为 NaN。

```javascript
var a = 2 / "foo"; // NaN
typeof a === "number"; // true

a == NaN; // false
a === NaN; // false
```

NaN 是一个特殊值，它和自身不相等，是唯一一个非自反（自反，reflexive，即 x === x 不成立）的值。而 **NaN != NaN 为 true**，很奇怪吧？

可以使用内建的全局工具函数 isNaN(..) 来判断一个值是否是 NaN。但isNaN(..) 有一个严重的缺陷，它的检查方式过于死板，就是“检查参数是否不是 NaN，也不是数字”。

```javascript
var a = 2 / "foo";
var b = "foo";

a; // NaN
b; "foo"

window.isNaN( a ); // true
window.isNaN( b ); // true——晕！
```

从 ES6 开始我们可以使用工具函数 Number.isNaN(..)。ES6 之前的浏览器的 polyfill 如下：
```javascript
if (!Number.isNaN) {
  Number.isNaN = function(n) {
    return (
      typeof n === "number" &&
      window.isNaN( n )
    );
  };
}

var a = 2 / "foo";
var b = "foo";

Number.isNaN( a ); // true
Number.isNaN( b ); // false——好！
```

实际上还有一个更简单的方法，即利用 NaN 不等于自身这个特点。NaN 是 JavaScript 中唯一一个不等于自身的值。于是我们可以这样：

```javascript
if (!Number.isNaN) {
  Number.isNaN = function(n) {
    return n !== n;
  };
}
```

无穷数

```javascript
var a = 1 / 0; // Infinity
var b = -1 / 0; // -Infinity
```

JavaScript 使用有限数字表示法，所以和纯粹的数学运算不同，JavaScript 的运算结果有可能溢出，此时结果为Infinity 或者 -Infinity。

```javascript
var a = Number.MAX_VALUE; // 1.7976931348623157e+308

a + a; // Infinity
a + Math.pow( 2, 970 ); // Infinity
a + Math.pow( 2, 969 ); // 1.7976931348623157e+308
```

规范规定，如果数学运算（如加法）的结果超出处理范围，则由 IEEE 754 规范中的“就近取整”（round-to-nearest）模式来决定最后的结果。

例如，相对于 Infinity，Number.MAX_VALUE + Math.pow(2, 969) 与 Number.MAX_VALUE 更为接近，因此它被“**向下取整**”（round
down）；  
而 Number.MAX_VALUE + Math.pow(2, 970) 与Infinity 更为接近，所以它被“**向上取整**”（round up）。

Infinity/Infinity 是一个未定义操作，结果为 NaN。

零值

JavaScript 有一个常规的 0（也叫作+0）和一个 -0。-0 除了可以用作常量以外，也可以是某些数学运算的返回值。例如：

    var a = 0 / -3; // -0
    var b = 0 * -3; // -0

而加法和减法运算不会得到负零（negative zero）。

根据规范，对**负零进行字符串化会返回 "0"**：

```javascript
var a = 0 / -3;
// 至少在某些浏览器的控制台中显示是正确的
a; // -0
// 但是规范定义的返回结果是这样！
a.toString(); // "0"
a + ""; // "0"
String( a ); // "0"
// JSON也如此，很奇怪
JSON.stringify( a ); // "0"
```
有意思的是，如果**反过来将其从字符串转换为数字，得到的结果是准确的**：

```javascript
+"-0"; // -0
Number( "-0" ); // -0
JSON.parse( "-0" ); // -0
```

负零转换为字符串的结果令人费解，它的比较操作也是如此：

```javascript
var a = 0;
var b = 0 / -3;

a == b; // true
-0 == 0; // true

a === b; // true
-0 === 0; // true

0 > -0; // false
a > b; // false
```

要区分 -0 和 0，不能仅仅依赖开发调试窗口的显示结果，还需要做一些特殊处理：

```javascript
function isNegZero(n) {
  n = Number( n );
  return (n === 0) && (1 / n === -Infinity);
}

isNegZero( -0 ); // true
isNegZero( 0 / -3 ); // true
isNegZero( 0 ); // false
```
我们为什么需要负零呢？

有些应用程序中的数据需要以级数形式来表示（比如动画帧的移动速度），*数字的符号位（sign）用来代表其他信息（比如移动的方向）。此时如果一个值为 0 的变量失去了它的符号位，它的方向信息就会丢失*。所以保留 0 值的符号位可以防止这类情况发生。

特殊等式

如前所述，NaN 和 -0 在相等比较时的表现有些特别。由于 NaN 和自身不相等，所以必须使用 ES6 中的 Number.isNaN(..)（或者polyfill）。而 -0 等于 0（对于 === 也是如此），因此我们必须使用 isNegZero(..) 这样的工具函数。

ES6 中新加入了一个工具方法 Object.is(..) 来判断两个值是否绝对相等，可以用来处理
上述所有的特殊情况：

```javascript
var a = 2 / "foo";
var b = -3 * 0;

Object.is( a, NaN ); // true
Object.is( b, -0 ); // true
Object.is( b, 0 ); // false
```

能使用 == 和 ===时就尽量不要使用 Object.is(..)，因为前者效率更高、更为通用。**Object.is(..) 主要用来处理那些特殊的相等比较**。

值和引用

在 JavaScript 中变量不可能成为指向另一个变量的引用。JavaScript **引用指向的是值**。如果一个值有 10 个引用，这些引用指向的都是同一个值，它们相互之间没有引用 / 指向关系。JavaScript 对值和引用的赋值 / 传递在语法上没有区别，完全**根据值的类型来决定**。

```javascript
var a = 2;
var b = a; // b是a的值的一个副本

b++;
a; // 2
b; // 3

var c = [1,2,3];
var d = c; // d是[1,2,3]的一个引用

d.push( 4 );
c; // [1,2,3,4]
d; // [1,2,3,4]
```

**简单值（即标量基本类型值，scalar primitive）总是通过值复制的方式来赋值 / 传递，包括null、undefined、字符串、数字、布尔和 ES6 中的 symbol。**

**复合值（compound value）——对象（包括数组和封装对象）和函数，则总是通过引用复制的方式来赋值 / 传递。**

如果**通过值复制的方式来传递复合值（如数组）**，就需要为其创建一个复本，这样传递的就不再是原始值。例如：

    foo( a.slice() );

slice(..) 不带参数会返回当前数组的一个浅复本（shallow copy）。由于传递给函数的是指向该复本的引用，所以 foo(..) 中的操作不会影响 a 指向的数组。

相反，如果**要将标量基本类型值传递到函数内并进行更改**，就需要将该值封装到一个复合值（对象、数组等）中，然后通过引用复制的方式传递。

```javascript
foo(wrapper) {
  wrapper.a = 42;
}

var obj = {
  a: 2
};

foo( obj );
obj.a; // 42
```

这里 obj 是一个封装了标量基本类型值 a 的**封装对象**。obj 引用的一个复本作为参数wrapper 被传递到 foo(..) 中。这样我们就可以通过 wrapper 来访问该对象并**更改它的属性**。函数执行结束后 obj.a 将变成 42。

这样看来，如果需要传递指向标量基本类型值（比如 2）的引用，就可以将其封装到对应的数字封装对象中。与预期不同的是，虽然传递的是指向数字对象的引用复本，但我们**并不能通过它来更改其中的基本类型值**：

```javascript
function foo(x) {
  x = x + 1;
  x; // 3
}

var a = 2;
var b = new Number( a ); // Object(a)也一样

foo( b );
console.log( b ); // 是2，不是3
```

原因是**标量基本类型值是不可更改的（字符串和布尔也是如此）**。如果一个数字对象的标量基本类型值是 2，那么该值就不能更改，除非创建一个包含新值的数字对象。

x = x + 1 中，x 中的标量基本类型值 2 从数字对象中拆封（或者提取）出来后，x 就神不知鬼不觉地从引用变成了数字对象，它的值为 2 + 1 等于 3。然而函数外的 b 仍然指向原来那个值为 2 的数字对象。

## 第3章 原生函数

常用的原生函数有：

* String()
* Number()
* Boolean()
* Array()
* Object()
* Function()
* RegExp()
* Date()
* Error()
* Symbol()——ES6 中新加入的！

实际上，它们就是内建函数。

原生函数可以被当作构造函数来使用，但其构造出来的对象可能会和我们设想的有所出入：

```javascript
var a = new String( "abc" );
typeof a; // 是"object"，不是"String"
a instanceof String; // true
Object.prototype.toString.call( a ); // "[object String]"
```

通过构造函数（如 new String("abc")）创建出来的是封装了基本类型值（如 "abc"）的封装对象。*请注意：typeof 在这里返回的是对象类型的子类型。*

内部属性[[Class]]

所有 **typeof 返回值为 "object" 的对象**（如数组）都包含一个内部属性 [[Class]]（我们可以把它看作一个内部的分类）。这个属性无法直接访问，**一般通过 Object.prototype.toString(..)** 来查看。例如：

```javascript
Object.prototype.toString.call( [1,2,3] );
// "[object Array]"
Object.prototype.toString.call( /regex-literal/i );
// "[object RegExp]"
```

上例中，数组的内部 [[Class]] 属性值是 "Array"，正则表达式的值是 "RegExp"。*多数情况下*，对象的内部 [[Class]] 属性和创建该对象的内建原生构造函数相对应。

那么基本类型值呢？下面先来看看 null 和 undefined：

```javascript
Object.prototype.toString.call( null );
// "[object Null]"
Object.prototype.toString.call( undefined );
// "[object Undefined]"
```

虽然 Null() 和 Undefined() 这样的原生构造函数并不存在，但是内部 [[Class]] 属性值仍然是 "Null" 和 "Undefined"。

其他基本类型值（如字符串、数字和布尔）的情况有所不同，通常称为“包装”（boxing):

```javascript
Object.prototype.toString.call( "abc" );
// "[object String]"
Object.prototype.toString.call( 42 );
// "[object Number]"
Object.prototype.toString.call( true );
// "[object Boolean]"
```

上例中基本类型值被各自的封装对象**自动包装**，所以它们的内部 [[Class]] 属性值分别为
"String"、"Number" 和 "Boolean"。

一般情况下，我们不需要直接使用封装对象。最好的办法是让 JavaScript 引擎自己决定什么时候应该使用封装对象。

如果想要自行封装基本类型值，可以使用 Object(..) 函数（不带 new 关键字）;如果想要得到封装对象中的基本类型值，可以使用 valueOf() 函数。

原生函数作为构造函数

关于数组（array）、对象（object）、函数（function）和正则表达式，我们通常喜欢以常量的形式来创建它们。实际上，使用常量和使用构造函数的效果是一样的（创建的值都是通过封装对象来包装）。

Array()

Array 构造函数只带一个数字参数的时候，该参数会被作为数组的预设长度（length），而非只充当数组中的一个元素。这实非明智之举：一是容易忘记，二是容易出错。

更为关键的是，数组并没有预设长度这个概念。这样创建出来的只是一个空数组，只不过它的 length 属性被设置成了指定的值。

*我们将包含至少一个“空单元”的数组称为“稀疏数组”。*

```javascript
var a = new Array( 3 );
var b = [ undefined, undefined, undefined ];
var c = [];
c.length = 3;

a;
b;
c;
```

我们可以创建包含空单元的数组，如上例中的 c。只要将 length 属性设置为超过实际单元数的值，就能隐式地制造出空单元。另外还可以通过 deleteb[1] 在数组 b 中制造出一个空单元。

在当前版本的 Chrome 中显示为 [ undefined, undefined,undefined ]，而 a 和 c 则显示为 [ undefined x 3 ]。

我们可以通过下述方式来创建包含 undefined 单元（而非“空单元”）的数组：

```javascript
var a = Array.apply( null, { length: 3 } );
a; // [ undefined, undefined, undefined ]
```

是 Array.apply(..) 调用 Array(..) 函数，并且将 { length: 3 } 作为函数的参数。我们可以设想 apply(..) 内部有一个 for 循环，从 0 开始循环到length（即循环到 2，不包括 3）。假设在 apply(..) 内部该数组参数名为 arr，for 循环就会这样来遍历数组：arr[0]、arr[1]、arr[2]。 然而，由于 { length: 3 } 中并不存在这样的属性，所以返回值为undefined。

换句话说，我们执行的实际上是 Array(undefined, undefined, undefined)，所以结果是单元值为 undefined 的数组，而非空单元数组。

Object(..)、Function(..) 和 RegExp(..)

在实际情况中没有必要使用 new Object() 来创建对象，因为这样就无法像常量形式那样一次设定多个属性，而必须逐一设定。构造函数 Function 只在极少数情况下很有用，比如动态定义函数参数和函数体的时候。

Date(..) 和 Error(..)

相较于其他原生构造函数，Date(..) 和 Error(..) 的用处要大很多，因为没有对应的常量形式来作为它们的替代。

创建日期对象必须使用 new Date()。Date(..) 可以带参数，用来指定日期和时间，而不带参数的话则使用当前的日期和时间。Date(..) 主要用来获得当前的 Unix 时间戳。该值可以通过日期对象中的 getTime() 来获得。

从 ES5 开始引入了一个更简单的方法，即静态函数 **Date.now()**。对 ES5 之前的版本我们可以使用下面的 polyfill：

```javascript
if (!Date.now) {
  Date.now = function(){
    return (new Date()).getTime();
  };
}
```
构造函数 Error(..)（与前面的 Array() 类似）带不带 new 关键字都可。

Symbol(..)

符号是具有唯一性的特殊值（并非绝对），用它来命名对象属性不容易导致重名。符号可以用作属性名，但无论是在代码还是开发控制台中都无法查看和访问它的值，只会显示为诸如 **Symbol(Symbol.create)** 这样的值。

我们可以使用 Symbol(..) 原生构造函数来自定义符号。但它比较特殊，不能带 new 关键字，否则会出错：

```javascript
var mysym = Symbol( "my own symbol" );

mysym; // Symbol(my own symbol)
mysym.toString(); // "Symbol(my own symbol)"
typeof mysym; // "symbol"

var a = { };

a[mysym] = "foobar";
Object.getOwnPropertySymbols( a );
// [ Symbol(my own symbol) ]
```

原生原型

原生构造函数有自己的 .prototype 对象，如 Array.prototype、String.prototype 等。这些对象包含其对应子类型所特有的行为特征。将字符串值封装为字符串对象之后，就能访问 String.prototype 中定义的方法。

其他构造函数的原型包含它们各自类型所特有的行为特征，比如 Number#tofixed(..)（将数字转换为指定长度的整数字符串）和Array#concat(..)（合并数组）。所有的函数都可以调用 Function.prototype 中的 apply(..)、call(..) 和 bind(..)。

然而，有些原生原型（native prototype）并非普通对象那么简单：

```javascript
typeof Function.prototype; // "function"
Function.prototype(); // 空函数！

RegExp.prototype.toString(); // "/(?:)/"——空正则表达式
"abc".match( RegExp.prototype ); // [""]
```

更糟糕的是，我们甚至可以修改它们（而不仅仅是添加属性）：
```javascript
Array.isArray( Array.prototype ); // true
Array.prototype.push( 1, 2, 3 ); // 3
Array.prototype; // [1,2,3]
// 需要将Array.prototype设置回空，否则会导致问题！
Array.prototype.length = 0;
```

这里，Function.prototype 是一个函数，RegExp.prototype 是一个正则表达式，而 Array.prototype 是一个数组。

我们可以将原型作为默认值，Function.prototype 是一个空函数，RegExp.prototype 是一个“空”的正则表达式（无任何匹配），而Array.prototype 是一个空数组。对未赋值的变量来说，它们是很好的默认值。例如：

```javascript
function isThisCool(vals,fn,rx) {
  vals = vals || Array.prototype;
  fn = fn || Function.prototype;
  rx = rx || RegExp.prototype;
  return rx.test(
  vals.map( fn ).join( "" )
  );
}
```

这种方法的一个好处是 .prototypes 已被创建并且仅创建一次。相反，如果将 []、function(){} 和 /(?:)/ 作为默认值，则每次调用 isThisCool(..) 时它们都会被创建一次（具体创建与否取决于 JavaScript 引擎，稍后它们可能会被垃圾回收），这样无疑会造成内存和 CPU 资源的浪费。

## 第4章 强制类型转换

值类型转换

>将值从一种类型转换为另一种类型通常称为类型转换（type casting），这是显式的情况；隐式的情况称为强制类型转换（coercion）。

JavaScript 中的**强制类型转换总是返回标量基本类型值**，如字符串、数字和布尔值，**不会返回对象和函数**。“封装”是为标量基本类型值封装一个相应类型的对象，但这并非严格意义上的强制类型转换。

在 JavaScript 中通常将它们统称为强制类型转换，方便理解可用“隐式强制类型转换”（implicit coercion）和“显式强制类型转换”（explicit coercion）来区分。

抽象值操作

以下是字符串、数字和布尔值之间类型转换的基本规则。

ToString

基本类型值的字符串化规则为：null 转换为 "null"，undefined 转换为 "undefined"，true转换为 "true"。数字的字符串化则遵循通用规则。

对普通对象来说，除非自行定义，否则 toString()（Object.prototype.toString()）返回内部属性 [[Class]] 的值，如 "[object Object]"。而如果对象有自己的 toString() 方法，字符串化时就会调用该方法并使用其返回值。

数组的默认 toString() 方法经过了重新定义，将所有单元字符串化以后再用 "," 连接起来：
    
    var a = [1,2,3];
    a.toString(); // "1,2,3"

toString() 可以被显式调用，或者在需要字符串化时自动调用。

工具函数 JSON.stringify(..) 在将 JSON 对象序列化为字符串时也用到了 ToString。请注意，**JSON 字符串化并非严格意义上的强制类型转换。**

对大多数简单值来说，JSON 字符串化和 toString() 的效果基本相同，只不过序列化的结果总是字符串：

```javascript
JSON.stringify( 42 ); // "42"
JSON.stringify( "42" ); // ""42"" （含有双引号的字符串）
JSON.stringify( null ); // "null"
JSON.stringify( true ); // "true"
```

undefined、function、symbol（ES6+）和包含循环引用（对象之间相互引用，形成一个无限循环）的对象都不符合 JSON结构标准，支持 JSON 的语言无法处理它们。**JSON.stringify(..) 在对象中遇到 undefined、function 和 symbol 时会自动将其忽略，在数组中则会返回 null（以保证单元位置不变）**。

如果对象中定义了 toJSON() 方法，JSON 字符串化时会首先调用该方法，然后用它的返回值来进行序列化。

如果要对含有非法 JSON 值的对象做字符串化，或者对象中的某些值无法被序列化时，就需要定义 toJSON() 方法来返回一个安全的 JSON 值。例如：

```javascript
var o = { };
var a = {
  b: 42,
  c: o,
  d: function(){}
};
// 在a中创建一个循环引用
o.e = a;
// 循环引用在这里会产生错误
// JSON.stringify( a );
// 自定义的JSON序列化
a.toJSON = function() {
  // 序列化仅包含b
  return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

toJSON() 返回的应该是一个适当的值，**可以是任何类型**，然后再由 JSON.stringify(..) 对其进行字符串化。

现在介绍几个不太为人所知但却非常有用的功能。

我们可以向 JSON.stringify(..) 传递一个**可选参数 replacer**，它可以是数组或者函数，用来指定对象序列化过程中哪些属性应该被处理，哪些应该被排除，和 toJSON() 很像。

如果**replacer 是一个数组**，那么它必须是一个字符串数组，其中包含序列化要处理的对象的属性名称，除此之外其他的属性则被忽略。

如果**replacer 是一个函数**，它会对对象本身调用一次，然后对对象中的每个属性各调用一次，每次传递两个参数，键和值。如果要忽略某个键就返回 undefined，否则返回指定的值。

```javascript
var a = {
  b: 42,
  c: "42",
  d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
  if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

JSON.string 还有一个**可选参数 space**，用来指定输出的缩进格式。space 为正整数时是指定每一级缩进的字符数，它还可以是字符串，此时最前面的十个字符被用于每一级的缩进。

ToNumber

有时我们需要将非数字值当作数字来使用，比如数学运算。

其中 true 转换为 1，false 转换为 0。undefined 转换为 NaN，null 转换为 0。

ToNumber 对字符串的处理基本遵循数字常量的相关规则 / 语法。""、"\n"（或者 " " 等其他空格组合）等空字符串被 ToNumber 强制类型转换为 0。处理失败时返回 NaN（处理数字常量失败时会产生语法错误）。不同之处是 ToNumber 对以 0 开头的十六进制数并不按十六进制处理（而是按十进制）。

对象（包括数组）会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，则再遵循以上规则将其强制转换为数字。为了将值转换为相应的基本类型值，抽象操作 ToPrimitive会首先检查该值是否有 valueOf() 方法。如果有并且返回基本类型值，就使用该值进行强制类型转换。如果没有就使用 toString()的返回值（如果存在）来进行强制类型转换。如果 valueOf() 和 toString() 均不返回基本类型值，会产生 TypeError 错误。

```javascript
var a = {
  valueOf: function(){
    return "42";
  }
};
var b = {
  toString: function(){
    return "42";
  }
};
var c = [4,2];
c.toString = function(){
  return this.join( "" ); // "42"
};

Number( a ); // 42
Number( b ); // 42
Number( c ); // 42
Number( "" ); // 0
Number( [] ); // 0
Number( [ "abc" ] ); // NaN
```

ToBoolean

假值：假值的布尔强制类型转换结果为 false。

以下这些是假值：
* undefined
* null
* false
* +0、-0 和 NaN
* ""


假值对象

假值对象看起来和普通对象并无二致（都有属性，等等），但将它们强制类型转换为布尔值时结果为 false。最常见的例子是 document.all，它是一个类数组对象，包含了页面上的所有元素，由DOM（而不是 JavaScript 引擎）提供给 JavaScript 程序使用。它以前曾是一个真正意义上的对象，布尔强制类型转换结果为 true，不过现在它是一个假值对象。

真值：真值就是假值列表之外的值。

显示强制类型转换

字符串和数字之间的显式转换

字符串和数字之间的转换是通过 String(..) 和 Number(..) 这两个内建函数（原生构造函数）来实现的，请注意**它们前面没有 new 关键字，并不创建封装对象**。

```javascript
var a = 42;
var b = String( a );
var c = "3.14";
var d = Number( c );
b; // "42"
d; // 3.14
```

String(..) 遵循前面讲过的 ToString 规则，将值转换为字符串基本类型。Number(..) 遵循前面讲过的 ToNumber 规则，将值转换为数字基本类型。

除了 String(..) 和 Number(..) 以外，还有其他方法可以实现字符串和数字之间的显式转换：

```javascript
var a = 42;
var b = a.toString();
var c = "3.14";
var d = +c;
b; // "42"
d; // 3.14
```

a.toString() 是显式的（“toString”意为“to a string”），不过其中涉及隐式转换。因为toString() 对 42 这样的基本类型值不适用，所以 JavaScript 引擎会**自动为 42 创建一个封装对象，然后对该对象调用 toString()**。这里显式转换中含有隐式转换。

+c 是 + 运算符的一元（unary）形式（即只有一个操作数）。+ 运算符显式地将 c 转换为数字，而非数字加法运算。

一元运算符 + 的另一个常见用途是将日期（Date）对象强制类型转换为数字，返回结果为Unix 时间戳，以微秒为单位（从 1970 年 1 月 1 日 00:00:00 UTC 到当前时间）：

    var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
    +d; // 1408369986000

我们常用下面的方法来获得当前的时间戳，例如：

    var timestamp = +new Date();

但不建议对日期类型使用强制类型转换，应该使用 Date.now() 来获得当前的时间戳，使用 new Date(..).getTime() 来获得指定时间的时间戳。

一个常被人忽视的地方是 ~ 运算符（即字位操作“非”）相关的强制类型转换。它首先将值强制类型转换为 32 位数字，然后执行字位操作“非”（对每一个字位进行反转（~ 返回 2 的补码））。

**~x 大致等同于 -(x+1)**。在 -(x+1) 中唯一能够得到 0（或者严格说是 -0）的 x 值是 -1。也就是说如果 x 为 -1 时，~和一些数字值在一起会返回假值 0，其他情况则返回真值。

而-1 是一个“哨位值”，哨位值是那些在各个类型中（这里是数字）被赋予了特殊含义的值。在 C 语言中我们用 -1 来代表函数执行失败，用大于等于 0 的值来代表函数执行成功。

JavaScript 中字符串的 indexOf(..) 方法也遵循这一惯例，该方法在字符串中搜索指定的子字符串，如果找到就返回子字符串所在的位置（从 0 开始），否则返回 -1。

```javascript
var a = "Hello World";

if (a.indexOf( "lo" ) >= 0) { // true
 // 找到匹配！
}
if (a.indexOf( "lo" ) != -1) { // true
 // 找到匹配！
}
if (a.indexOf( "ol" ) < 0) { // true
 // 没有找到匹配！
}
if (a.indexOf( "ol" ) == -1) { // true
 // 没有找到匹配！
}
```
\>= 0 和 == -1 这样的写法不是很好，称为“抽象渗漏”，意思是在代码中暴露了底层的实现细节，这里是指用 -1 作为失败时的返回值，这些细节应该被屏蔽掉。

~ 和 indexOf() 一起可以将结果强制类型转换（实际上仅仅是转换）为真 / 假值：

```javascript
var a = "Hello World";

~a.indexOf( "lo" ); // -4 <-- 真值!

if (~a.indexOf( "lo" )) { // true
 // 找到匹配！
}

~a.indexOf( "ol" ); // 0 <-- 假值!

!~a.indexOf( "ol" ); // true
if (!~a.indexOf( "ol" )) { // true
 // 没有找到匹配！
}
```

如果 indexOf(..) 返回 -1，~ 将其转换为假值 0，其他情况一律转换为真值。可以说~ 比 >= 0 和 == -1 更简洁。

一些开发人员**使用 \~\~ 来截除数字值的小数部分**，以为这和 Math.floor(..) 的效果一样，实际上并非如此。\~\~ 中的第一个 ~ 执行 ToInt32 并反转字位，然后第二个 ~ 再进行一次字位反转，即将所有字位反转回原值，最后得到的仍然是 ToInt32 的结果。

先它只适用于 32 位数字，更重要的是它对负数的处理与 Math.
floor(..) 不同。
    
    Math.floor( -49.6 ); // -50
    ~~-49.6; // -49

\~\~x 能将值截除为一个 32 位整数，x | 0 也可以，而且看起来还更简洁。出于对运算符优先级的考虑，我们可能更倾向于使用 \~\~x：

    ~~1E20 / 10; // 166199296
    (1E20 | 0) / 10; // 166199296

显式解析数字字符串

>解析字符串中的数字和将字符串强制类型转换为数字的返回结果都是数字。但解析和转换两者之间还是有明显的差别。

```javascript
var a = "42";
var b = "42px";

Number( a ); // 42
parseInt( a ); // 42

Number( b ); // NaN
parseInt( b ); // 42
```

解析允许字符串中含有非数字字符，解析按从左到右的顺序，如果遇到非数字字符就停止。而转换不允许出现非数字字符，否则会失败并返回 NaN。

*解析字符串中的浮点数可以使用 parseFloat(..) 函数。*

不要忘了 parseInt(..) 针对的是字符串值。向 parseInt(..) 传递数字和其他类型的参数是没有用的，比如 true、function(){...} 和 [1,2,3]。非字符串参数会首先被强制类型转换为字符串，依赖这样的隐式强制类型转换并非上策，应该避免向 parseInt(..) 传递非字符串参数。

解析非字符串

    parseInt( 1/0, 19 ); // 18

很多人想当然地以为（实际上大错特错）“如果第一个参数值为 Infinity，解析结果也应该是 Infinity”，返回 18 也太无厘头了。

而在实际的 JavaScript 代码中不会用到基数 19。它的有效数字字符范围是 0-9 和 a-i（区分大小写）。

parseInt(1/0, 19) 实际上是 parseInt("Infinity", 19)。第一个字符是 "I"，以 19 为基数时值为 18。第二个字符 "n" 不是一个有效的数字字符，解析到此为止，和 "42px" 中的 "p"一样。

此外还有一些看起来奇怪但实际上解释得通的例子：

    parseInt( 0.000008 ); // 0 ("0" 来自于 "0.000008")
    parseInt( 0.0000008 ); // 8 ("8" 来自于 "8e-7")
    parseInt( false, 16 ); // 250 ("fa" 来自于 "false")
    parseInt( parseInt, 16 ); // 15 ("f" 来自于 "function..")
    parseInt( "0x10" ); // 16
    parseInt( "103", 2 ); // 2

显示转换为布尔值

与前面的 String(..) 和 Number(..) 一样，Boolean(..)（不带 new）是显式的 ToBoolean 强制类型转换：

```javascript
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```

虽然 Boolean(..) 是显式的，但并不常用。和前面讲过的 + 类似，**一元运算符 ! 显式地将值强制类型转换为布尔值**。但是**它同时还将真值反转为假值（或者将假值反转为真值）**。

所以**显式强制类型转换为布尔值最常用的方法是 !!**，因为第二个 ! 会将结果反转回原值：

```javascript
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a; // true
!!b; // true
!!c; // true

!!d; // false
!!e; // false
!!f; // false
!!g; // false
```

在 if(..).. 这样的布尔值上下文中，如果没有使用 Boolean(..) 和 !!，就会自动隐式地进行 ToBoolean 转换。建议使用 Boolean(..) 和 !! 来进行显式转换以便让代码更清晰易读。

```javascript
var a = 42;
var b = a ? true : false;
```

三元运算符 ? : 判断 a 是否为真，如果是则将变量 b 赋值为 true，否则赋值为 false。

表面上这是一个显式的 ToBoolean 强制类型转换，因为返回结果是 true 或者 false。然而这里涉及隐式强制类型转换，因为 a 要首先被强制类型转换为布尔值才能进行条件判断。这种情况称为“显式的隐式”，有百害而无一益，我们应彻底杜绝。

隐式强制类型转换

隐式强制类型转换的作用是减少冗余，让代码更简洁。

字符串和数字之间的隐式强制类型转换

```javascript
var a = "42";
var b = "0";
var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42

var e = [1,2];
var f = [3,4];
e + f; // "1,23,4"
```

如果某个操作数是字符串或者能够通过以下步骤转换为字符串的话，+ 将进行拼接操作。

如果其中一个操作数是对象（包括数组），则首先对其调用
ToPrimitive 抽象操作，该抽象操作再调用 [[DefaultValue]]，以数字作为上下文。你或许注意到这与 ToNumber 抽象操作处理对象的方式一样。因为数组的valueOf() 操作无法得到简单基本类型值，于是它转而调用 toString()。因此上例中的两个数组变成了 "1,2" 和 "3,4"。+ 将它们拼接后返回 "1,23,4"。

简单来说就是，如果 + 的其中一个操作数是字符串（或者通过以上步骤可以得到字符串），则执行字符串拼接；否则执行数字加法。

对隐式强制类型转换来说，这意味着什么？我们可以将数字和空字符串 "" 相 + 来将其转换为字符串：
```javascript
var a = 42;
var b = a + "";
b; // "42"
```

*+ 作为数字加法操作是可互换的，即 2 + 3 等同于 3 + 2。作为字符串拼接操作则不行，但对空字符串 "" 来说，a + "" 和 "" + a 结果一样。*

a + ""（隐式）和前面的 String(a)（显式）之间有一个细微的差别需要注意。根据ToPrimitive 抽象操作规则，a + "" 会对 a 调用 valueOf() 方法，然后通过 ToString 抽象操作将返回值转换为字符串。而 String(a) 则是直接调用 ToString()。

也可以使用a - 1, a * 1 和 a /1。

布尔值到数字的隐式强制类型转换

```javascript
function onlyOne() {
  var sum = 0;
  for (var i=0; i < arguments.length; i++) {
  // 跳过假值，和处理0一样，但是避免了NaN
      if (arguments[i]) {
        sum += arguments[i];
      }
  }
  return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a ); // true
onlyOne( b, a, b, b, b ); // true
```

通过 sum += arguments[i] 中的隐式强制类型转换，将真值（true/truthy）转换为 1 并进行累加。如果有且仅有一个参数为 true，则结果为 1；否则不等于 1，sum == 1 条件不成立。

```javascript
function onlyOne() {
  var sum = 0;
  for (var i=0; i < arguments.length; i++) {
    sum += Number( !!arguments[i] );
  }
  return sum === 1;
}
```

!!arguments[i] 首先将参数转换为 true 或 false。因此非布尔值参数在这里也是可以的，比如：onlyOne("42", 0)（否则的话，字符串会执行拼接操作，这样结果就不对了）。

隐式强制类型转换为布尔值

* if (..) 语句中的条件判断表达式。
* for ( .. ; .. ; .. ) 语句中的条件判断表达式（第二个）。
* while (..) 和 do..while(..) 循环中的条件判断表达式。
* ? : 中的条件判断表达式。
* 逻辑运算符 ||（逻辑或）和 &&（逻辑与）左边的操作数（作为条件判断表达式）。

以上情况中，非布尔值会被隐式强制类型转换为布尔值，遵循前面介绍过的 ToBoolean 抽象操作规则。

|| 和 &&

&& 和 || 运算符的返回值并不一定是布尔类型，而是两个操作数其中一个的值。

```javascript
var a = 42;
var b = "abc";
var c = null;
a || b; // 42
a && b; // "abc"
c || b; // "abc"
c && b; // null
```

在 C 和 PHP 中，上例的结果是 true 或 false，在 JavaScript（以及 Python 和 Ruby）中却是某个操作数的值。

|| 和 && 首先会对第一个操作数（a 和 c）执行条件判断，如果其不是布尔值（如上例）就先进行 ToBoolean 强制类型转换，然后再执行条件判断。

对于 || 来说，如果条件判断结果为 true 就返回第一个**操作数（a 和 c）的值**，如果为false 就返回第二个操作数（b）的值。

&& 则相反，如果条件判断结果为 true 就返回**第二个操作数（b）的值**，如果为 false 就返回第一个操作数（a 和 c）的值。

下面是一个十分常见的 || 的用法，也许你已经用过但并未完全理解：

```javascript
function foo(a,b) {
  a = a || "hello";
  b = b || "world";
  console.log( a + " " + b );
}

foo(); // "hello world"
foo( "yeah", "yeah!" ); // "yeah yeah!"
```

如果第一个操作数为真值，则 && 运算符“选择”第二个操作数作为返回值，这也叫作“守护运算符”（guard operator），即前面的表达式为后面的表达式“把关”：

```javascript
function foo() {
  console.log( a );
}

var a = 42;

a && foo(); // 42
```

符号的强制类型转换

ES6 允许从符号到字符串的显式强制类型转换，然而隐式强制类型转换会产生错误。例如：


```javascript
var s1 = Symbol( "cool" );
String( s1 ); // "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + ""; // TypeError
```

符号不能够被强制类型转换为数字（显式和隐式都会产生错误），但可以被强制类型转换为布尔值（显式和隐式结果都是 true）。

宽松相等和严格相等

== 允许在相等比较中进行强制类型转换，而 === 不允许。

*宽松不相等（loose not-equality）!= 就是 == 的相反值，!== 同理。*

定如果两个值的类型相同，就仅比较它们是否相等。例如，42
等于 42，"abc" 等于 "abc"。有几个非常规的情况需要注意。
* NaN 不等于 NaN。
* +0 等于 -0。

两个对象（包括函数和数组）指向同一个值时即视为相等，不发生强制类型转换。

== 在比较两个不同类型的值时会发生隐式强制类型转换，会将其中之
一或两者都转换为相同的类型后再进行比较。

1. 字符串和数字之间的相等比较  
  如果 Type(x) 是数字，Type(y) 是字符串，则返回 x==ToNumber(y) 的结果。  
  如果 Type(x) 是字符串，Type(y) 是数字，则返回ToNumber(x) == y 的结果。
2. 其他类型和布尔类型之间的相等比较  
  如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y 的结果；  
  如果 Type(y) 是布尔类型，则返回 x == ToNumber(y) 的结果。
3. null 和 undefined 之间的相等比较
  如果 x 为 null，y 为 undefined，则结果为 true。
  如果 x 为 undefined，y 为 null，则结果为 true。  
     
   这也就是说在 == 中 null 和 undefined 是一回事,可以相互进行隐式强制类型转换。

    ```javascript
    var a = null;
    var b;
    
    a == b; // true
    a == null; // true
    b == null; // true
    
    a == false; // false
    b == false; // false
    a == ""; // false
    b == ""; // false
    a == 0; // false
    b == 0; // false
    ```
   通过这种方式将 null 和 undefined作为等价值来处理比好。例如：
    ```javascript
    var a = doSomething();
    if (a == null) {  // 在保证安全性的同时还能提高代码读性
     // ..
    }
    ``` 
    
   下面是显式的做法，其中不涉及强制类型转换。
    
     ```javascript
     var a = doSomething();
     if (a === undefined || a === null) {
      // ..
     }
     ```

4. 对象和非对象之间的相等比较  
  如果 Type(x) 是字符串或数字，Type(y) 是对象，则返回 x == ToPrimitive(y) 的结果；  
  如果 Type(x) 是对象，Type(y) 是字符串或数字，则返回 ToPromitive(x) == y 的结果。
      
    “打开”封装对象（如 new String("abc")），返回其中的基本数据类型值（"abc"）。== 中的 ToPromitive 强制类型转换也会发生这样的情况：

     ```javascript
     var a = "abc";
     var b = Object( a ); // 和new String( a )一样
     
     a === b; // false
     a == b; // true
     ```
    但有一些值不这样，原因是 == 算法中其他优先级更高的规则。例如：

     ```javascript
     var a = null;
     var b = Object( a ); // 和Object()一样
     a == b; // false
     
     var c = undefined;
     var d = Object( c ); // 和Object()一样
     c == d; // false
     
     var e = NaN;
     var f = Object( e ); // 和new Number( e )一样
     e == f; // false
     ```
    因为没有对应的封装对象，所以 null 和 undefined 不能够被封装（boxed），Object(null)和 Object() 均返回一个常规对象。NaN 能够被封装为数字封装对象，但拆封之后 NaN == NaN 返回false，因为 NaN 不等于 NaN。

比较少见的情况

1. 返回其他数字

   ```javascript
   Number.prototype.valueOf = function() {
     return 3;
   };

   new Number( 2 ) == 3; // true
   ```

   而 Number(2) 涉及 ToPrimitive 强制类型转换，因此会调用 valueOf()。

2. 假值的相等比较

   * "0" == false;  // true -- 晕！
   * false == 0;    // true -- 晕！
   * false == "";   // true -- 晕！
   * false == [];   // true -- 晕！
   * "" == 0;       // true -- 晕！
   * "" == [];      // true -- 晕！
   * 0 == [];       // true -- 晕！

   如果两边的值中有 true 或者 false，千万不要使用 ==。  
   如果两边的值中有 []、"" 或者 0，尽量不要使用 ==。

抽象关系比较

比较双方首先调用 ToPrimitive，如果结果出现非字符串，就根据 ToNumber 规则将双方强制类型转换为数字来进行比较。

```javascript
var a = [ 42 ];
var b = [ "43" ];
a < b; // true
b < a; // false
```

如果比较双方都是字符串，则按字母顺序来进行比较：

```javascript
var a = [ "42" ];
var b = [ "043" ];
a < b; // false
```

同理：

```javascript
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];
a < b; // false
```

再比如：

```javascript
var a = { b: 42 };
var b = { b: 43 };
a < b; // ??
```

结果还是 false，因为 a 是 [object Object]，b 也是 [object Object]，所以按照字母顺序a < b 并不成立。

下面的例子就有些奇怪了：
```javascript
var a = { b: 42 };
var b = { b: 43 };

a < b; // false
a == b; // false
a > b; // false

a <= b; // true
a >= b; // true
```

根据规范 a <= b 被处理为 b < a，然后将结果反转。因为 b < a 的结果是 false，所以 a <= b 的结果是 true。实际上JavaScript 中 <= 是“不大于”的意思（即 !(a > b)，处理为 !(b < a)）。同理 a >= b 处理为 b <= a。

## 第5章 语法

语句和表达式

```javascript
var a = 3 * 6;
var b = a;
b;
```

这里，3 * 6 是一个表达式（结果为 18）。第二行的 a 也是一个表达式，第三行的 b 也是。表达式 a 和 b 的结果值都是 18。这三行代码都是包含表达式的语句。  
var a = 3 * 6 和 var b = a 称为“**声明语句**”（declaration statement），因为它们声明了变量（还可以为其赋值）。  
a = 3 * 6 和 b = a（不带 var）叫作“**赋值表达式**”。  
第三行代码中只有一个表达式 b，同时它也是一个语句（虽然没有太大意义）。这样的情况通常叫作“**表达式语句**”（expression statement）。

语句的结果值

很多人不知道，语句都有一个结果值。获得结果值最直接的方法是在浏览器开发控制台中输入语句，默认情况下控制台会显示所执行的最后一条语句的结果值。

如果在控制台中输入 var a = 42 会得到结果值 undefined，而非 42。

```javascript
var b;
if (true) {
  b = 4 + 38;
}
```

在控制台 /REPL 中输入以上代码应该会显示 42，即最后一个语句 / 表达式 b = 4 + 38 的结果值。  
换句话说，代码块的结果值就如同一个隐式的返回，即返回最后一个语句的结果值。

但下面这样的代码无法运行：

```javascript
var a, b;

a = if (true) {
  b = 4 + 38;
};
```

因为语法不允许我们获得语句的结果值并将其赋值给另一个变量（至少目前不行）。

可以使用万恶的 eval(..)（又读作“evil”）来获得结果值：

```javascript
var a, b;
a = eval( "if (true) { b = 4 + 38; }" ); // 不要使用eval！
a; // 42
```

表达式的副作用

最常见的有副作用（也可能没有）的表达式是函数调用：

```javascript
function foo() {
  a = a + 1;
}

var a = 1;

foo(); // 结果值：undefined。副作用：a的值被改变
```

另一个有趣的例子是 = 赋值运算符。例如：

```javascript
var a;
a = 42; // 42
a; // 42
```

a = 42 中的 = 运算符看起来没有副作用，**实际上它的结果值是 42，它的副作用是将 42 赋值给 a。**

上下文规则

1. 大括号
   
   下面两种情况会用到大括号 { .. }
   
   (1) 对象常量
   
   (2) 标签
   
   ```javascript
   // 假定函数bar()已经定义
   {
     foo: bar()
   }
   ```
   
   很多开发人员以为这里的 { .. } 只是一个孤立的对象常量，没有赋值。事实上不是这样。{ .. } 在这里只是一个普通的代码块。语法上是完全合法的，特别是和 let（块作用域声明）在一起时非常有用。
   
   但 foo: bar() 这样奇怪的语法为什么也合法呢？
   这里涉及 JavaScript 中一个不太为人知（也不建议使用）的特性,叫作“标签语句”（labeled statement）。foo 是语句 bar() 的标签（后面没有 ;）。
   
   JavaScript 通过标签跳转能够实现 goto 的部分功能。continue和 break 语句都可以带一个标签，因此能够像 goto 那样进行跳转：
   
   ```javascript
   // 标签为foo的循环
   foo: for (var i=0; i<4; i++) {
     for (var j=0; j<4; j++) {
     // 如果j和i相等，继续外层循环
       if (j == i) {
       // 跳转到foo的下一个循环
         continue foo;
       }
       // 跳过奇数结果
       if ((j * i) % 2 == 1) {
       // 继续内层循环（没有标签的）
         continue;
       }
       console.log( i, j );
     }
   } 
   // 1 0
   // 2 0
   // 2 1
   // 3 0
   // 3 2
   ```
   
   contine foo 并不是指“跳转到标签 foo 所在位置继续执行”，而是“执行foo 循环的下一轮循环”。所以这里的 foo 并非 goto。
   另外有break foo ，它不是指“跳转到标签 foo 所在位置继续执行”，而是“跳出标签foo 所在的循环 / 代码块，继续执行后面的代码”。因此它并非传统意义上的goto。
   
   JSON 被普遍认为是 JavaScript 语言的一个真子集，{"a":42} 这样的 JSON 字符串会被当作合法的 JavaScript 代码（请注意 JSON 属性名必须使用双引号！）。其实不是！如果在控制台中输入{"a":42} 会报错。因为标签不允许使用双引号，所以 "a" 并不是一个合法的标签，因此后面不能带 :。

2. 代码块

   ```javascript
   [] + {}; // "[object Object]"
   {} + []; // 0
   ```
   表面上看 + 运算符根据第一个操作数（[] 或 {}）的不同会产生不同的结果，实则不然。
   
   第一行代码中，{} 出现在 + 运算符表达式中，因此它被当作一个值（空对象）来处理。 [] 会被强制类型转换为 ""，而 {} 会被强制类型转换为 "[object Object]"。
   
   但在第二行代码中，{} 被当作一个独立的空代码块（不执行任何操作）。代码块结尾不需要分号，所以这里不存在语法上的问题。最后 + [] 将 [] 显式强制类型转换为 0。

3. 对象解构

```javascript
function getData() {
  // ..
  return {
    a: 42,
    b: "foo"
  };
}

var { a, b } = getData();
console.log( a, b ); // 42 "foo"
```

{ a , b } = .. 就是 ES6 中的解构赋值。

{ .. } 还可以用作函数命名参数（named function argument）的对象解构（object destructuring），方便隐式地用对象属性赋值：

```javascript
function foo({ a, b, c }) {
  // 不再需要这样:
  // var a = obj.a, b = obj.b, c = obj.c
  console.log( a, b, c );
}

foo( {
  c: [1,2,3],
  a: 42,
  b: "foo"
} ); // 42 "foo" [1, 2, 3]
```

4. else if 和可选代码块

```javascript
if (a) {
  // ..
}
else if (b) {
  // ..
}
else {
  // ..
}
```

事实上 JavaScript 没有 else if，但 if 和 else 只包含单条语句的时候可以省略代码块的{ }。比如：

    if (a) doSomething( a );

else 也是如此，所以我们经常用到的 else if 实际上是这样的：

```javascript
if (a) {
 // ..
}
else {
  if (b) {
  // ..
  }
  else {
  // ..
  }
}
```

if (b) { .. } else { .. } 实际上是跟在 else 后面的一个单独的语句，所以带不带 { } 都可以。

运算符优先级

*具体优先级请百度*

1. 短路

   对 && 和 || 来说，如果从左边的操作数能够得出结果，就可以忽略   右边的操作数。我们将这种现象称为“短路”（即执行最短路径）。
   
   “短路”很方便，也很常用，如：
   
   ```javascript
   function doSomething(opts) {
     if (opts && opts.cool) {
       // ..
     }
   }
   
   // || 运算符也一样：
   function doSomething(opts) {
     if (opts.cache || primeCache()) {
     // ..
     }
   }
   ```

2. 关联

   运算符的关联（associativity）不是从左到右就是从右到左，这取决于组合（grouping）是从左开始还是从右开始。
   
   一些运算符在左关联和右关联时的表现截然不同。比如 ? :（即三元运算符或者条件运算符）：
   
       a ? b : c ? d : e;
   
   ? : 是右关联，它的组合顺序是以下哪一种呢？
       
       a ? b : (c ? d : e)
       (a ? b : c) ? d : e
   
   答案是 a ? b : (c ? d : e)。和 && 以及 || 运算符不同，右关联在这里会影响返回结果，因为 (a ? b : c) ? d : e 对有些值（并非所有值）的处理方式会有所不同。
   
   可知? : 是右关联，并且它的组合方式会影响返回结果。另一个右关联（组合）的例子是 = 运算符。
   
   **如果运算符优先级 / 关联规则能够令代码更为简洁，就使用运算符优先级 / 关联规则；而如果 ( ) 有助于提高代码可读性，就使用 (    )。**

自动分号

有时 JavaScript 会自动为代码行补上缺失的分号，即自动分号插入（Automatic SemicolonInsertion，ASI）。

*ASI 只在换行符处起作用，而不会在代码行的中间插入分号。*

语法规定 do..while 循环后面必须带 ;，而 while 和 for 循环后则不需要。大多数开发人员都不记得这一点，此时 ASI 就会自动补上分号。其他涉及 ASI 的情况是 break、continue、return 和 yield（ES6）等关键字。

函数参数

在 ES6 中，如果参数被省略或者值为 undefined，则取该参数的默认值：

```javascript
function foo( a = 42, b = a + 1 ) {
  console.log( a, b );
}

foo(); // 42 43
foo( undefined ); // 42 43
foo( 5 ); // 5 6
foo( void 0, 7 ); // 42 7
foo( null ); // null 1
```

然而某些情况下，它们之间还是有区别的：


```javascript
function foo( a = 42, b = a + 1 ) {
  console.log(
    arguments.length, a, b,
    arguments[0], arguments[1]
  );
}

foo(); // 0 42 43 undefined undefined
foo( 10 ); // 1 10 11 10 undefined
foo( 10, undefined ); // 2 10 11 10 undefined
foo( 10, null ); // 2 10 null 10 null
```

虽然参数 a 和 b 都有默认值，但是函数不带参数时，arguments 数组为空。

ES6 参数默认值会导致 arguments 数组和相对应的命名参数之间出现偏差，ES5 也会出现这种情况：

```javascript
function foo(a) {
  a = 42;
  console.log( arguments[0] );
}

foo( 2 ); // 42 (linked)
foo(); // undefined (not linked)
```

向函数传递参数时，arguments 数组中的对应单元会和命名参数建立关联（linkage）以得到相同的值。相反，不传递参数就不会建立关联。

try..finally

finally 中的代码总是会在 try 之后执行，如果有 catch 的话则在 catch 之后执行。也可以将 finally 中的代码看作一个回调函数，即无论出现什么情况最后一定会被调用。

如果 try 中有 return 语句会出现什么情况呢？ return 会返回一个值，那么调用该函数并得到返回值的代码是在 finally 之前还是之后执行呢？
```javascript
function foo() {
  try {
    return 42;
  }
  finally {
    console.log( "Hello" );
  }
  console.log( "never runs" );
}
console.log( foo() );
// Hello
// 42
```
这里 return 42 先执行，并将 foo() 函数的返回值设置为 42。然后 try 执行完毕，接着执行 finally。最后 foo() 函数执行完毕，console.log(..) 显示返回值。  
*continue 和 break 等控制语句也是如此。*

如果 finally 中抛出异常（无论是有意还是无意），函数就会在此处终止。如果此前 try 中已经有 return 设置了返回值，则该值会被丢弃。

另外finally 中的 return 会覆盖 try 和 catch 中 return 的返回值。

通常来说，在函数中省略 return 的结果和 return; 及 return undefined; 是一样的，但是在 finally 中省略 return 则会返回前面的 return 设定的返回值。

switch

```javascript
switch (a) {
  case 2:
     // 执行一些代码
     break;
  case 42:
     // 执行另外一些代码
     break;
  default:
     // 执行缺省代码
}
```
这里 a 与 case 表达式逐一进行比较。如果匹配就执行该 case 中的代码，直到 break 或者switch 代码块结束。

这看似并无特别之处，但其中存在一些不太为人所知的陷阱。

首先，a 和 case 表达式的匹配算法与 ===（参见第 4 章）相同。通常 case 语句中的 switch都是简单值，所以这并没有问题。

然而，有时可能会需要通过强制类型转换来进行相等比较，这时就需要做一些特殊处理：

```javascript
var a = "42";
switch (true) {
  case a == 10:
    console.log( "10 or '10'" );
    break;
  case a == 42;
    console.log( "42 or '42'" );
    break;
  default:
    // 永远执行不到这里
}
// 42 or '42'
```
除简单值以外，case 中还可以出现各种表达式，它会将表达式的结果值和 true 进行比较。因为 a == 42 的结果为 true，所以条件成立。

尽管可以使用 ==，但 switch 中 true 和 true 之间仍然是严格相等比较。即如果 case 表达式的结果为真值，但不是严格意义上的 true，则条件不成立。

var a = "hello world";
var b = 10;
switch (true) {
 case (a || b == 10):
 // 永远执行不到这里
 break;
 default:
 console.log( "Oops" );
}
// Oops
因为 (a || b == 10) 的结果是 "hello world" 而非 true，所以严格相等比较不成立。此时可以通过强制表达式返回 true 或 false，如 case !!(a || b == 10):。
