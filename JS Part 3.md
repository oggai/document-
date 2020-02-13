# you don't know JS 学习笔记

## 第一章 类型

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

## 第二章 值

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

## 第三章 原生函数
54