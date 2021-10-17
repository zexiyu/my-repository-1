# 一、ECMAScript 6 简介

ECMAScript 6.0（以下简称 ES6）是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布了。它的目标，是使得 JavaScript 语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

## 1、ECMAScript 和 JavaScript 的关系

一个常见的问题是，ECMAScript 和 JavaScript 到底是什么关系？

要讲清楚这个问题，需要回顾历史。1996 年 11 月，JavaScript 的创造者 Netscape 公司，决定将 JavaScript 提交给标准化组织 ECMA，希望这种语言能够成为国际标准。次年，ECMA 发布 262 号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为 ECMAScript，这个版本就是 1.0 版。

因此，ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现（另外的 ECMAScript 方言还有 Jscript 和 ActionScript）

## 2、ES6 与 ECMAScript 2015 的关系

ECMAScript 2015（简称 ES2015）这个词，也是经常可以看到的。它与 ES6 是什么关系呢？

2011 年，ECMAScript 5.1 版发布后，就开始制定 6.0 版了。因此，ES6 这个词的原意，就是指 JavaScript 语言的下一个版本。

ES6 的第一个版本，在 2015 年 6 月发布，正式名称是《ECMAScript 2015 标准》（简称 ES2015）。

2016 年 6 月，小幅修订的《ECMAScript 2016 标准》（简称 ES2016）如期发布，这个版本可以看作是 ES6.1 版，因为两者的差异非常小，基本上是同一个标准。根据计划，2017 年 6 月发布 ES2017 标准。

因此，ES6 既是一个历史名词，也是一个泛指，含义是 5.1 版以后的 JavaScript 的下一代标准，涵盖了 ES2015、ES2016、ES2017 等等，而 ES2015 则是正式名称，特指该年发布的正式版本的语言标准。本书中提到 ES6 的地方，一般是指 ES2015 标准，但有时也是泛指“下一代 JavaScript 语言”。

 

# 二、基本语法

ES标准中不包含 DOM 和 BOM的定义，只涵盖基本数据类型、关键字、语句、运算符、内建对象、内建函数等通用语法。

本部分只学习前端开发中ES6的最少必要知识，方便后面项目开发中对代码的理解。



## 1、let声明变量 

```javascript
// var 声明的变量没有局部作用域

// let 声明的变量  有局部作用域

{

var a = 0

let b = 1

}

console.log(a)  // 0

console.log(b)  // ReferenceError: b is not defined

 

// var 可以声明多次

// let 只能声明一次

var m = 1

var m = 2

let n = 3

let n = 4

console.log(m)  // 2

console.log(n)  // Identifier 'n' has already been declared

 

// var 会变量提升

// let 不存在变量提升

console.log(x)  //undefined

var x = "apple"



console.log(y)  //ReferenceError: y is not defined

let y = "banana"

 
```



## 2、const声明常量（只读变量)

```javascript
// 1、声明之后不允许改变   

const PI = "3.1415926"

PI = 3  // TypeError: Assignment to constant variable.
 

// 2、一但声明必须初始化，否则会报错

const MY_AGE  // SyntaxError: Missing initializer in const declaration


```

 

## 3、解构赋值

解构赋值是对赋值运算符的扩展。

他是一种针对数组或者对象进行模式匹配，然后对其中的变量进行赋值。

在代码书写上简洁且易读，语义更加清晰明了；也方便了复杂对象中数据字段获取。

```javascript
//1、数组解构

// 传统

let a = 1, b = 2, c = 3

console.log(a, b, c)

// ES6

let [x, y, z] = [1, 2, 3]

console.log(x, y, z)

 

//2、对象解构

let user = {name: 'Helen', age: 18}

// 传统

let name1 = user.name

let age1 = user.age

console.log(name1, age1)

// ES6

let { name, age } =  user//注意：结构的变量必须是user中的属性

console.log(name, age)

 
```

 



## 4、模板字符串  

模板字符串相当于加强版的字符串，用反引号 `,除了作为普通字符串，还可以用来定义多行字符串，还可以在字符串中加入变量和表达式。

```javascript
// 1、多行字符串

let string1 =  `Hey,

can you stop angry now?`

console.log(string1)

// Hey,can you stop angry now?

 
// 2、字符串插入变量和表达式。变量名写在 ${} 中，${} 中可以放入 JavaScript 表达式。

