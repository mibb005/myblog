---
title: vue原理
date: 2021-09-23 15:10:18
tags: vue
---

# 前言

经过几天的研究，发现学习框架的底层技术，收获颇丰，相比只学习框架的使用要来的合算；如果工作急需，快速上手应用，掌握如何使用短期内更加高效；如果有较多的时间来系统学习，建议研究一下框架的等层技术、原理。

## Vue、React、Angular三大框架对比

### 1、Vue

Vue是尤雨溪编写的一个构建数据驱动的Web界面的库，准确来说不是一个框架，它聚焦在V（view）视图层。

它有以下的特性：

1. 轻量级的框架

2. 双向数据绑定

3. 指令

4. 插件化

优点：

简单：官方文档很清晰，比 Angular 简单易学。
快速：异步批处理方式更新 DOM。
组合：用解耦的、可复用的组件组合你的应用程序。
紧凑：~18kb min+gzip，且无依赖。
强大：表达式 无需声明依赖的可推导属性 (computed properties)。
对模块友好：可以通过 NPM、Bower 或 Duo 安装，不强迫你所有的代码都遵循 Angular 的各种规定，使用场景更加灵活。
缺点：

新生儿：Vue.js是一个新的项目，没有angular那么成熟。
影响度不是很大：google了一下，有关于Vue.js多样性或者说丰富性少于其他一些有名的库。
不支持IE8
### 2、React

React 起源于 Facebook 的内部项目，用来架设 Instagram 的网站， 并于 2013年 5 月开源。React 拥有较高的性能，代码逻辑非常简单，越来越多的人已开始关注和使用它。

它有以下的特性：

1. 声明式设计：React采用声明范式，可以轻松描述应用。

2. 高效：React通过对DOM的模拟，最大限度地减少与DOM的交互。

3. 灵活：React可以与已知的库或框架很好地配合。

优点：

速度快：在UI渲染过程中，React通过在虚拟DOM中的微操作来实现对实际DOM的局部更新。
跨浏览器兼容：虚拟DOM帮助我们解决了跨浏览器问题，它为我们提供了标准化的API，甚至在IE8中都是没问题的。
模块化：为你程序编写独立的模块化UI组件，这样当某个或某些组件出现问题是，可以方便地进行隔离。
单向数据流：Flux是一个用于在JavaScript应用中创建单向数据层的架构，它随着React视图库的开发而被Facebook概念化。
同构、纯粹的javascript：因为搜索引擎的爬虫程序依赖的是服务端响应而不是JavaScript的执行，预渲染你的应用有助于搜索引擎优化。
兼容性好：比如使用RequireJS来加载和打包，而Browserify和Webpack适用于构建大型应用。它们使得那些艰难的任务不再让人望而生畏。
缺点：

React本身只是一个V而已，并不是一个完整的框架，所以如果是大型项目想要一套完整的框架的话，基本都需要加上ReactRouter和Flux才能写大型应用。
### 3、Angular

Angular是一款优秀的前端JS框架，已经被用于Google的多款产品当中。

它有以下的特性：

1. 良好的应用程序结构

2. 双向数据绑定

3. 指令

4. HTML模板

5. 可嵌入、注入和测试

优点：

模板功能强大丰富，自带了极其丰富的angular指令。
是一个比较完善的前端框架，包含服务，模板，数据双向绑定，模块化，路由，过滤器，依赖注入等所有功能；
自定义指令，自定义指令后可以在项目中多次使用。
ng模块化比较大胆的引入了Java的一些东西（依赖注入），能够很容易的写出可复用的代码，对于敏捷开发的团队来说非常有帮助。
angularjs是互联网巨人谷歌开发，这也意味着他有一个坚实的基础和社区支持。
缺点：

angular 入门很容易 但深入后概念很多, 学习中较难理解.
文档例子非常少, 官方的文档基本只写了api, 一个例子都没有, 很多时候具体怎么用都是google来的, 或直接问misko,angular的作者.
对IE6/7 兼容不算特别好, 就是可以用jQuery自己手写代码解决一些.
指令的应用的最佳实践教程少, angular其实很灵活, 如果不看一些作者的使用原则,很容易写出 四不像的代码, 例如js中还是像jQuery的思想有很多dom操作.
DI 依赖注入 如果代码压缩需要显示声明.
通过以上相比较，您更加倾向于学习哪一个呢？

