## 前言
首先，我们看下面代码, 我们提前执行了`foo`和`bar`.可以看到输出结果不一样， 用函数声明创建的foo能正常输出，用函数表达式创建的bar报错了
```javascript

foo();// 1
bar();// Uncaught TypeError: bar is not a function

//函数声明 创建
function foo() {
	console.log(1)
}

//函数表达式 创建
var bar = function() {
	console.log(2)
}

```
如果有人问为什么时，你肯定能张口就来: “函数声明创建的函数会提升到作用域顶部,可以先使用后定义” 这其实没有什么错误，便于大家理解。但如果要准确的解答这个问题，就要深入到本章节的内容了
## 执行上下文
当 JavaScript 代码执行一段可执行代码(executable code)时，会创建对应的执行上下文(execution context)。

对于每个执行上下文，都有三个重要属性：

* 变量对象(Variable object，VO)
* 作用域链(Scope chain)
* this

今天重点讲讲创建变量对象的过程。

## 变量对象
变量对象是与 执行上下文 相关的数据作用域，对象里面储存了我们保存的变量和函数声明。

而执行上下文 分为两种
* 全局上下文的变量对象
* 函数上下文的变量对象,由于和全局的稍有不同，它有另一个名称,称之为活动对象(activation object, AO)以示区分

```JavaScript
var a = 1;
function foo(i) {
  var b = 2;
}
foo(111)
```

在上述代码中，执行栈中有两个上下文，全局上下文和函数foo上下文,我们用数组的模拟一下
```javascript
stach = [
  globalContext,
  fooContext
]
```
对于全局上下文来说，VO(变量对象)大概是这样的
```javascript
globalContext.VO = {
  a:undefined,
  foo: reference to function foo(){},
}
```
而对于函数 `foo`来说，AO(活动对象)是这样的
```javascript
fooContext.AO {
  arguments: {
    0:111,
    length: 1
  },
  a: undefined,
  i: undefined,
  b: undefined,
}
```
## 执行过程
执行上下文的代码会分成两个阶段进行处理，分析和执行，我们也可以叫做

1.进入执行上下文

2.代码执行
### 进入执行上下文

当进入执行上下文时，这时候还没有执行代码，JS引擎会找出需要提升的变量和函数，并且给他们提前在内存中开辟好空间，函数的话会将整个函数存入内存中，变量只声明并且赋值为 undefined，所以在第二个阶段，也就是代码执行阶段，我们可以直接提前使用。
举个例子:
```JavaScript
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};

  b = 3;

}

foo(1);
```
在第一阶段进入执行上下文后，这时候的 AO 是：
```JavaScript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c(){},
    d: undefined
}
```
在第二阶段 代码执行 ，这时候的AO是
```JavaScript
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```
这就是一开始例子说的函数表达式会报错，而函数声明正常执行的原因了。



另外如果变量名称跟已经声明的形式参数或函数相同，则函数声明的权重高。
请看下面代码,
```JavaScript
console.log(foo);//会打印函数

function foo(){
    console.log("foo");
}

var foo = 1;
console.log(foo);//1
```
可以看到，提前使用的时候,函数声明的权重高,所以第一个console打印了函数。但这样不会觉得很乱吗？

## 遵循先定义再使用原则
在我们的潜在意识中，代码从上到下按顺序执行更直观，也不容易出错的。变量提升反而会把人绕晕。**所以谷歌建议的代码规范中也倡导使用函数表达式创建函数，而在ES6中，使用let，const等定义变量或者函数，也不会提升, 会存在 暂时性死区**

函数的表现
```JavaScript
函数提升
bar(); // 2
function bar() {
	console.log(2)
}

函数表达式
foo(); // 不管是var还是let定义，都会报错
var foo = function() {
	console.log(1)
};
```

变量的表现
```JavaScript
//es5, 变量提升
console.log(foo)//undefined
var foo = 1;

//es6 暂时性死区
console.log(bar)//Uncaught ReferenceError: bar is not defined
let bar = 2;
```
所以es6的**暂时性死区**其实就是提前调用变量或函数,在进入上下文时，**不会**在AO中提前开辟内存。而是直接抛出ReferenceError错误


**遵循先定义再使用原则能一定程度降低项目出错的可能,增加项目可维护性**

## 补充
**arguments**

如果你有留意上面的示例代码，会发现有一个叫arguments的对象。引用《JavaScript权威指南》
>调用函数时，会为其创建一个Arguments对象，并自动初始化局部变量arguments，指代该Arguments对象。所有作为参数传入的值都会成为Arguments对象的数组元素。
