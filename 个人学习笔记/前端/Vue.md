# （前言）前端三要素
## 1. HTML（结构）

## 2. CSS(表现) ：
CSS层叠样式表是一门标记语言，并不是编程语言，因此不可以自定义变量，不可以引用等，换句话说就是不具备任何语法支持
它主要的缺陷如下：
- 语法不够强大：无法嵌套书写，导致模块化开发中需要重复的书写选择器；
- 没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护；

### CSS预处理器：
纯手写CSS导致了我们在工作中无端增加了许多工作量。为了解决这个问题，前端开发人员会使用一种称之为【CSS 预处理器】 的工具，
提供 CSS 缺失的样式层复用机制、减少冗余代码，提高样式代码的可维护性。大大提高了前端在样式上的开发效率

CSS 预处理器定义了一种新的语言，其基本思想是，用一种专门的编程语言，为 CSS 增加了一些编程的
特性，将 CSS 作为目标生成文件，然后开发者就只要使用这种语言进行 CSS 的编码工作。转化成通俗易
懂的话来说就是“用一种专门的编程语言，进行 Web 页面样式设计，再通过编译器转化为正常的 CSS 文
件，以供项目使用”。

### 常用的 CSS 预处理器:
- SASS：基于 Ruby，通过服务端处理，功能强大。解析效率高。需要学习 Ruby 语言，上手难度高
于 LESS。
- LESS：基于 NodeJS，通过客户端处理，使用简单。功能比 SASS 简单，解析效率也低于 SASS，但
在实际开发中足够了，所以我们后台人员如果需要的话，建议使用 LESS。

## 3. JavaScript(行为):
### Native 原生 JS 开发
原生 JS 开发，也就是让我们按照 【ECMAScript】 标准的开发方式，简称是 ES，特点是所有浏览器都
支持。ES 标准已发布如下版本：
- ES3
- ES4（内部，未正式发布）
- ES5（全浏览器支持）
- ES6（常用，当前主流版本：webpack打包成为ES5支持！）
- ES7
- ES8
- ES9（草案阶段）

从 ES6 开始每年发布一个版本，以年份作为名称，区别就是逐步增加新特性。

### JavaScript框架
- jQuery库：最熟知的JavaScript 库，优点是简化了 DOM 操作，缺点是 DOM 操作太频繁，影响前端性能；使用它仅仅是为了兼容 IE6、7、8；
- Angular:由Google收购，由一群Java程序员开发，将后台的MVC模式了前端，并增加了模
       块化开发的理念，与微软合作，采用 TypeScript 语法开发；对后台程序员友好，对前端程序员不太友
       好；最大的缺点是版本迭代不合理（如：1代 -> 2代，除了名字，基本就是两个东西；已推出了
       Angular6）
- React：Facebook 出品，一款高性能的 JS 前端框架；特点是提出了新概念 【虚拟 DOM】 用于减少真实 DOM
      操作，在内存中模拟 DOM 操作，有效的提升了前端渲染效率；缺点是使用复杂，因为需要额外学习一
      门 【JSX】 语言；
- Vue：一款渐进式 JavaScript 框架，所谓渐进式就是逐步实现新特性的意思，如实现模块化开发、路由、状态
    管理等新特性。其特点是综合了 Angular（模块化） 和 React（虚拟 DOM） 的优点；
- Axios:前端通信框架；因为 Vue 的边界很明确，就是为了处理 DOM，所以并不具备通信能力，此时就需要额
      外使用一个通信框架与服务器交互；当然也可以直接选择使用jQuery提供的AJAX通信功能；
      

# （前言）MVVM架构
MVVM（Model-View-ViewModel）是一种软件架构设计模式，是一种简化用户界面的事件驱动编程方式。
- Model：模型层，在这里表示 JavaScript 对象
-View：视图层，在这里表示 DOM（HTML 操作的元素）
- ViewModel：连接视图和数据的中间件，Vue.js 就是 MVVM 中的 ViewModel 层的实现者
- MVVM 源自于经典的 MVC（Model-View-Controller）模式MVVM的核心是ViewModel层，ViewModel 是由前端开发人员组织生成和维护的视图数据层。

在 MVVM 架构中，是不允许 数据和视图直接通信的，只能通过ViewModel来通信，而 ViewModel
就是定义了一个 Observer 观察者。
ViewModel 能够观察到数据的变化，并对视图对应的内容进行更新
ViewModel 能够监听到视图的变化，并能够通知数据发生改变

在这一层，前端开发者对从后端获取的Model 数据进行转换处理，做二次封装，以生成符合 View 层使用预期的视图数据模型。

作用：该层向上与视图层进行双向数据绑定，向下与 Model 层通过接口请求进行数据交互