# 正题：Vue的基本原理
Vue的原理图

1、 建立虚拟DOM Tree，通过document.createDocumentFragment()，遍历指定根节点内部节点，根据{{ prop }}、v-model等规则进行compile；
2、 通过Object.defineProperty()进行数据变化拦截；
3、 截取到的数据变化，通过发布者-订阅者模式，触发Watcher，从而改变虚拟DOM中的具体数据；
4、 通过改变虚拟DOM元素值，从而改变最后渲染dom树的值，完成双向绑定

完成数据的双向绑定在于Object.defineProperty()

## Vue双向绑定的实现

### 1、简易双绑

首先，我们把注意力集中在这个属性上：Object.defineProperty。


Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。
语法：Object.defineProperty(obj, prop, descriptor)
什么叫做，定义或修改一个对象的新属性，并返回这个对象呢？
```
var obj = {};
Object.defineProperty(obj,'hello',{
  get:function(){
    //我们在这里拦截到了数据
    console.log("get方法被调用");
  },
  set:function(newValue){
    //改变数据的值，拦截下来额
    console.log("set方法被调用");
  }
});
obj.hello//输出为“get方法被调用”，输出了值。
obj.hello = 'new Hello';//输出为set方法被调用，修改了新值


通过以上方法可以看出，获取对象属性值触发get、设置对象属性值触发set，因此我们可以想象到数据模型对象的属性设置和读取可以驱动view层的数据变化，view的数据变化传递给数据模型对象，在set里面可以做很多事情。

在这基础上，我们可以做到数据的双向绑定：

    let obj = {};
    Object.defineProperty(obj, 'name', {
        set: function(newValue){
            console.log('触发setter');
            document.querySelector('.text-box').innerHTML = newValue;
            document.querySelector('.inp-text').value = newValue;
        },
        get: function(){
            console.log('触发getter');
        }
    });

    document.querySelector('.inp-text').addEventListener('keyup', function(e){
        obj.name = e.target.value;
    }, false);
```

html
```
<input class="inp-text" type="text">
<div class="text-box"></div>
```

以上只是vue的核心思想，通过对象底层属性的set和get进行数据拦截，vue的虚拟dom又是怎么实现的，且看以下分解。

### 2、虚拟DOM树

创建虚拟DOM：
var frag = document.createDocumentFragment();
view层的{{msg}}和v-model的编译规则如下：
html:

```
<div id="container">
    {{ msg }}<br>
    <input class="inp-text" type="text" v-model="inpText">
    <div class="text-box">
        <p class="show-text">{{ msg }}</p>
    </div>
</div>
```

view层做了多层嵌套，这样测试更多出现错误的可能性。

```
    var container = document.getElementById('container');
    //这里我们把vue实例中的data提取出来，更加直观
    var data = {
        msg: 'Hello world!',
        inpText: 'Input text'
    };
    var fragment = virtualDom(container, data);
    container.appendChild(fragment);

    //虚拟dom创建方法
    function virtualDom(node, data){
        let frag = document.createDocumentFragment();
        let child;
        // 遍历dom节点
        while(child = node.firstChild){
            compile(child, data);
            frag.appendChild(child);
        }
        return frag;
    }
     
    //编译规则
    function compile(node, data){
        let reg = /\{\{(.*)\}\}/g;
        if(node.nodeType === 1){ // 标签
            let attr = node.attributes;
            for(let i = 0, len = attr.length; i < len; i++){
                // console.log(attr[i].nodeName, attr[i].nodeValue);
                if(attr[i].nodeName === 'v-model'){
                    let name = attr[i].nodeValue;
                    node.value = data[name];
                }
            }
            if(node.hasChildNodes()){
                node.childNodes.forEach((item) => {
                    compile(item, data); // 递归
                });
            }
        }
        if(node.nodeType === 3){ // 文本节点
            if(reg.test(node.nodeValue)){
                let name = RegExp.$1;
                name = name.trim();
                node.nodeValue = data[name];
            }
        }
    }
```

