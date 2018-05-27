# 整理的一些问题

## 1、使用typeof bar ===“object”来确定bar是否是一个对象时有什么潜在的缺陷？这个陷阱如何避免？

- null在javascript中会被认为是一个对象

``` js
bar = null
console.log(typeof bar) //"object"
```

- 数组也会被认为是一个对象

``` js
bar = []
console.log(typeof bar) //"object"
```

- 函数不会被认为是一个对象

``` js
bar = function (){}
console.log(typeof bar) //"function"
```

所以可以用如下代码避免

``` js
//ES5
console.log((bar !== null) && (typeof bar === "object") && (toString.call(bar) !== "[object Array]"))

//ES6
console.log((bar !== null) && (typeof bar === "object") && !Array.isArray(bar))
```

## 2、下面的代码将输出到控制台的是什么，为什么？

``` js
(function(){
  var a = b = 3;
})();
console.log("a defined? " + (typeof a !== 'undefined'));
console.log("b defined? " + (typeof b !== 'undefined'));
```

上述代码相当于

``` js
b = 3 // b 相当于全局变量
var a = b
```

因此在上面封闭函数之外,b作为全局变量仍在作用域内,而a则为undefined，所以代码输出如下:

``` js
a defined? false
b defined? true
```

- **注意:** 上述代码在严格模式下会报"ReferenceError",因为b未定义

## 3、下面的代码将输出到控制台的是什么？，为什么？

```js
var myObject = {
    foo: "bar",
    func: function() {
        var self = this;
        console.log("outer func:  this.foo = " + this.foo);
        console.log("outer func:  self.foo = " + self.foo);
        (function() {
            console.log("inner func:  this.foo = " + this.foo);
            console.log("inner func:  self.foo = " + self.foo);
        }());
    }
};
myObject.func();
```

在内部自执行函数中,this不再指向myObject,所以上述代码输出为:

```js
outer func:  this.foo = bar
outer func:  self.foo = bar
inner func:  this.foo = undefined
inner func:  self.foo = bar
```

## 4、考虑下面的两个函数。他们都会返回同样的值吗？为什么或者为什么不？

```js
function foo1()
{
  return {
      bar: "hello"
  };
}
 
function foo2()
{
  return
  {
      bar: "hello"
  };
}

```

javascript中遇到包含return语句的行（没有其他内容）时，会在return语句之后立即自动插入分号,所以foo2相当于:

```js
function foo2()
{
  return ;
  {
      bar: "hello"
  };
}
```

所以两个函数不会返回相同的值

```js
console.log(foo1()) // {bar:"hello"}
console.log(foo2()) // undefined
```

## 5、什么是NaN？它的类型是什么？如何可靠地测试一个值是否等于NaN

NaN属性表示“不是数字”的值。这个特殊值是由于一个操作数是非数字的（例如“abc”/ 4）或者因为操作的结果是非数字而无法执行的。NaN有一些令人惊讶的特征:

- 虽然NaN的意思是“不是数字”，但它的类型是数字:

```js
console.log(typeof NaN) // number
```

- NaN相比任何事情,甚至本身都是false

```js
console.log(NaN === NaN) //false
```

测试一个值是否为NaN,javascript提供了一个内置函数isNaN(),但是他不可靠:

```js
isNaN(NaN);       // true
isNaN(undefined); // true
isNaN({});        // true
isNaN(true);      // false
isNaN(null);      // false
isNaN(37);        // false
// strings
isNaN('37');      // false: "37" is converted to the number 37 which is not NaN
isNaN('37.37');   // false: "37.37" is converted to the number 37.37 which is not NaN
isNaN("37,5");    // true
isNaN('123ABC');  // true:  parseInt("123ABC") is 123 but Number("123ABC") is NaN
isNaN('');        // false: the empty string is converted to 0 which is not NaN
isNaN(' ');       // false: a string with spaces is converted to 0 which is not NaN
// dates
isNaN(new Date());                // false
isNaN(new Date().toString());     // true
```

