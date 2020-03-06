### JavaScript组成

1. ECMAScript
2. DOM
3. BOM

### JavaScript原始值的类型

1. Number
2. Boolean 
3. String
4. undefined
5. null

### 显示类型转换用到的几个方法

* Number\(mix\)
* parseInt\(string, radix\)
* parseFloat\(string\)
* toString\(radix\)
* String\(radix\)
* Boolean\(\)

如何将二进制1000转换成16进制打印

```javascript
var num = 10000;
var test = parseInt(num, 2);
console(test.toString(16));
```

### 会发生隐士类型转换的运算

* isNaN\(\) ---&gt; Number\(\)
* ++/-- +/-\(一元正负\)
* * （1 + ‘1’）
* -\*/% ---&gt;Number\(\)
* && \|\| !
* &lt; &gt; &lt;= &gt;=
* ==  !=

### 不会发生类型转换的运算符

* ===
* !==

### 关于函数的几点说明

* 函数声明：

```javascript
function test() {}
```

* 函数表达式

```javascript
var test = function a() {} //命名函数表达式
var test2 = function (){} // 匿名函数表达式
```

* 函数默认都是不定参数， 如果没有形参接收多余参数， 会有一个默认的对象arguments来接收

### 函数的预编译

> imply global暗示全局变量：即任何变量，如果没有经过声明就赋值，此变量属于全局对象所有。
>
> 一切声明的全局变量，都是window的属性

1. 创建AO对象
2. 找形参和变量声明， 将变量和形参的名作为AO的属性名，值为undefined
3. 将实参的值域形参统一
4. 在函数体里面找函数声明， 值赋予函数体

```js
// 练习
function fn(a) {
    console.log(a);
    var a = 123;
    console.log(a);
    function a() {}
    console.log(a);
    var b = function() {}
    console.log(b);
    function d() {}
}
fn(1);
// 预编译发生在函数执行的前一刻


// 1 创建AO对象
var AO = {}; 
// 2 
var AO = {
    a: undefined;
    b: undefined;
};
// 3
AO.a = 1;
// 4
AO.a = function a() {}

// 开始执行
console.log(a); // 打印AO.a            function a() {}
var a = 123;    // 复制AO.a = 123 
console.log(a); // 打印AO.a            123
function a() {} // 变量声明，不做处理
console.log(a); // 打印AO.a            123
var b = function(){} // 复制AO.b = function(){}
console.log(b); // 打印AO.b            function b() {}
```

### 一个函数预编译的面试题

```js
// 写出打印结果
global = 100;
function fn() {
    console.log(global);
    global = 200;
    console.log(global);
    var global = 300;
}
fn();
var global;

///////// 结题步骤
//1.鉴于预编译发生在函数执行前的一刻， 而且默认的作用域为window（GO）
// 创建GO全局window对象
var GO = {};
// 2
var GO = {
    global: undefined;
    fn: undefined;
}
// 3 没有形参实参， 刺不略过
// 4
GO.fn = function() {...}
// 5 开始执行代码
global = 200; // GO.global = 200;
function fn() {...} // 声明函数不执行；
// fn();函数执行前， 进行函数fn的预编译过程
// 5.1 创建AO
var AO = {}
// 5.2 
var AO = {
    global: undefined;
}
// 5.3 省略
// 5.4 省略
// 执行fn函数
console.log(global); // 打印AO.global        undefined
global = 200;        // 复制AO.global = 200
console.log(global); // 打印AO.global        200
```