解释：
1、通过virtualDom创建虚拟节点，将目标盒子内所有子节点添加到其内部，注意这里只是子节点；
2、子节点通过compile进行编译，a:如果节点为元素，其nodeType = 1,b:如果节点为文本，其nodeType = 3,具体可以查看详情http://www.w3school.com.cn/js...；
3、如果第二步子节点仍有子节点，通过hasChildNodes()来确认，如果有递归调用compile方法。

clipboard.png

### 3、响应式原理
核心思想：Object.defineProperty(obj, key, {set, get})

```
    function defineReact(obj, key, value){
        Object.defineProperty(obj, key, {
            set: function(newValue){
                console.log(`触发setter`);
                value = newValue;
                console.log(value);
            },
            get: function(){
                console.log(`触发getter`);
                return value;
            }
        });
    }
这里是针对data数据的属性的响应式定义，但是如何去实现vue实例vm绑定data每个属性，通过以下方法：

    function observe(obj, vm){
        Object.keys(obj).forEach((key) => {
            defineReact(vm, key, obj[key]);
        })
    }
vue的构造函数：

    function Vue(options){
        this.data = options.data;
        let id = options.el;

        observe(this.data, this); // 将每个data属相绑定到Vue的实例上this
    }
通过以上我们可以实现Vue实例绑定data属性。

如何去实现Vue，通常我们实例化Vue是这样的：

    var vm = new Vue({
        el: 'container',
        data: {
            msg: 'Hello world!',
            inpText: 'Input text'
        }
    });
    
    console.log(vm.msg); // Hello world!
    console.log(vm.inpText); // Input text
实现以上效果，我们必须在vue内部初始化虚拟Dom

     function Vue(options){
        this.data = options.data;
        let id = options.el;

        observe(this.data, this); // 将每个data属相绑定到Vue的实例上this
        
        //------------------------添加以下代码
        let container = document.getElementById(id);
        let fragment = virtualDom(container, this); // 这里通过vm对象初始化
        container.appendChild(fragment);
        
    }
```

这是我们再对Vue进行实例化，则可以看到以下页面：

clipboard.png

至此我们实现了dom的初始化，下一步我们在v-model元素添加监听事件，这样就可以通过view层的操作来修改vm对应的属性值。在compile编译的时候，可以准确的找到v-model属相元素，因此我们把监听事件添加到compile内部。

```
    function compile(node, data){
        let reg = /\{\{(.*)\}\}/g;
        if(node.nodeType === 1){ // 标签
            let attr = node.attributes;
            for(let i = 0, len = attr.length; i < len; i++){
                // console.log(attr[i].nodeName, attr[i].nodeValue);
                if(attr[i].nodeName === 'v-model'){
                    let name = attr[i].nodeValue;
                    node.value = data[name];

                    // ------------------------添加监听事件
                    node.addEventListener('keyup', function(e){
                        data[name] = e.target.value;
                    }, false);
                    // -----------------------------------
                }
            }
            if(node.hasChildNodes()){
                node.childNodes.forEach((item) => {
                    compile(item, data);
                });
            }
        }
        if(node.nodeType === 3){ // 文本节点
            if(reg.test(node.nodeValue)){
                let name = RegExp.$1;
                name = name.trim();
                node.nodeValue = data[name];
            }
        }
    }

```

这一步我们操作页面输入框，可以看到以下效果，证明监听事件添加有效。
clipboard.png

到这里我们已经实现了MVVM的，即Model -> vm -> View || View -> vm -> Model 中间桥梁就是vm实例对象。

4、观察者模式原理

观察者模式也称为发布者-订阅者模式，这样说应该会更容易理解，更加形象。
订阅者：

```
    var subscribe_1 = {
        update: function(){
            console.log('This is subscribe_1');
        }
    };
    var subscribe_2 = {
        update: function(){
            console.log('This is subscribe_2');
        }
    };
    var subscribe_3 = {
        update: function(){
            console.log('This is subscribe_3');
        }
    };
```

三个订阅者都有update方法。