let name = "Mike"

let age = 27

let info = `My Name is ${name},I am ${age+1} years old next year.`

console.log(info)

// My Name is Mike,I am 28 years old next year.

 

// 3、字符串中调用函数

function f(){

   return "have fun!"

}

let string2 = `Game start,${f()}`

console.log(string2);  // Game start,have fun!


```

 

## 5、声明对象简写

```javascript
const age = 12

const name = "Amy"



// 传统

const person1 = {age: age, name: name}

console.log(person1)



// ES6

const person2 = {age, name}

console.log(person2) //{age: 12, name: "Amy"}
```



 

## 6、定义方法简写

 ```javascript
// 传统
const person1 = {
   sayHi:function(){
	     console.log("Hi")
   }
}
person1.sayHi();//"Hi"


// ES6
const person2 = {
   sayHi(){
      console.log("Hi")
   }
}
person2.sayHi()  //"Hi"
 ```



 

## 7、对象拓展运算符

拓展运算符（...）用于取出参数对象所有可遍历属性然后拷贝到当前对象。

 ```javascript
// 1、拷贝对象

let person1 = {name: "Amy", age: 15}

let someone = { ...person1 }

console.log(someone)  //{name: "Amy", age: 15}

 

// 2、合并对象

let age = {age: 15}

let name = {name: "Amy"}

let person2 = {...age, ...name}

console.log(person2)  //{age: 15, name: "Amy"}
 ```





 

## 8、函数的默认参数

```javascript
function showInfo(name, age = 17) {

   console.log(name + "," + age)

}

// 只有在未传递参数，或者参数为 undefined 时，才会使用默认参数

// null 值被认为是有效的值传递。

showInfo("Amy", 18)  // Amy,18

showInfo("Amy", "")  // Amy,

showInfo("Amy", null)  // Amy, null

showInfo("Amy")   // Amy,17

showInfo("Amy", undefined) // Amy,17.
```



 



## 9、不定参数



不定参数用来表示不确定参数个数，形如，...变量名，由...加上一个具名参数标识符组成。具名参数只能放在参数列表的最后，并且有且只有一个不定参数。

```javascript
function f(...values) {
   console.log(values.length)
}

f(1, 2)    //2

f(1, 2, 3, 4)  //4
```



 

## 10、箭头函数

箭头函数提供了一种更加简洁的函数书写方式。基本语法是：

参数 => 函数体

```javascript
// 传统
var f1 = function(a){
   return a
}
console.log(f1(1))

// ES6
var f2 = a => a
console.log(f2(1))
 
/*当箭头函数没有参数或者有多个参数，要用 () 括起来。 
	当箭头函数函数体有多行语句，用 {} 包裹起来，表示代码块
	当只有一行语句，并且需要返回结果时，可以省略 {} , 结果会自动返回。*/

var f3 = (a,b) => {
   let result = a+b
   return result
}
console.log(f3(6,2))  // 8

// 前面代码相当于：

var f4 = (a,b) => a+b
```



## 11、Promise

在JavaScript的世界中，所有代码都是单线程执行的。由于这个“缺陷”，导致JavaScript的所有网络操作，浏览器事件，都必须是异步执行。

异步执行可以用回调函数实现：

**例1、定时器**

 ```javascript
// 1、timeout的定时器功能使用了回调函数

console.log('before setTimeout()')

setTimeout(()=>{

   console.log('Done')

}, 1000) // 1秒钟后调用callback函数

console.log('after setTimeout()')  
 ```





**例2、ajax**

mock/user.json

```json
{
   "id": "1",
   "username":"helen",
   "age":18
}
```

 



html页面引入jquery.js

 

<script src="jquery.min.js"></script>

 

 // 2、ajax功能使用了回调函数

```javascript
$.get('mock/user.json', (data)=>{
   console.log(data)
})
```



例3、1秒后执行ajax操作

```javascript
 // 1、timeout的定时器功能使用了回调函数