# 初识Vue
## Vue简介
Vue 是一套用于构建用户界面的渐进式框架，发布于 2014 年 2 月。与其它
大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上
手，还便于与第三方库（如：vue-router，vue-resource，vuex）或既有项目整合。
Vue.js 就是一个 MVVM 的实现者，他的核心就是实现了 DOM 监听和数据绑定

## Vue的优点
- 轻量级，体积小，压缩后有只有 20+kb （对比Angular压缩后56kb+，React压缩后44kb+）
- 移动优先。更适合移动端，比如移动端的 Touch 事件
- 易上手，学习曲线平稳，文档齐全
- 吸取了 Angular（模块化）和 React（虚拟 DOM）的长处，并拥有自己独特的功能，如：计算属性
- 开源，社区活跃度高

## 见识下Vue的数据绑定功能

{{}}插值表达式：
{{}}插值表达式里面 只能写表达式,不能写语句

1. 引入外部资源换
```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
```
2. 创建Vue实例
```html
<script type="text/javascript">
    var element = new Vue({
        el: "#helloVue",
        data:{
            message:"hello ! Vue!"
        }
    })
</script>
```

3. 实现数据绑定到页面
```html
<div id = "helloVue">
    {{message}}
</div>
```

测试：浏览器控制台输出element.message = "" ,可以实现数据的动态绑定

# Vue基础语法
## v-bind
如果要将模型数据绑定在html属性中 则使用 v-bind 指令,此时title中显示的是模型数据
```html

<div id="app">
    <h1 v-bind:title="message1">鼠标悬停几秒钟查看此处动态绑定的提示信息！</h1>
    <!-- v-bind 指令的简写为冒号（:） -->
    <h1 :title="message2">鼠标悬停几秒钟查看此处动态绑定的提示信息！</h1>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            message1: "123",
            message2:"234"
        }
    })
</script>
```
## v-if v-else-if v-else系列
```html
<div id="case" >
    <h1 v-if="result==='a'">A</h1>
    <h1 v-else-if="result==='b'">B</h1>
    <h1 v-else-if="result==='c'">C</h1>
    <h1 v-else="result==='d'">D</h1>
</div>
<script>
    var vm = new Vue({
        el:"#case",
        data:{
            result:"b"
        }
    })
</script>
```

## v-for
```html
<div id="vFor">
    <li v-for="item in items">
        {{item.message}}
    </li>
</div>

<script>
    var vm = new Vue({
        el: "#vFor",
        data: {
            items: [
                {message: '张立群'},
                {message: 'Larry'},
                {message: 'Susan'}
            ]
        }
    });
</script>
```

测试 ：在控制台输入 vm.items.push({message: 'Tom'}) ，尝试追加一条数据，你会发现
浏览器中显示的内容会增加一条 Tom
## v-on事件
```html
<!--使用v-on绑定click事件-->
<!--指定sayHi的方法-->
<div id="testVue">
<!--    v-on绑定click事件，并制定sayHi方法-->
    <button v-on:click="sayHi">按钮1</button>
<!--    v-on绑定click事件的简写形式：用@click代替v-on:-->
    <button @click="sayHi">按钮2</button>
</div>

<script type="text/javascript">
    new Vue({
        el: "#testVue",
        data:{
            message:'Hello World'
        },
        // 一定要用methods，否则无法识别
        methods:{
          sayHi: function (event) {
              alert(this.message);
          }
        }
    })
</script>
```
通过观察发现new一个vue的结构基本的模板就是 ：new Vue({}),
在里面追加键值对形式的元素，其中 el: 是绑定id或者class属性的相当于操作dom元素。
data: 相当于显示页面内容的，其值要用花括号{}。
methods: 绑定各种事件 （注：一定要用methods，否则无法识别）



## v-model（双向数据绑定）
Vue.js的精髓之处即为数据的双向绑定：数据变化随之视图变化，视图变化随之数据变化

1. v-model 可以绑定text输入框中的value
```html
<div id="testVue">
    <input type="text" v-bind:value="searchMap.message">
    <input type="text" v-model:value="searchMap.message">
    <P>
        输入的数据是：{{searchMap.message}}
    </P>
</div>
<script type="text/javascript">
    new Vue({
        el:"#testVue",
        data:{
            searchMap:{
                message:'123'
            }
        }
    });
</script>
```
2. v-model 可以绑定select下拉框中的value
```html
<body>
<div id="app">
    <select v-model:value="value">
        <option disabled value="" >请选择</option>
        <option>a</option>
        <option>b</option>
        <option>c</option>
        <option>d</option>
        <option>e</option>
        <option>f</option>
    </select> <br/>
    <span>选中的值为：{{value}}</span>
</div>
<script>
    var vm = new Vue({
        el:"#app",
        data:{
            value:'a'
        }
    })
</script>
</body>
```
3. v-model可以绑定checkbox单、复选框中的checked值
```html
<div id="app">
    单复选框： <input type="checkbox" id="checkbox" v-model:checked="checked">
    &nbsp;&nbsp; <label for="checkbox">{{ checked }}</label>
</div>
<script>
    var vm = new Vue({
        el: '#app',
        data: {
            checked: false,
        }
    })
</script>
```