发布者：
```

    function Publisher(){
        this.subs = [subscribe_1, subscribe_2, subscribe_3]; // 添加订阅者
    }
    Publisher.prototype = {
        constructor: Publisher,
        notify: function(){
            this.subs.forEach(function(sub){
                sub.update();
            })
        }
    };

发布者通过notify方法对订阅者广播，订阅者通过update来接受信息。
实例化publisher：

    var publisher = new Publisher();
    publisher.notify();
clipboard.png

这里我们可以做一个中间件来处理发布者-订阅者模式：

    var publisher = new Publisher();
    var middleware = {
        publish: function(){
            publisher.notify();
        }
    };
    middleware.publish();
```

5、观察者模式嵌入
到这一步，我们已经实现了：
1、修改v-model属性元素 -> 触发修改vm的属性值 -> 触发set
2、发布者添加订阅 -> notify分发订阅 -> 订阅者update数据
接下来我们要实现：更新视图，同时把订阅——发布者模式嵌入。

发布者：

```
    function Publisher(){
        this.subs = []; // 订阅者容器
    }
    Publisher.prototype = {
        constructor: Publisher,
        add: function(sub){
            this.subs.push(sub); // 添加订阅者
        },
        notify: function(){
            this.subs.forEach(function(sub){
                sub.update(); // 发布订阅
            });
        }
    };
订阅者：
考虑到要把订阅者绑定data的每个属性，来观察属性的变化，参数：name参数可以有compile中获取的name传参。由于传入的node节点类型分为两种，我们可以分为两订阅者来处理，同时也可以对node节点类型进行判断，通过switch分别处理。

    function Subscriber(node, vm, name){
        this.node = node;
        this.vm = vm;
        this.name = name;
    }
    Subscriber.prototype = {
        constructor: Subscriber,
        update: function(){
            let vm = this.vm;
            let node = this.node;
            let name = this.name;
            switch(this.node.nodeType){
                case 1:
                    node.value = vm[name];
                    break;
                case 3:
                    node.nodeValue = vm[name];
                    break;
                default:
                    break;
            }
        }
    };
我们要把订阅者添加到compile进行虚拟dom的初始化，替换掉原来的赋值：

    function compile(node, data){
        let reg = /\{\{(.*)\}\}/g;
        if(node.nodeType === 1){ // 标签
            let attr = node.attributes;
            for(let i = 0, len = attr.length; i < len; i++){
                // console.log(attr[i].nodeName, attr[i].nodeValue);
                if(attr[i].nodeName === 'v-model'){
                    let name = attr[i].nodeValue;
                    // --------------------这里被替换掉
                    // node.value = data[name]; 
                    new Subscriber(node, data, name);

                    // ------------------------添加监听事件
                    node.addEventListener('keyup', function(e){
                        data[name] = e.target.value;
                    }, false);
                }
            }
            if(node.hasChildNodes()){
                node.childNodes.forEach((item) => {
                    compile(item, data);
                });
            }
        }
        if(node.nodeType === 3){ // 文本节点
            if(reg.test(node.nodeValue)){
                let name = RegExp.$1;
                name = name.trim();
                // ---------------------这里被替换掉
                // node.nodeValue = data[name];
                new Subscriber(node, data, name);
            }
        }
    }
既然是对虚拟dom编译初始化，Subscriber要初始化，即Subscriber.update,因此要对Subscriber作进一步的处理：

   function Subscriber(node, vm, name){
        this.node = node;
        this.vm = vm;
        this.name = name;
        
        this.update();
    }
    Subscriber.prototype = {
        constructor: Subscriber,
        update: function(){
            let vm = this.vm;
            let node = this.node;
            let name = this.name;
            switch(this.node.nodeType){
                case 1:
                    node.value = vm[name];
                    break;
                case 3:
                    node.nodeValue = vm[name];
                    break;
                default:
                    break;
            }
        }
    };
发布者添加到defineReact，来观察数据的变化：

    function defineReact(data, key, value){
        let publisher = new Publisher();
        Object.defineProperty(data, key, {
            set: function(newValue){
                console.log(`触发setter`);
                value = newValue;
                console.log(value);
                publisher.notify(); // 发布订阅
            },
            get: function(){
                console.log(`触发getter`);
                if(Publisher.global){ //这里为什么来添加判断条件，主要是让publisher.add只执行一次，初始化虚拟dom编译的时候来执行
                    publisher.add(Publisher.global); // 添加订阅者
                }
                return value;
            }
        });
    }
这一步将订阅者添加到发布者容器内，对订阅者改造：

  function Subscriber(node, vm, name){
        Publisher.global = this;
        this.node = node;
        this.vm = vm;
        this.name = name;
        
        this.update();
        Publisher.global = null;
    }
    Subscriber.prototype = {
        constructor: Subscriber,
        update: function(){
            let vm = this.vm;
            let node = this.node;
            let name = this.name;
            switch(this.node.nodeType){
                case 1:
                    node.value = vm[name];
                    break;
                case 3:
                    node.nodeValue = vm[name];
                    break;
                default:
                    break;
            }
        }
    };
6、完整效果

html:

<div id="container">
    {{ msg }}<br>
    <input class="inp-text" type="text" v-model="inpText">
    <p>{{ inpText }}</p>
    <div class="text-box">
        <p class="show-text">{{ msg }}</p>
    </div>
</div>
javascript:

    function Publisher(){
        this.subs = [];
    }
    Publisher.prototype = {
        constructor: Publisher,
        add: function(sub){
            this.subs.push(sub);
        },
        notify: function(){
            this.subs.forEach(function(sub){
                sub.update();
            });
        }
    };

    function Subscriber(node, vm, name){
        Publisher.global = this;
        this.node = node;
        this.vm = vm;
        this.name = name;
        this.update();
        Publisher.global = null; // 清空
    }
    Subscriber.prototype = {
        constructor: Subscriber,
        update: function(){
            let vm = this.vm;
            let node = this.node;
            let name = this.name;
            switch(this.node.nodeType){
                case 1:
                    node.value = vm[name];
                    break;
                case 3:
                    node.nodeValue = vm[name];
                    break;
                default:
                    break;
            }
        }
    };

    function virtualDom(node, data){
        let frag = document.createDocumentFragment();
        let child;
        // 遍历dom节点
        while(child = node.firstChild){
            compile(child, data);
            frag.appendChild(child);
        }
        return frag;
    }

    function compile(node, data){
        let reg = /\{\{(.*)\}\}/g;
        if(node.nodeType === 1){ // 标签
            let attr = node.attributes;
            for(let i = 0, len = attr.length; i < len; i++){
                // console.log(attr[i].nodeName, attr[i].nodeValue);
                if(attr[i].nodeName === 'v-model'){
                    let name = attr[i].nodeValue;
                    // node.value = data[name];

                    // ------------------------添加监听事件
                    node.addEventListener('keyup', function(e){
                        data[name] = e.target.value;
                    }, false);

                    new Subscriber(node, data, name);

                }
            }
            if(node.hasChildNodes()){
                node.childNodes.forEach((item) => {
                    compile(item, data);
                });
            }
        }
        if(node.nodeType === 3){ // 文本节点
            if(reg.test(node.nodeValue)){
                let name = RegExp.$1;
                name = name.trim();
                // node.nodeValue = data[name];

                new Subscriber(node, data, name);
            }
        }
    }


    function defineReact(data, key, value){
        let publisher = new Publisher();
        Object.defineProperty(data, key, {
            set: function(newValue){
                console.log(`触发setter`);
                value = newValue;
                console.log(value);
                publisher.notify(); // 发布订阅
            },
            get: function(){
                console.log(`触发getter`);
                if(Publisher.global){
                    publisher.add(Publisher.global); // 添加订阅者
                }
                return value;
            }
        });
    }


    // 将data中数据绑定到vm实例对象上
    function observe(data, vm){
        Object.keys(data).forEach((key) => {
            defineReact(vm, key, data[key]);
        })
    }


    function Vue(options){
        this.data = options.data;
        let id = options.el;

        observe(this.data, this); // 将每个data属相绑定到Vue的实例vm上

        //------------------------
        let container = document.getElementById(id);
        let fragment = virtualDom(container, this); // 这里通过vm对象初始化
        container.appendChild(fragment);

    }


    var vm = new Vue({
        el: 'container',
        data: {
            msg: 'Hello world!',
            inpText: 'Input text'
        }
    });
```