一个比较好的做法是使用 value !== value,只有当value为NaN时上述表达式为才true,另外ES6提供了Number.isNaN()函数,他的与旧的isNaN()不同,也更加可靠

## 6、下面的代码将输出到控制台，为什么？

```js
var arr1 = "john".split('');
var arr2 = arr1.reverse();
var arr3 = "jones".split('');
arr2.push(arr3);
console.log("array 1: length=" + arr1.length + " last=" + arr1.slice(-1));
console.log("array 2: length=" + arr2.length + " last=" + arr2.slice(-1));
```

- 数组的reverse方法不仅以相反的顺序返回数组,还颠倒了数组本身的顺序

- reverse方法返回对数组本身的引用

所以上述代码实际上是这样:

```js
var arr1 = "john".split('');
arr1 = arr1.reverse();
var arr2 = arr1; //arr2其实是arr1的引用,两者指向同一对象
var arr3 = "jones".split('');
```

所以arr2 和 arr1一样,输出结果如下:

```js
"array 1: length=5 last=j,o,n,e,s"
"array 2: length=5 last=j,o,n,e,s"
```

## 7、下面的代码将输出到控制台，为什么？

```js
console.log(1 +  "2" + "2");
console.log(1 +  +"2" + "2");
console.log(1 +  -"1" + "2");
console.log(+"1" +  "1" + "2");
console.log( "A" - "B" + "2");
console.log( "A" - "B" + 2);

"122"
"32"
"02"
"112"
"NaN2"
NaN
```

- 示例2中第二个+为一元运算符,所以javascript会将"2"类型转为数字,然后将+应用于他,相当于视其为正整数2,所以1++"2",实为1+2

- 示例3原理同上,-"1",实际为负数1,1+-"1",实际为1-1
- 示例5 6中"A"-"B"的值均为NaN,后面+"2"为字符串拼接,而+2为数值相加,NaN任何数值操作均为NaN

## 8、如果数组列表太大，以下递归代码将导致堆栈溢出。你如何解决这个问题，仍然保留递归模式？

```js
var list = readHugeList();
var nextListItem = function() {
    var item = list.pop();
 
    if (item) {
        // process the list item...
        nextListItem();
    }
};
```

通过修改nextListItem函数可以避免潜在的堆栈溢出，如下所示:

```js
var list = readHugeList();
var nextListItem = function() {
    var item = list.pop();
    if (item) {
        // process the list item...
        setTimeout( nextListItem, 0);
    }
};
```

堆栈溢出被消除，因为事件循环处理递归，而不是调用堆栈。当nextListItem运行时，如果item不为null，则将超时函数（nextListItem）推送到事件队列，并且函数退出，从而使调用堆栈清零。当事件队列运行超时事件时，将处理下一个项目，并设置一个计时器以再次调用nextListItem。因此，该方法从头到尾不经过直接递归调用即可处理，因此调用堆栈保持清晰，无论迭代次数如何。

## 9、以下代码的输出是什么？解释你的答案。

```js
var a = {},
    b = {key:'b'},
    c = {key:'c'};

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```

代码将输出456。设置对象属性时，JavaScript会隐式地将参数值串联起来。在这种情况下，由于b和c都是对象，它们都将被转换为“[object Object]”。因此，a [b]和a [c]都等价于[“[object Object]”]，并且可以互换使用。因此，设置或引用[c]与设置或引用[b]完全相同。

## 10、考虑下面的代码片段。控制台的输出是什么，为什么？

```js
(function(x) {
    return (function(y) {
        console.log(x);
    })(2)
})(1);
```

输出将为1，即使x的值从未在内部函数中设置。原因如下：

