## 作用域
作用域是一套查找变量位置和查找范围的规则,他的作用是协助引擎找到需要的变量

如下面代码
```javascript
var value = 1;
function foo() {
  console.log(value)
}
foo() //1
```
执行foo时，foo的作用域块中，找不到变量value, 这时候根据作用域定义的规则，向上查找，foo的上级的全局作用域，于是在这里找到了value并返回。

同时作用域规定查找变量终点是全局作用域。不管能不能找到，都要给引擎一个交代

## 作用域和引擎如何配合查找变量
作用域会协助引擎查找需要的变量，那么引擎怎样查询的变量呢
引擎有两种查询形式，LHS 和 RHS

L 表示左侧， R表示右侧

引用《你不知道的JavaScript》的话
>当变量出现在赋值操作的左侧进行LHS查询,出现在右侧进行RHS查询

LHS查找表示为赋值操作寻找一个容器,而RHS查找只想知道容器里面装了什么
通俗点讲 赋值操作 就是 “=”号
```
var a = 1;
```
我们需要对变量a进行**赋值操作**，显而易见的变量a也在 “=”号左边, 对 a 就是LHS查询,赋值操作之外的查询，就是RHS查询,比如
```javascript
//对value是LHS查询，对a,b是RHS查询
var value = a + b;
//这里对value是RHS查询
console.log(value)
```
### 为什么要区分LHS和RHS查询？
因为在**变量还没有声明**的情况下,两种查找返回的结果不一样，

LHS查询从内部作用域向上找直到全局作用域都没找到变量的情况下,引擎会帮你在全局作用域创建一个变量，作为赋值的容器(严格模式下除外,而es6 module模块化自动开启严格模式)

RHS查询在找不到的情况下会，引擎会抛出ReferenceError错误

这就是上面说的`不管能不能找到，都要给引擎一个交代`

```
function foo() {
  console.log(a)
}
foo();
```



## 词法作用域
词法作用域(lexical scoping),也叫静态作用域。 顾名思义静态作用域是相对静止的。 函数的作用域在定义时候就决定了。

我们来看下面代码
```javascript
var value = 1;
function bar() {
  var value = 2;
  function foo() {
      console.log(value);
  }
  foo();
}
bar(); //2
```
我们看到，foo会输出一个2，因为它定义在了bar的内部，所以foo的作用域是 foo => bar => 全局作用域。向上查找的过程中，引擎在bar作用域块发现了变量value，于是返回给了引擎，查找结束。

我们再来看下面一段代码
```javascript
var value = 1;

function foo() {
    console.log(value);
}
function bar() {
    var value = 2;
    //在bar内部调用
    foo();
}

bar(); // 1
```
我们看到，就算foo是在bar内部调用的，但foo会输出一个1而不是2，原因就是 JavaScript采用词法作用域(静态作用域),**函数的作用域在定义时候就决定了**。 foo定义在了全局作用域中,他的作用域是 foo => 全局作用域。

#### 动态作用域
在编程语言界中，作用域有两种工作模型, 刚刚介绍的词法作用域(静态作用域) 是最普遍的。 相对的另外一种就是动态作用域了, 如果JavaScript是采用动态作用域的，那么上一个例子输出的就是2，

动态作用域应用的语言比较少,有Bash脚本，Perl。 这里不作过多讲解

参考：你不知道的JavaScript（上卷）