```html
<div id="app">
    <input type="checkbox" id="tom" value="tom" v-model:checked="checkedNames" >
    <label for="tom">tom</label><br/>
    <input type="checkbox" id="susan" value="susan" v-model:checked="checkedNames" >
    <label for="tom">susan</label><br/>
    <input type="checkbox" id="alie" value="alex" v-model:checked="checkedNames" >
    <label for="tom">alex</label><br/>
    <input type="checkbox" id="limy" value="limy" v-model:checked="checkedNames" >
    <label for="tom">limy</label><br/>
    <input type="checkbox" id="test1" value="test1" v-model:checked="checkedNames" >
    <label for="tom">测试</label><br/>
    <span>选中的值为：{{checkedNames}}</span>

</div>
<script>
    var vm = new Vue({
        el:"#app",
        data:{
            checkedNames:[]
        }
    })
</script>
```


3. v-model可以绑定radio单选按钮中的checked值
```html
<div id="testVue">
<!--    label标签for绑定的是id-->
    <label for="boy">男</label>
    <input type="radio"  id="boy" value="男" v-model:checked="checked">
    <label for="girl">女</label>
    <input type="radio"  id="girl" value="女" v-model:checked="checked"><br>
    <span>选中的值为： {{checked}}</span>
</div>

<script type="text/javascript">
    new Vue({
       el:"#testVue",
       data:{
           checked:''
       }
    });
</script>
```
v-bind和v-model的区别
前面的都只能显示vue对象中数据, 页面数据的变化不会影响vue对象中的数据，而v-model绑定的数据,页面数据的改变,vue对象中的数据也会跟着改变,这非常关键

# 组件
使用Vue . component()方法注册组件。
注意：在实际开发中，我们并不会用以下方式开发组件，以下方法只是为了让大家理解什么是组件。

```html
<body>
<div id="testVue">
<!--自定义一个标签-->
    <my-defined-text></my-defined-text>
</div>
<script type="text/javascript">
    Vue.component("my-defined-text",
        {
            //注意：template内要用单引号
            template:'<input type="text" value="这是一个自定义组件的内容">'
        });

    //不要忘了Vue的实例化
    new Vue({
        el:"#testVue"
    });
</script>
```

```html
<div id="testVue" >
    <ul>
        <my-defined-text v-for="item in items" v-bind:item="item">
        </my-defined-text>
    </ul>
</div>

<script type="text/javascript">
    Vue.component("my-defined-text",{
        props:['item'],
        template:'<li>{{item}}</li>'
        });

    new Vue({
        el:"#testVue",
        data:{
            items:["Larry","Betty","Susan"]
        }
    });
</script>
```
使用 props 属性用来传递参数到组件

# 计算属性
计算属性是一个属性，它有计算功能，它用来计算出来的结果保存在属性中，这里的计算功能是个函数；
它是一个能够将计算结果缓存起来的属性（将行为转化成了静态的属性）可以想象为缓存！
```html
<div id="app">
    <p>调用当前时间的方法：{{currentTime1()}}</p>
    <p>调用当前时间的计算属性：{{currentTime2}}</p>
</div>

<script>
    var vm = new Vue({
        el: "#app",
        data: {
            message: 'Hello Vue'
        },
        methods: {
            currentTime1: function () {
                return Date.now();
            }
        },
        computed: {
            //在computed计算属性中，currentTime2是一个属性，不是一个方法！
            currentTime2: function () {
                this.message;
                return Date.now();
            }
        }
    });
```
测试：在控制台多次输入vm . currentTime1()，发现其时间戳是变化的；
在控制台多次输入vm.currentTime2，发现其值是不变的，很好的证明了计算的时间戳值存到了缓存中
，如果我们输入vm.message="12312"，修改了computed计算属性中的数据，就会重新进行计算，输入vm.currentTime2
发现结果也变了。

调用方法时，每次都需要进行计算，既然有计算过程则必定产生系统开销，那如果这个结果是不经常变
化的呢？此时就可以考虑将这个结果缓存起来，采用计算属性可以很方便的做到这一点，计算属性的主
要特性就是为了将不经常变化的计算结果进行缓存，以节约我们的系统开销;