闭包是一个函数，以及创建闭包时在范围内的所有变量或函数。在JavaScript中，闭包被实现为“内部函数”;即在另一功能的主体内定义的功能。闭包的一个重要特征是内部函数仍然可以访问外部函数的变量。

因此，在这个例子中，因为x没有在内部函数中定义，所以在外部函数的作用域中搜索一个定义的变量x，该变量的值为1。

## 11、下面这段代码将输出什么？

```js
var length = 10;
function fn() {
    console.log(this.length);
}

var obj = {
  length: 5,
  method: function(fn) {
    fn();
    arguments[0]();
  }
};

obj.method(fn, 1);
```

```js
//10
//2
```

- this的指向问题可以参考[这篇文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

- 当function作为参数传递的时候，其中的this指向的是全局作用域，所以第一次输出为10；

- argumnets是一个类数组对象，元素即为所传参数的列表，所以arguments[0]为fn，此时fn中的this指向了调用他的对象，即为argumnets，因此this.length即为arguments.length，所以第二次输出为2；

## 12、下面的代码。输出是什么，为什么？

```js
(function () {
    try {
        throw new Error();
    } catch (x) {
        var x = 1, y = 2;
        console.log(x);
    }
    console.log(x);
    console.log(y);
})();
```

javascript中 var语句会被提升到它所属的全局或者函数作用域的顶部。具体可以参考[这篇文章](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var)。所以上述代码相当于如下：

```js
(function () {
    var x, y; //外部的x,y
    try {
        throw new Error();
    } catch (x //catch 内部的x 只在catch内才能访问) {
        x = 1; //catch 内部的x 
        y = 2; //外部的y
        console.log(x);//catch 内部的x
    }
    console.log(x);//外部的x
    console.log(y);//外部的y
})();

/*
1
undefined
2
*／
```

## 13、下面代码输出什么

```js
console.log(1 < 2 < 3);
console.log(3 > 2 > 1);
```

- 第一行从左到右 1 < 2 返回true，true < 3会转化成1 < 3，所以第一行输出true

- 第二行从左到右 3 > 2 返回true，true > 1会转化成1 > 1，所以第二行输出false

- 归根到底还是javascript类型转换的问题，如下同理： 

```js
console.log(1 < 2 < 2); //true
console.log(1 > 2 < 1); //true
```

## 14、考虑如下代码

```js
var a = [1, 2, 3];

//下面代码会不会崩溃？
a[10] = 99;

//输出什么？
console.log(a[6]);
```

- 不会崩溃，javascript引擎会使得3-9变成空插槽，所以a[10] = 99后，数组a变成如下形式：

```js
[1, 2, 3, empty × 7, 99]
```

- 所以a[6] 输出undefined，但是索引3-9依然为空而不是被undefined填充，这会在某些情况下导致些重要的差别。例如使用map的时候，空插槽依然为空，但是undefined插槽会利用传入的函数重新映射

```js
var b = [undefined];
b[2] = 1;
console.log(b);             // (3) [undefined, empty × 1, 1]
console.log(b.map(e => 7)); // (3) [7,         empty × 1, 7]
```

## 15、typeof undefined == typeof NULL的值是什么

值为true，首先javascript区分大小写，所以NULL不同于null，而是被当作一个未被定义的变量，因此两边都是 "undefined"

## 16、javascript中如何检测变量是否是String类型

```js
typeof(obj) === 'string'
typeof obj === 'string'
obj.constructor === String
```

## 17、javascript去除字符串空格

- 使用replace正则匹配

```js
str = str.replace(/\s*/g,"")
\\去除所有空格
str = str.replace(/^\s*|\s*$/g,"");
\\去除两头空格
str = str.replace( /^\s*/, “”);
\\去除左空格
str = str.replace(/(\s*$)/g, "");
\\去除右空格
```

- str.trim()方法，但是其只能去除字符串的左右空格，无法去除中间的空格。同理还有str.trimLeft(),str.trimRight(),str.trimStart(),str.trimEnd()
