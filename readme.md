# 你不知道的JavaScript 阅读笔记

## 相关
- 可以在这里下载本书第一部分“作用域和闭包”随附的资料（代码示例、练习题等）：
http://bit.ly/1c8HEWF。
- 可以在这里下载本书第二部分“this 和对象原型”随附的资料（代码示例、练习题等）：
http://bit.ly/ydkjs-this-code

## 作用域是什么
### 1.编译原理
传统语言中，三个步骤

__分词/词法分析（Tokenizing/Lexing）:__ 这个过程会将由字符组成的字符串分解成（对编程语言来说）有意义的代码块，这些代
码块被称为词法单元（token）。例如，考虑程序 var a = 2;。这段程序通常会被分解成
为下面这些词法单元：var、a、=、2 、;。

__解析/语法分析（Parsing）:__ 这个过程是将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法
结构的树。这个树被称为“抽象语法树”（Abstract Syntax Tree，AST）。

__代码生成：__ 将AST 转换为可执行代码的过程称被称为代码生成，包括分配内存等

比起那些编译过程只有三个步骤的语言的编译器，JavaScript 引擎要复杂得多，任何JavaScript 代码片段在执行前都要进行编译（通常就在执行前）

### 2.理解作用域

- __引擎__&emsp;从头到尾负责整个JavaScript 程序的编译及执行过程。
- __编译器__&emsp;引擎的好朋友之一，负责语法分析及代码生成等脏活累活（详见前一节的内容）。
- __作用域__&emsp;引擎的另一位好朋友，负责收集并维护由所有声明的标识符（变量）组成的一系列查
询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。

#### 2.1工作流程
_举例：`var a = 2`_
1. 遇到var a，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的
集合中。如果是，编译器会忽略该声明，继续进行编译；否则它会要求作用域在当前作
用域的集合中声明一个新的变量，并命名为a。
2. 接下来编译器会为引擎生成运行时所需的代码，这些代码被用来处理a = 2 这个赋值
操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫作a 的
变量。如果是，引擎就会使用这个变量；如果否，引擎会继续查找该变量（查看1.3
节）。

总结：变量的赋值操作会执行两个动作，首先__编译器__会在当前作用域中声明一个变量（如
果之前没有声明过），然后在运行时__引擎__会在作用域中查找该变量，如果能够找到就会对
它赋值

3.
 ```
function foo(a) {
  console.log( a ); // 2
}
foo( 2 );
```

让我们把上面这段代码的处理过程想象成一段对话，这段对话可能是下面这样的。

引擎：我说作用域，我需要为 foo 进行RHS引用。你见过它吗？

作用域：别说，我还真见过，编译器那小子刚刚声明了它。它是一个函数，给你。

引擎：哥们太够意思了！好吧，我来执行一下 foo 。

引擎：作用域，还有个事儿。我需要为 a 进行LHS引用，这个你见过吗？

作用域：这个也见过，编译器最近把它声名为 foo 的一个形式参数了，拿去吧。

引擎：大恩不言谢，你总是这么棒。现在我要把 2 赋值给 a 。

引擎：哥们，不好意思又来打扰你。我要为 console 进行RHS引用，你见过它吗？

作用域：咱俩谁跟谁啊，再说我就是干这个。这个我也有， console 是个内置对象。
给你。

引擎：么么哒。我得看看这里面是不是有 log(..) 。太好了，找到了，是一个函数。

引擎：哥们，能帮我再找一下对 a 的RHS引用吗？虽然我记得它，但想再确认一次。

作用域：放心吧，这个变量没有变动过，拿走，不谢。

引擎：真棒。我来把 a 的值，也就是 2 ，传递进 log(..)

4. 作用域嵌套

引擎会一层一层往上找，直到全局

### 3. 异常
- 如果 RHS 查询在所有嵌套的作用域中遍寻不到所需的变量，引擎就会抛出 ReferenceError
异常。

- 当引擎执行 LHS 查询时，如果在顶层（全局作用域）中也无法找到目标变量，
全局作用域中就会创建一个具有该名称的变量，并将其返还给引擎，前提是程序运行在非
“严格模式”下。
在
严格模式中 LHS 查询失败时，并不会创建并返回一个全局变量，引擎会抛出同 RHS 查询
失败时类似的 ReferenceError 异常。

- 如果 RHS 查询找到了一个变量，但是你尝试对这个变量的值进行不合理的操作，
比如试图对一个非函数类型的值进行函数调用，或着引用 null 或 undefined 类型的值中的
属性，那么引擎会抛出另外一种类型的异常，叫作 TypeError 。

## 词法作用域
- 作用域共有两种主要的工作模型。第一种是最为普遍的，被大多数编程语言所采用的__词法
作用域__，我们会对这种作用域进行深入讨论。
另外一种叫作__动态作用域__，仍有一些编程语
言在使用（比如 Bash 脚本、Perl 中的一些模式等）。

### 1. 词法阶段
大部分标准语言编译器的第一个工作阶段叫作词法化（也叫单词化）。
词法作用域就是定义在词法阶段的作用域（由你在写
代码时将变量和块作用域写在哪里来决定的）。
#### 1.1 查找
作用域查找始终从运行时所处的最内部作用域开始，逐级向外或者说向上进行，直到遇见
第一个匹配的标识符为止。

### 2. 欺骗词法

JavaScript 中有两种机制来实现这个目的.

_欺骗词法作用域会导致性能下降_

_另外一个不推荐使用 eval(..) 和 with 的原因是会被严格模式所影响（限
制）。 with 被完全禁止，而在保留核心功能的前提下，间接或非安全地使用
eval(..) 也被禁止了。_

#### 2.1 eval
```
function foo(str, a) {
eval( str ); // 欺骗！
console.log( a, b );
}
var b = 2;
foo( "var b = 3;", 1 ); // 1, 3
```
eval(..) 调用中的 "var b = 3;" 这段代码会被当作本来就在那里一样来处理。在严格模式的程序中， eval(..) 在运行时有其自己的词法作用域

#### 2.2 with
```
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

可以这样理解，当我们传递 o1 给 with 时， with 所声明的作用域是 o1 ，而这个作用域中含
有一个同 o1.a 属性相符的标识符。但当我们将 o2 作为作用域时，其中并没有 a 标识符，
因此进行了正常的 LHS 标识符查找（查看第 1 章）。

o2 的作用域、 foo(..) 的作用域和全局作用域中都没有找到标识符 a ，因此当 a＝2 执行
时，自动创建了一个全局变量（因为是非严格模式）。

## 函数作用域和块作用域