console.log('before setTimeout()')
setTimeout(() => {
   console.log('Done')
   // 2、ajax功能使用了回调函数

   $.get('mock/user.json', (data) => {
	     console.log(data)
   })

}, 1000); // 1秒钟后调用callback函数
console.log('after setTimeout()')
```

 

**例4、1秒后获取用户数据，然后根据用户id获取用户登录日志**

```javascript
mock/login-log-1.json
{
   "items":
   [
     {
       "date":"2018-12-01",
       "ip":"10.10.10.10"
     },
     {
       "date":"2018-12-01",
       "ip":"10.10.10.10"
     }
   ]
}
回调函数嵌套的噩梦


// 1、1秒后获取用户数据

console.log('before setTimeout()')
setTimeout(() => {
   console.log('Done')

   // 2、获取用户数据
   $.get('mock/user.json', (data) => {

     console.log(data)

     // 3、获取当前用户的登录日志

     $.get(`mock/login-log-${data.id}.json`, (data) => {

       console.log(data)
     })
   })
}, 1000) // 1秒钟后调用callback函数
console.log('after setTimeout()')
```



 

Promise是异步编程的一种解决方案。从语法上说，Promise 是一个对象，从它可以获取异步操作的响应消息。

古人云：“君子一诺千金”，这种“承诺将来会执行”的对象在JavaScript中称为Promise对象。

 ```javascript
<script src="jquery.min.js"></script>
<script>
   var p1 = new Promise((resolve, reject)=>{
     setTimeout(()=>{
       resolve()//成功后交给resolve去执行回调的内容
     }, 1000)
   }) 
   p1.then(()=>{//成功
     console.log('Done')

     //还有后续动作，所以接着返回promise
     return new Promise((resolve, reject)=>{
       $.get(`mock/user.json`, (data) => {
         resolve(data)
       })
     })
   })
     .then((data)=>{//成功
     console.log(data)
     $.get(`mock/login-log-${data.id}.json`, (data) => {
       console.log(data)
     })
   })
</script>

有错误处理的promise案例

<script src="jquery.min.js"></script>
<script>
   var p1 = new Promise((resolve, reject)=>{
     setTimeout(()=>{
       resolve()//成功后交给resolve去执行回调的内容
     }, 1000)
   })  
   p1.then(()=>{//成功
     console.log('Done')

     //还有后续动作，所以接着返回promise
     return new Promise((resolve, reject)=>{
       $.ajax({
         url: 'mock/user.json',
         type: 'get',
         success(data){
           resolve(data)//成功后交给resolve去执行回调的内容
         },
         error(){
           reject()//失败后交给reject去处理失败
         }
       })
     })
   })
   .then((data)=>{//成功
     console.log(data)
     $.get(`mock/login-log-${data.id}.json`, (data) => {
       console.log(data)
     })
   }, ()=>{//处理失败：then方法的第二个参数是失败的回调
     console.log('出错啦！')
   })
</script>

也可以使用catch处理失败

.then((data)=>{//成功
})
.catch(()=>{//处理失败：then方法的第二个参数是失败的回调
   console.log('出错啦！')
})
 ```

 

***\*12、模块化\****

随着网站逐渐变成"互联网应用程序"，嵌入网页的Javascript代码越来越庞大，越来越复杂。

![img](file:////private/var/folders/z2/gjbxj4mj3bv6gxrlm_9wwx1c0000gn/T/com.kingsoft.wpsoffice.mac/wps-liqunzhang/ksohtml/wpsQNbffb.jpg) 

Javascript模块化编程，已经成为一个迫切的需求。理想情况下，开发者只需要实现核心的业务逻辑，其他都可以加载别人已经写好的模块。

但是，Javascript不是一种模块化编程语言，它不支持"类"（class），更遑论"模块"（module）了。

在 ES6 前， 实现模块化使用的是 RequireJS 或者 seaJS（分别是基于 AMD 规范的模块化库， 和基于 CMD 规范的模块化库）。

ES6 引入了模块化，其设计思想是在编译时就能确定模块的依赖关系，以及输入和输出的变量。

ES6 的模块化分为导出（export） @与导入（import）两个模块。

 

创建api/user.js

 

let getList =  () => {

   console.log('获取数据列表')

}



let save =  () => {

   console.log('保存数据')

}

export { getList, save }

创建component/user.js（注意：浏览器不支持很多ES6的高级功能，我们需要使用babel将其转换成ES5，后面的课程将会介绍）

 

import { getList, save } from "../api/user.js"

console.log(getList())

console.log(save())

 