---
title: Vue.js设计与实现
date: 2023-06-12 22:47:50
tags: [前端,Vue]

categories: [前端, Vue]

---



## Vue



> 这是一篇针对Vue相关知识点整合的文，用于剖析Vue的相关原理以及特性梳理。



### Vue的设计模式



#### MVC 和 MVVM 区别

##### MVC

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范

- Model（模型）：是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据
- View（视图）：是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的
- Controller（控制器）：是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据

MVC 的思想：一句话描述就是 Controller 负责将 Model 的数据用 View 显示出来，换句话说就是在 Controller 里面把 Model 的数据赋值给 View。

##### MVVM

MVVM 新增了 VM 类

> ViewModel 层：做了两件事达到了数据的双向绑定 一是将【模型】转化成【视图】，即将后端传递的数据转化成所看到的页面。实现的方式是：数据绑定。二是将【视图】转化成【模型】，即将所看到的页面转化成后端的数据。实现的方式是：DOM 事件监听。

MVVM 与 MVC 最大的区别就是：它实现了 View 和 Model 的自动同步，也就是当 Model 的属性改变时，我们不用再自己手动操作 Dom 元素，来改变 View 的显示，而是改变属性后该属性对应 View 层显示会自动改变（对应Vue数据驱动的思想）

整体看来，MVVM 比 MVC 精简很多，不仅简化了业务与界面的依赖，还解决了数据频繁更新的问题，不用再用选择器操作 DOM 元素。因为在 MVVM 中，View 不知道 Model 的存在，Model 和 ViewModel 也观察不到 View，这种低耦合模式提高代码的可重用性

> 注意：Vue 并没有完全遵循 MVVM 的思想 这一点官网自己也有说明。严格的 MVVM 要求 View 不能和 Model 直接通信，而 Vue 提供了$refs 这个属性，让 Model 可以直接操作 View，违反了这一规定，所以说 Vue 没有完全遵循 MVVM。



### 生命周期



![Vue生命周期](../images/Vue.js%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/vue-life.jpg)

> **beforeCreate** 在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。在当前阶段 data、methods、computed 以及 watch 上的数据和方法都不能被访问
>
> **created** 实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。这里没有$el,如果非要想与 Dom 进行交互，可以通过 vm.$nextTick 来访问 Dom
>
> **beforeMount** 在挂载开始之前被调用：相关的 render 函数首次被调用。
>
> **mounted** 在挂载完成后发生，在当前阶段，真实的 Dom 挂载完毕，数据完成双向绑定，可以访问到 Dom 节点
>
> **beforeUpdate** 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁（patch）之前。可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程
>
> **updated** 发生在更新完成之后，当前阶段组件 Dom 已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新，该钩子在服务器端渲染期间不被调用。
>
> **beforeDestroy** 实例销毁之前调用。在这一步，实例仍然完全可用。我们可以在这时进行善后收尾工作，比如清除计时器。
>
> **destroyed** Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。 该钩子在服务器端渲染期间不被调用。
>
> **activated** keep-alive 专属，组件被激活时调用
>
> **deactivated** keep-alive 专属，组件被销毁时调用



#### keep-alive 中的生命周期有哪些

> ​	keep-alive 是 Vue 提供的一个内置组件，用来对组件进行缓存，在组件切换过程中将状态保留在内存中，防止重复渲染 DOM。 如果为一个组件包裹了 keep-alive，那么它会多出两个生命周期：deactivated、activated。
>
> ​	同时，beforeDestroy 和 destroyed 就不会再被触发了，因为组件不会被真正销毁。 当组件被换掉时，会被缓存到内存中、触发 deactivated 生命周期； 当组件被切回来时，再去缓存里找这个组件、触发 activated 钩子函数。



#### 异步请求在哪一步发起？

> 可以在钩子函数 created、beforeMount、mounted 中进行异步请求，因为在这三个钩子函数中，data 已经创建，可以将服务端端返回的数据进行赋值。
>
> 如果异步请求不需要依赖 Dom 推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点：
>
> - 能更快获取到服务端数据，减少页面  loading 时间；
> - ssr  不支持 beforeMount 、mounted 钩子函数，所以放在 created 中有助于一致性；



#### Vue 的父子组件生命周期钩子函数执行顺序

>**加载渲染过程:**
>
>父 beforeCreate->父 created->父 beforeMount->子 beforeCreate->子 created->子 beforeMount->子 mounted->父 mounted
>
>**子组件更新过程:**
>
>父 beforeUpdate->子 beforeUpdate->子 updated->父 updated
>
>**父组件更新过程:**
>
>父 beforeUpdate->父 updated
>
>**销毁过程:**
>
>父 beforeDestroy->子 beforeDestroy->子 destroyed->父 destroyed



###  数据绑定

#### 组件中为什么 `data` 是一个函数？

> 在 Vue 组件中，为什么 `data` 是一个函数而不是一个对象的原因是为了确保每个组件实例都有其自己的数据副本。当组件被定义时，`data` 必须是一个函数。每次创建组件实例时，Vue 都会调用该函数来返回一个全新的数据对象。
>
> 这是因为 Vue 组件可以在应用中存在多个实例，每个实例都应该具有独立的状态和数据。如果 `data` 是一个对象，那么所有组件实例将共享相同的数据对象，这将导致一个实例的数据变化会影响到其他实例。
>
> 通过将 `data` 定义为函数，Vue 在创建组件实例时会为每个实例调用该函数，从而返回一个新的数据对象。这样，每个组件实例都有自己的数据副本，它们之间相互独立，可以独立地修改和维护各自的状态。



#### v-model 原理

v-model 其实就是语法糖，v-model 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：

>text 和 textarea 元素使用 value property 和 input 事件；
>
>checkbox 和 radio 使用 checked property 和 change 事件；
>
>select 字段将 value 作为 prop 并将 change 作为事件。

在普通标签上:

```html
<input v-model="sth" />  //这一行等于下一行
<input v-bind:value="sth" v-on:input="sth = $event.target.value" />
```

在组件上:

```html
<currency-input v-model="price"></currentcy-input>
<!--上行代码是下行的语法糖
 <currency-input :value="price" @input="price = arguments[0]"></currency-input>
-->

<!-- 子组件定义 -->
Vue.component('currency-input', {
 template: `
  <span>
   <input
    ref="input"
    :value="value"
    @input="$emit('input', $event.target.value)"
   >
  </span>
 `,
 props: ['value'],
})
```

#### Vue数据双向绑定原理

##### vue的响应式基本原理：

> 1、vue会遍历此data中对象所有的属性，
>
> 2、并使用Object.defineProperty进行数据劫持，把这些属性全部转为getter/setter，
>
> 3、而每个组件实例都有watcher对象，
>
> 4、它会在组件渲染的过程中把属性记录为依赖，
>
> 5、之后当依赖项的 setter被调用时，会通知watcher重新计算，从而致使它关联的组件得以更新。

eg：

Object.defineProperty( ):

```js
var person = {}
var name = '';
Object.defineProperty(person, 'name', {
  set: function (value) {
    name = value;
  },
  get: function () {
    return "My name is " + name
  }
})
 
person.name = 'Jayhodon';
console.log(person.name);  // My name is Jayhodon
```



##### vue的响应式原理设计三个重要对象：

> Observer对象：vue中的数据对象在初始化过程中转换为Observer对象。
>
> Watcher对象：将模板和Observer对象结合在一起生成Watcher实例，Watcher是订阅者中的订阅者。
>
> Dep对象：Watcher对象和Observer对象之间纽带，每一个Observer都有一个Dep实例，用来存储订阅者Watcher。

当属性变化会执行主题对象Observer的dep.notify方法， 这个方法会遍历订阅者Watcher列表向其发送消息， Watcher会执行run方法去更新视图。模板编译过程中的指令和数据绑定都会生成Watcher实例，实例中的watch属性也会生成Watcher实例。

总的来说就是：

> 1、在生命周期的initState方法中将data，prop，method，computed，watch中的数据劫持， 通过observe方法与Object.defineProperty方法将相关对象转为换Observer对象。
>
> 2、然后在initRender方法中解析模板，通过Watcher对象，Dep对象与观察者模式将模板中的 指令与对象的数据建立依赖关系，使用全局对象Dep.target实现依赖收集。
>
> 3、当数据变化时，setter被调用，触发Object.defineProperty方法中的dep.notify方法， 遍历该数据依赖列表，执行器update方法通知Watcher进行视图更新。



##### 使用Object.defineProperty实现监听变量:

实现步骤：

Observer：

 用来劫持并监听所有属性，如果有变动的，就通知订阅者。

```js
function Observer(data) {
    this.data = data;
    this.walk(data);
}

Observer.prototype = {
    walk: function(data) {
        var self = this;
        Object.keys(data).forEach(function(key) {
            self.defineReactive(data, key, data[key]);
        });
    },
    defineReactive: function(data, key, val) {
        var dep = new Dep();
        var childObj = observe(val);// 递归遍历所有子属性
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get: function getter () {
                if (Dep.target) {
                    dep.addSub(Dep.target);//判断是否需要添加订阅者，并添加订阅者
                }
                return val;
            },
            set: function setter (newVal) {
                if (newVal === val) {
                    return;
                }
                val = newVal;
                dep.notify();//如果数据发生了变化，则通知所有的订阅者
            }
        });
    }
};

function observe(value, vm) {
    if (!value || typeof value !== 'object') {
        return;
    }
    return new Observer(value);
};

//Dep主要负责收集订阅者，然后再属性变化的时候执行对应订阅者的更新函数
function Dep () {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};
Dep.target = null;
```

Watcher：

 可以收到属性的变化通知并执行相应的函数，从而更新视图。

```js
function Watcher(vm, exp, cb) {
    this.cb = cb;
    this.vm = vm;
    this.exp = exp;
    this.value = this.get();  // 将自己添加到订阅器的操作
}

Watcher.prototype = {
    update: function() {
        this.run();
    },
    run: function() {
        var value = this.vm.data[this.exp];
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);
        }
    },
    get: function() {
        Dep.target = this;  // 缓存自己
        var value = this.vm.data[this.exp]  // 强制执行监听器里的get函数
        Dep.target = null;  // 释放自己
        return value;
    }
};
```

这时只要将Observer和Watcher关联起来，就可以实现一个简单的双向绑定数据了

```js
function SelfVue (data, el, exp) {
    this.data = data;
    observe(data);
    el.innerHTML = this.data[exp];  // 初始化模板数据的值
    new Watcher(this, exp, function (value) {
        el.innerHTML = value;
    });
    return this;
}
```

完整版参考自：[vue的双向绑定原理及实现](https://www.cnblogs.com/libin-1/p/6893712.html)    [源 码](https://github.com/canfoo/self-vue/tree/master/v3)



##### 使用ES6的proxy简单实现监听变量:

```js
let obj = {
 msg: {
  a: 10
 },
 arr: [1, 2, 3]
}

let handler = {
 get(target, key){
  //懒监听，去获取的时候才监听对象里面的对象，而不是直接递归循环监听
  if(typeof target[key] === 'object' && target[key] !== null){
   return new Proxy(target[key], handler);
  }
  return Reflect.get(target, key);
 },
 set(target, key, value){
  console.log('set', target, key, value);
  //数组新增会执行两次，一次是修改length，一次是添加值
  let oldValue = target[key];
  if(!oldValue){
   //找不到老值，新增
  }else if(oldValue !== value){
   //老值和新值不相等，修改
  }
  return Reflect.set(target, key, value);
 }
}

let proxy = new Proxy(obj, handler)
proxy.arr.push(4);
proxy.msg.a = 50;
proxy.msg.b = 60;
proxy.c = 70;
```

区别：

> 1、语法层面上
>
> defineProperty只能响应首次渲染时候的属性，
>
> defineProperty无法一次性监听所有属性，必须通过遍历或者递归的方式来实现且无法监听新增的属性，对于数组defineProperty则需要劫持数组方法。
>
> Proxy需要的是整体监听，不需要关心里面有什么属性，而且Proxy的配置项有13种，可以做更细致的事情，这是之前的defineProperty无法达到的。
>
> 2、兼容层面上
>
> vue2.x之所以只能兼容到IE8就是因为defineProperty无法兼容IE8,其他浏览器也会存在轻微兼容问题。
>
> proxy的话除了IE，其他浏览器都兼容，这次vue3还是使用了它，说明vue3直接放弃了IE的兼容考虑。

 为什么Vue 3 使用 Proxy 替代了 Vue 2 中的 Object.defineProperty：

> 1. 更好的性能：Vue 3 使用 Proxy 可以提供比 Object.defineProperty 更高的性能。Proxy 可以拦截更多的操作，包括属性访问、属性设置、删除属性等，而 Object.defineProperty 只能拦截属性的读取和设置。
> 2. 更好的扩展性：Proxy 提供了更多的拦截方法，可以针对更多的操作进行定制。这使得 Vue 3 在响应式系统的实现上更加灵活和可扩展。
> 3. 更好的支持嵌套对象：Proxy 对于嵌套对象的响应式支持更加完善。Vue 3 的响应式系统可以追踪到嵌套对象的变化，并在需要时触发更新。
> 4. 更好的 TypeScript 支持：Proxy 的类型推断更准确，可以提供更好的 TypeScript 支持，让开发者在编码过程中能够获得更准确的类型提示和错误检查。

​	总体来说，Vue 3 使用 Proxy 替代了 Vue 2 中的 Object.defineProperty，主要是为了提供更好的性能、更好的扩展性、更好的嵌套对象支持和更好的 TypeScript 支持。这些改进使得 Vue 3 在响应式系统的实现上更加先进和灵活，提供了更好的开发体验和性能优化。



#### Vue 中的数据为什么频繁变化时只会更新一次 ？

​	在 Vue 中，当数据频繁变化时，Vue 会对数据变化进行优化处理，以减少不必要的更新操作，从而提高性能。	

> Vue 使用了异步更新队列的机制，即将数据变化的通知放入队列中，然后在下一个事件循环周期中统一进行更新操作。这样做的好处是，当数据频繁变化时，不会立即触发更新，而是等待下一个事件循环周期进行批量更新，从而避免了频繁的更新操作。
>
> 具体来说，当多次修改数据时，Vue 会将这些修改操作合并为一个更新操作。例如，连续对同一个数据进行多次修改，只会触发一次更新操作，以最终的修改结果为准。
>
> 这种优化机制可以有效减少不必要的 DOM 操作和重新渲染，提升性能和效率。同时，也避免了频繁的更新导致的性能问题和不必要的资源消耗。

​	然而，需要注意的是，由于异步更新机制，Vue 在某些特定场景下可能无法立即获取到最新的数据。如果需要在数据更新后立即执行某些操作，可以利用 Vue 提供的 `$nextTick` 方法或使用 Vue 的生命周期钩子函数来确保在更新完成后执行相应的操作。

​	总之，Vue 的更新机制保证了数据变化的高效处理，并通过合并操作减少了不必要的更新，提升了应用的性能和响应速度。

​	

##### this.$nextTick() 作用及实现原理

​	`this.$nextTick()` 是 Vue 实例提供的一个方法，用于在 DOM 更新完成后执行回调函数。它的作用是确保在下次 DOM 更新循环结束后执行回调函数，以获取到最新的 DOM 状态。

> 实现原理：
>
> 1. 当调用 `this.$nextTick()` 方法时，Vue 会将传入的回调函数添加到一个队列中，该队列用于存储待执行的回调函数。
> 2. Vue 会检测当前是否存在微任务（Promise 或 MutationObserver）或宏任务（setTimeout 或 setImmediate）队列。
> 3. 如果存在微任务或宏任务队列，则将回调函数添加到微任务或宏任务队列中，确保在下一个事件循环周期中执行。
> 4. 如果当前不存在微任务或宏任务队列，则创建一个微任务队列，并将回调函数添加到其中。
> 5. 在下一个事件循环周期中，Vue 会执行微任务队列中的所有回调函数，并将其从队列中移除。

​	通过这种机制，`this.$nextTick()` 方法可以保证在 DOM 更新完成后执行回调函数，以便获取到最新的 DOM 状态。这在某些场景下非常有用，比如在修改数据后立即获取更新后的 DOM 元素，或在更新后对某些 DOM 操作进行后续处理。

​	需要注意的是，由于 `this.$nextTick()` 方法使用了异步更新机制，因此回调函数的执行时机不是立即的，而是在下一个事件循环周期中。如果需要在回调函数执行完成后执行一些操作，可以在回调函数中进行处理，或者使用 `Promise` 或 `async/await` 来等待回调函数执行完成。



### 组件通讯

####  Vue 组件通讯有哪几种方式

> 1. props 和$emit 父组件向子组件传递数据是通过 prop 传递的，子组件传递数据给父组件是通过$emit 触发事件来做到的
>
> 2. $parent,$children 获取当前组件的父组件和当前组件的子组件
>
> 3. 父组件中通过 provide 来提供变量，然后在子组件中通过 inject 来注入变量。(官方不推荐在实际业务中使用，但是写组件库时很常用，用于跨级通讯)
>
> 4. $refs 获取组件实例
>
> 5. envetBus 兄弟组件数据传递 这种情况下可以使用事件总线的方式
>
> 6. vuex 状态管理

**props & $emit:**

```vue
<Son @changeData="changeData"></Son>

<script>
 import Son from '@/components/son'
 export default{
   name:'Father',
   components:{Son},
   methods:{
     changeData(name){
       console.log(name) //来自子组件的数据
     }
   }
 }
</script>
<el-button @click="handleEmit">通知父组件需要更改数据了</el-button>

<script>
 export default{
   name:'Son',
   methods:{
     handleEmit(){
       this.$emit('changeData','这是来自子组件的数据')
     }
   }
 }
</script>
```

**$parent & $children：**

```vue
<template>
  <div>子组件</div>
</template>

<script>
export default{
  name:"Son",
  data(){
    return{
      sonData: '子组件的数据'
    }
  },
  methods:{
    sonHandle(){
      console.log('子组件的方法')
    }
  },
  created(){
    console.log(this.$parent)
    console.log(this.$parent.fatherData) //父组件的数据
    this.$parent.fantherHandle() //父组件的方法
  }
}
</script>
```



```vue
<template>
  <div>
    <Son>父组件</Son>
  </div>
</template>

<script>
import Son from './son.vue'

export default{
  name: 'father',
  components:{
    Son
  },
  data(){
    return{
      fatherData: '父组件的数据'
    }
  },
  methods:{
    fantherHandle(){
      console.log('父组件的方法')
    }
  },
  mounted(){
    console.log(this.$children)
    console.log(this.$children[0].sonTitle) //子组件的数据
    this.$children[0].sonHandle() //子组件的方法
  }
}
</script>
```

这里要注意父组件要在mounted（）这个生命周期对子组件进行取值因为在这时候子组件才完成了 created 与 mounted，并且获取到的数据是一个数组的形式。

**provide $ inject：**

```vue
/*父组件*/
export default{
 provide: {
   return{
     provideData: '父组件的数据'
   }
 }
}
/*子组件*/
export default{
  inject: ['provideData'],
  created () {
    console.log(this.provideName) //"父组件的数据"
  }
}
```

**$refs：**

```vue
<template>
  <div>
    <Son ref="son"></Son>
  </div>
</template>

<script>
import Son from './son.vue'

export default{
  name: 'father',
  components:{
    Son
  },
  mounted(){
    console.log(this.$refs.son) /*组件实例*/
  }
}
</script>
```

**envetBus：**

需要先创建一个公共的eventBus.js，并将Vue实例暴露出去

```vue
import Vue from "vue"
export default new Vue()
```

 在需要组件通信的组件A中引入eventBus.js，并通过$emit发布回调事件

```vue
<template>
  <div>
    <div>组件A</div>
    <el-button @click="changeData">修改数据A</el-button>
  </div>
</template>

<script>
import { EventBus } from "../bus.js"
export default{
  data(){
    return{}
  },
  methods:{
    changeData(){
      EventBus.$emit("editData", '这是修改后的数据')
    }
  }
}
</script>
```

组件B中同样引入eventBus.js文件，并通过$on监听事件回调

``` vue
<template>
  <div>组件B</div>
</template

<script>
import { EventBus } from "../bus.js"
export default{
  data(){
    return{}
  },
  mounted:{
    EventBus.$on('editData',(data)=>{
      console.log(data) 
    })
  }
}
</script>
```



#### 父子组件通讯

在 Vue 中，父子组件之间可以通过 props 和事件来进行通信。

1. 父组件向子组件传递数据：通过在父组件中使用 props 将数据传递给子组件。子组件可以在其模板中通过使用 props 来接收父组件传递的数据。

   父组件：

   ```vue
   <template>
     <ChildComponent :message="parentMessage" />
   </template>
   
   <script>
   import ChildComponent from './ChildComponent.vue';
   
   export default {
     data() {
       return {
         parentMessage: 'Hello from parent',
       };
     },
     components: {
       ChildComponent,
     },
   };
   </script>
   ```

   子组件：

   ```vue
   <template>
     <div>{{ message }}</div>
   </template>
   
   <script>
   export default {
     props: ['message'],
   };
   </script>
   ```

2. 子组件通过触发事件改变父组件的值：子组件可以通过 $emit 方法触发自定义事件，从而通知父组件进行相应的操作。

   父组件：

   ```vue
   <template>
     <div>
       <ChildComponent :counter="counter" @increment="handleIncrement" />
       <p>Counter: {{ counter }}</p>
     </div>
   </template>
   
   <script>
   import ChildComponent from './ChildComponent.vue';
   
   export default {
     data() {
       return {
         counter: 0,
       };
     },
     components: {
       ChildComponent,
     },
     methods: {
       handleIncrement() {
         this.counter++;
       },
     },
   };
   </script>
   ```

   子组件：

   ```vue
   <template>
     <button @click="handleClick">Increment</button>
   </template>
   
   <script>
   export default {
     props: ['counter'],
     methods: {
       handleClick() {
         this.$emit('increment');
       },
     },
   };
   </script>
   ```

​	通过上述方式，父子组件可以进行数据的传递和通信。子组件可以通过触发事件的方式来改变父组件的值。需要注意的是，子组件不能直接修改父组件的 props 数据，而是通过事件向父组件发送请求，由父组件进行相应的操作来修改数据。

​	如果子组件需要修改父组件的数据，父组件可以将需要修改的数据作为 props 传递给子组件，然后子组件通过触发事件来请求父组件进行数据的修改。这样可以保持数据流的单向性，提高组件的可维护性和可预测性。



#### 平行组件通信

在 Vue 中，平行组件之间的通信可以通过以下几种方式实现：

> 1. 使用共享状态（Shared State）：可以创建一个共享的数据源，例如 Vuex 状态管理库或者一个全局的事件总线。所有的平行组件都可以访问和修改这个共享的状态，从而实现通信。
> 2. 使用父组件作为中介：如果平行组件位于同一个父组件下，可以通过父组件作为中介来进行通信。平行组件通过将需要共享的数据或者方法传递给父组件，再由父组件将数据或者方法传递给其他平行组件。
> 3. 使用事件总线：可以创建一个全局的事件总线，利用 Vue 的实例作为事件中心，平行组件通过订阅和触发事件来进行通信。
> 4. 使用浏览器的事件系统：平行组件可以通过浏览器的事件系统（如 `window.dispatchEvent` 和 `window.addEventListener`）来进行通信。组件可以通过触发自定义事件，然后在其他组件中通过监听这些事件来实现通信。
> 5. 使用非父子组件通信插件：可以使用一些 Vue 的插件来实现非父子组件之间的通信，例如 Vue Bus、mitt 等。

​	需要根据具体的场景和需求选择合适的通信方式。在选择通信方式时，可以考虑组件之间的关系、数据的复杂度以及通信的频率等因素。



### 状态管理 - Vuex

​	Vuex是Vue.js官方提供的状态管理模式和库。它被设计用于解决Vue应用中的状态管理问题。Vuex基于Flux架构和Redux模式，提供了一种集中管理和共享状态的机制。

Vuex的核心概念包括：

> 1. State（状态）：用于存储应用的状态数据，类似于组件中的data。它是响应式的，可以通过getter获取和修改。
> 2. Mutation（变更）：用于修改状态的方法，类似于组件中的methods。只能进行同步操作，且只能在mutation中修改状态。
> 3. Action（动作）：用于处理异步操作或复杂的业务逻辑，可以包含多个mutation的组合。可以触发mutation来修改状态。
> 4. Getter（获取器）：用于派生状态，类似于组件中的computed。可以对状态进行计算和包装，提供派生的数据。
> 5. Module（模块）：将大型应用的状态拆分为多个模块，每个模块有自己的state、mutation、action和getter。

​	通过使用Vuex，我们可以集中管理和共享应用的状态，使得状态的变化和处理逻辑更可控和可维护。它适用于大型的、状态复杂的应用，可以简化组件之间的通信，提高代码的可读性和可测试性。同时，Vuex也提供了开发工具和插件，方便调试和扩展。

​	在Vue应用中使用Vuex需要先安装和配置，然后在组件中引入和使用。通过定义state、mutations、actions和getters，我们可以在组件中访问和修改共享状态。Vuex还提供了一些辅助函数和工具，用于简化使用和处理异步操作。



#### mutations 能不能做异步

​	在Vuex中，mutations默认是同步操作，只能用于修改状态的同步变更。这是为了确保状态的变更是可追踪和可预测的。

​	在某些情况下，我们可能需要在mutations中进行异步操作，比如在异步请求数据后再修改状态。然而，直接在mutations中执行异步操作是不被推荐的做法，因为它会破坏状态变更的可追踪性。

​	如果需要进行异步操作，可以使用actions来处理。Actions可以包含异步操作，并且可以触发mutations来修改状态。这样可以保持状态变更的可追踪性，同时也可以方便地进行异步处理。

示例代码：

```js
// Vuex store
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    },
    decrement(state) {
      state.count--;
    }
  },
  actions: {
    asyncIncrement(context) {
      setTimeout(() => {
        context.commit('increment');
      }, 1000);
    }
  }
});

// 在组件中使用
export default {
  methods: {
    increment() {
      this.$store.commit('increment');
    },
    decrement() {
      this.$store.commit('decrement');
    },
    asyncIncrement() {
      this.$store.dispatch('asyncIncrement');
    }
  }
}
```

​	在上述示例中，通过`context.commit`来触发mutation的执行，`this.$store.dispatch`来触发action的执行。这样就可以实现异步操作并且保持状态变更的可追踪性。

​	虽然mutations默认是同步操作，但可以通过actions来处理异步操作，并在actions中触发mutations来修改状态。这样可以更好地控制和管理状态的变更过程。



#### Vuex 页面刷新数据丢失怎么解决

​	当刷新页面时，由于Vuex中的数据存储在内存中，数据会丢失。这是因为Vuex是基于客户端的状态管理库，数据不会被持久化保存。每次刷新页面都会重新初始化Vuex的状态。

​	为了解决刷新页面导致Vuex数据丢失的问题，可以考虑以下几种方法：

> 1. 使用持久化方案：可以将Vuex的状态持久化保存到本地存储（如localStorage或sessionStorage）中。在应用初始化时，从本地存储中读取状态并还原到Vuex中。这样可以在刷新页面后重新加载保存的状态，避免数据丢失。
> 2. 利用路由参数或URL查询参数传递数据：将需要持久化的数据通过路由参数或URL查询参数传递给其他页面。这样在刷新页面时，可以通过获取路由参数或URL查询参数来恢复数据。
> 3. 使用后端存储方案：如果应用需要长期保存状态数据，并且在不同设备或浏览器中共享状态，可以考虑将状态数据存储在后端服务器或数据库中。在刷新页面时，可以通过请求后端获取保存的状态数据并还原到Vuex中。

​	vuex 数据持久化 一般使用本地存储的方案来保存数据 可以自己设计存储方案 也可以使用第三方插件。推荐使用 `vuex-persist` 插件，它就是为 Vuex 持久化存储而生的一个插件。不需要你手动存取 storage ，而是直接将状态保存至 cookie 或者 localStorage 中

> ​	ps: 需要根据具体的应用场景和需求选择适合的方法。持久化方案和后端存储方案可能需要进行数据序列化和反序列化操作，以确保数据的正确保存和恢复。同时，需要注意数据的安全性和隐私保护，避免敏感数据泄露。



#### Vuex 和 localStorage 的区别

Vuex和localStorage是两种不同的概念和用途：

> 1. Vuex：Vuex是Vue.js官方提供的状态管理库。它用于在Vue应用中集中管理和共享状态。Vuex的目的是解决组件之间共享数据、状态管理和数据流的问题。Vuex提供了一种集中式的数据存储机制，可以在多个组件中访问和修改共享的状态，以实现组件之间的通信和数据同步。Vuex的数据存储在内存中，它是基于内存的状态管理解决方案，适用于应用内部的数据共享和状态管理。
> 2. localStorage：localStorage是Web浏览器提供的一种本地存储机制。它允许将数据以键值对的形式存储在浏览器的本地存储空间中，并且数据在页面刷新或关闭后仍然保持有效。localStorage提供了持久化的存储能力，可以在不同的页面和会话中共享数据。localStorage中的数据以字符串形式存储，需要进行数据的序列化和反序列化。

区别：

> - Vuex是专门为Vue应用设计的状态管理库，用于管理应用内的状态和数据流，解决组件之间的数据共享和状态管理问题。而localStorage是Web浏览器提供的一种本地存储机制，用于持久化地存储数据，并且在不同的浏览器会话中共享数据。
> - Vuex的数据存储在内存中，适用于应用内部的数据共享和状态管理，而localStorage的数据存储在浏览器的本地存储空间中，可以在不同的页面和会话中共享数据。
> - Vuex提供了一套API和机制用于管理和修改状态，包括使用mutation修改状态、使用action处理异步操作等。而localStorage是基于键值对的简单存储机制，只能通过setItem和getItem等方法进行基本的数据存储和读取，没有提供数据管理和状态变更的机制。

​	Vuex适用于Vue应用内部的状态管理和数据共享，而localStorage适用于持久化地存储和共享数据。在具体使用时，可以根据应用需求选择合适的方案或结合两者使用。



### 虚拟 dom 和 diff 算法

​	虚拟DOM（Virtual DOM）和差异算法（Diff Algorithm）是前端领域中常用的概念，它们在优化页面渲染性能和提升开发效率方面发挥了重要作用。

> ​	虚拟DOM是一种以JavaScript对象形式表示的轻量级的内存中的DOM表示。它是对真实DOM的一种抽象和模拟，在内存中进行操作和计算，然后再将结果批量更新到真实的DOM上。虚拟DOM的目的是在保持视图和状态同步的同时，减少对真实DOM的直接操作，从而提升性能和响应速度。
>
> ​	差异算法（Diff Algorithm）是虚拟DOM的核心算法之一，用于比较两个虚拟DOM树之间的差异，并将差异应用到真实的DOM上。通过比较新旧虚拟DOM树的差异，可以准确地找出需要更新的部分，避免全量更新整个DOM树，从而减少不必要的操作和性能损耗。

​	Diff算法的基本原理是对比新旧虚拟DOM树的节点，找出发生变化的节点，然后根据变化类型（插入、更新、移除等）对真实DOM进行相应的操作。Diff算法通常采用深度优先遍历的方式，通过递归遍历虚拟DOM树的节点，对比节点之间的差异，生成更新操作的指令。

常用的Diff算法有两种实现方式：基于递归的Diff算法和基于循环的Diff算法。

> ​	基于递归的Diff算法简单直观，但在处理大型虚拟DOM树时可能存在性能问题。
>
> ​	基于循环的Diff算法采用迭代的方式，通过循环遍历虚拟DOM树的节点，以较低的时间复杂度找出差异。

​	虚拟DOM和差异算法是前端开发中用于优化页面渲染性能的重要技术。虚拟DOM提供了一种高效的内存中的DOM表示，而差异算法则能够准确地比较两个虚拟DOM树之间的差异，最小化对真实DOM的操作，从而提高页面的性能和响应速度。



#### 什么是虚拟 dom ？有什么用？

​	虚拟DOM（Virtual DOM）是一种在内存中构建和操作的虚拟的DOM树。它是对真实DOM的一种抽象和模拟，用JavaScript对象表示整个DOM结构及其属性。

​	虚拟DOM的基本思想是将页面的状态（数据）和视图（DOM）分离，通过对虚拟DOM的操作来更新视图，而不是直接操作真实的DOM。当数据发生变化时，会通过比较新旧虚拟DOM的差异（Diff算法），找出需要更新的部分，然后将更新应用到真实DOM上，从而保持页面和数据的同步。

虚拟DOM的主要优点包括：

> 1. 性能优化：通过将DOM操作集中在虚拟DOM上，减少了直接操作真实DOM的次数，从而提高了性能。虚拟DOM可以批量更新真实DOM，避免了频繁的重绘和回流。
> 2. 跨平台能力：虚拟DOM可以在不同的平台上运行，例如浏览器、移动端、服务器端等。这样可以实现一次编写，多平台复用。
> 3. 方便的UI组件化：虚拟DOM可以将整个页面划分为组件，每个组件都有自己的虚拟DOM。这样可以实现组件的高度复用和模块化开发。
> 4. 简化复杂的UI更新逻辑：通过比较新旧虚拟DOM的差异，可以精确地找出需要更新的部分，避免全量更新整个DOM树，从而简化了复杂的UI更新逻辑。

​	虚拟DOM通过在内存中构建和操作DOM的抽象表示，提供了一种高效、跨平台、组件化的方式来管理和更新页面的视图。它是现代前端框架（如React、Vue等）中的重要概念，能够提升开发效率和页面性能。

##### 虚拟DOM的解析过程:

虚拟DOM的解析过程可以简单分为三个步骤：**创建**、**更新**和**渲染**。

> 1. 创建虚拟DOM（Virtual DOM）：在应用程序初始化时，通过使用特定的语法或调用相关API创建虚拟DOM。虚拟DOM是使用JavaScript对象表示整个DOM结构及其属性，它包括节点类型、标签名、属性、子节点等信息。
> 2. 更新虚拟DOM：当应用程序的状态（数据）发生变化时，需要更新虚拟DOM以反映这些变化。这个过程通常由框架或库内部的更新机制自动处理，它会根据新的数据生成新的虚拟DOM，并与旧的虚拟DOM进行比较，找出差异（Diff算法）。
> 3. 渲染虚拟DOM：在更新虚拟DOM后，需要将最新的虚拟DOM渲染到真实的DOM上，以更新页面的显示。这个过程通常由框架或库提供的渲染引擎负责处理，它会根据虚拟DOM的变化，将变化部分应用到真实DOM上，从而更新页面的内容。

具体来说，虚拟DOM的解析过程如下：

> 1. 创建虚拟DOM：根据应用程序的需求，通过特定的语法或调用相关API创建虚拟DOM。这可以是手动创建的，也可以是由框架或库自动生成的。
> 2. 更新虚拟DOM：当应用程序的状态发生变化时，生成新的虚拟DOM，并与旧的虚拟DOM进行比较，找出差异（Diff算法）。差异通常包括节点的增、删、改、移等操作。
> 3. 应用差异：将差异应用到真实DOM上，更新页面的显示。这个过程可以通过操作真实DOM进行，也可以通过一些优化技术（例如批量更新、异步渲染等）来提高性能。
> 4. 渲染页面：根据最新的虚拟DOM，将其渲染到真实DOM上，更新页面的内容。这可以是整个虚拟DOM的渲染，也可以是部分虚拟DOM的渲染，具体取决于差异的范围。

​	虚拟DOM的解析过程是一个通过创建、更新和渲染虚拟DOM来实现页面更新的过程。它通过比较差异，减少了对真实DOM的频繁操作，提高了性能和用户体验。



#### diff 算法

​	Diff算法是虚拟DOM更新的核心算法，它用于比较新旧虚拟DOM之间的差异，并根据差异进行最小化的DOM操作，以提高更新效率。

​	Diff算法的基本原理是通过逐层比较新旧虚拟DOM的节点，找出节点之间的差异，并记录下来。它遵循以下几个基本规则：

> 1. 同层比较：Diff算法会逐层比较新旧虚拟DOM的节点，并将它们进行一一对比。如果节点类型不同，直接替换节点；如果节点类型相同但属性不同，更新节点的属性；如果节点类型和属性都相同，继续比较其子节点。
> 2. 列表比较：当比较的节点是列表（数组）类型时，Diff算法会使用特定的算法进行列表比较，例如使用唯一的key来标识列表项，以避免整个列表重新渲染。
> 3. 唯一标识：Diff算法会使用唯一的标识（通常是key属性）来识别节点，以便在比较过程中准确定位节点的位置，从而进行精确的差异对比和操作。
> 4. 递归比较：Diff算法会递归地比较节点的子节点，以确保所有层级的差异都能被发现和处理。

​	通过以上规则，Diff算法能够在比较的过程中找出两个虚拟DOM之间的差异，并生成一个差异对象（或称为补丁），记录下需要进行的DOM操作，例如插入、删除、更新等。然后，这个差异对象可以被应用到真实的DOM上，进行最小化的DOM操作，以实现页面的更新。

​	Diff算法的优点是能够准确地识别差异，并最小化DOM操作，提高页面更新的效率。然而，它也有一些限制和性能方面的考虑，例如当比较的虚拟DOM结构非常复杂时，Diff算法的性能可能会下降，需要合理地设计虚拟DOM结构和优化更新策略来提升性能。

​	Diff算法是虚拟DOM实现高效更新的核心，它通过比较新旧虚拟DOM之间的差异，实现最小化的DOM操作，从而提高页面更新的效率。



#### Vue 中 key 的作用

在Vue中，key是用于识别VNode（虚拟DOM节点）的特殊属性。它的作用主要有以下几个方面：

> 1. 提供唯一标识：每个VNode都应该具有唯一的key值，用于在diff算法中准确地识别VNode节点的变化。通过key，Vue可以精确地判断哪些VNode是新增的、哪些是删除的，从而最小化DOM操作，提高页面更新的效率。
> 2. 优化列表渲染：当使用v-for指令进行列表渲染时，每个列表项都应该提供一个唯一的key值。这样，Vue可以基于key的变化来确定列表项的新增、删除、移动等操作，从而避免重新渲染整个列表，提高列表渲染的性能。
> 3. 维持组件状态：当使用key在动态组件或条件渲染中切换组件时，key的变化可以强制Vue销毁旧组件并创建新组件。这样做可以保持组件的状态和避免重用旧组件的状态，确保组件能够正确地更新和重新渲染。

​	需要注意的是，key的值应该是稳定且唯一的。在使用v-for渲染列表时，推荐使用具有唯一性的属性值作为key，如ID或其他唯一标识符。避免使用索引作为key，因为索引在列表发生变化时可能会导致错误的渲染结果。

​	Vue中的key属性用于唯一标识VNode，它在diff算法中起着重要的作用，能够提高页面更新的效率、优化列表渲染，并维持组件的状态。合理使用key可以确保Vue能够准确地识别和处理VNode的变化，提供更好的性能和用户体验。



### Vue2 与 Vue3的区别

#### 生命周期：

> - Vue2：Vue2的生命周期包括beforeCreate、created、beforeMount、mounted、beforeUpdate、updated、beforeDestroy、destroyed等钩子函数。这些钩子函数允许我们在组件的不同生命周期阶段执行特定的操作，例如在created钩子函数中进行数据初始化，在mounted钩子函数中操作DOM元素等。
> - Vue3：Vue3保留了大部分Vue2的生命周期钩子函数，但也引入了新的钩子函数如setup。setup钩子函数用于代替Vue2中的beforeCreate和created钩子函数，它提供了更灵活的组合式函数编程方式，可以更好地封装组件逻辑和复用代码。



#### Diff算法：

> - Vue2：Vue2使用基于虚拟DOM的Diff算法来计算需要更新的最小操作，然后将这些操作应用于真实的DOM树。Vue2的Diff算法会比较新旧虚拟DOM树的差异，然后只对差异部分进行更新，以减少不必要的DOM操作，提高性能和渲染效率。
> - Vue3：Vue3在Diff算法方面进行了优化。首先，Vue3使用了基于Proxy的响应式系统，通过代理对象来监听数据的变化，从而减少了对getter和setter的劫持，提升了性能。其次，Vue3引入了静态标记，即在编译阶段对模板进行静态分析，标记出静态节点，从而避免在Diff算法中对这些静态节点进行比较，进一步提高了性能和渲染效率。



#### 数据响应式原理：

> - Vue2：Vue2使用Object.defineProperty实现数据的响应式。当数据被访问或修改时，Vue2通过劫持数据的get和set方法来追踪数据的变化。这样一来，当数据发生变化时，Vue2能够检测到变化并通知相关的组件进行更新。
> - Vue3：Vue3使用Proxy对象实现数据的响应式。Proxy对象可以代理目标对象并拦截对目标对象的访问和修改操作。通过代理对象的监听和触发机制，Vue3能够实时地追踪数据的变化，并触发相应的更新操作，从而实现数据响应式。



#### 组件通讯：

> - Vue2：Vue2中组件通讯主要通过props和$emit进行父子组件之间的通讯。父组件通过props将数据传递给子组件，子组件通过$emit触发自定义事件来通知父组件。此外，Vue2还提供了事件总线、Vuex等方式来实现非父子组件之间的通讯。
> - Vue3：Vue3保留了Vue2的组件通讯方式，即父子组件之间通过props和emit进行通讯。而与Vue2不同的是，Vue3引入了Composition API，提供了更灵活的组合式函数编程方式来处理组件之间的通讯。通过使用Composition API中的响应式函数、上下文传递等特性，我们可以更方便地在组件之间共享状态和方法。



#### Composition API 



​	Composition API 是 Vue 3 中引入的一种新的 API 风格，用于编写组件的逻辑和复用代码。它是一种基于函数的 API，与 Vue 2 中的选项式 API（Options API）相比，Composition API 更加灵活、组合性更强，能够更好地组织和管理组件的代码。

​	使用 Composition API，我们可以将一个组件的相关逻辑聚合在一起，而不是按照选项的顺序分散在不同的生命周期钩子函数中。这使得组件的逻辑更加清晰、可读性更高，同时也方便代码的复用和测试。

​	Composition API 提供了一系列的函数和响应式的 API，例如 `setup` 函数、`ref`、`reactive`、`watch` 等，用于定义组件的状态和行为。下面是 Composition API 的一些特点和用法：

> 1. `setup` 函数：在组件中使用 `setup` 函数来定义组件的状态和行为。`setup` 函数在组件创建之前执行，可以访问组件的 props 和 context，并返回一个对象，这个对象中的属性和方法将被暴露给组件的模板部分使用。
> 2. `ref` 和 `reactive`：`ref` 和 `reactive` 是用于创建响应式数据的 API。`ref` 用于创建一个单一的响应式数据，而 `reactive` 用于创建一个包含多个属性的响应式对象。
> 3. `watch`：`watch` 函数用于监听响应式数据的变化，并在数据变化时执行相应的操作。它可以监听单个数据或多个数据，还可以设置深度监听、异步监听等。
> 4. 生命周期钩子函数：在 Composition API 中，生命周期钩子函数的命名发生了变化，例如 `beforeCreate` 和 `created` 改为了 `onBeforeMount` 和 `onMounted`。这些钩子函数可以在 `setup` 函数中使用，与其他逻辑代码一起组合。
> 5. 自定义函数：在 Composition API 中，我们可以自定义函数来封装和复用一些逻辑代码，而不必依赖于特定的生命周期钩子函数。

​	使用 Composition API，我们能够更好地组织组件的代码，提高代码的可读性和维护性。它适用于编写中小型到大型复杂组件，并且提供了更好的代码复用和测试能力。

​	Vue3在许多其他方面也进行了改进和优化，如编译优化、TypeScript支持、组合式API等。开发者在选择Vue版本时，需要根据项目需求和实际情况综合考虑，以及考虑迁移成本和团队熟悉度等因素。



### 路由 - vue-router

#### vue-router 中常用的路由模式实现原理吗

##### **hash 模式**

1. location.hash 的值实际就是 URL 中#后面的东西 它的特点在于：hash 虽然出现 URL 中，但不会被包含在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。
2. 可以为 hash 的改变添加监听事件

```javascript
window.addEventListener("hashchange", funcRef, false);
```

每一次改变 hash（window.location.hash），都会在浏览器的访问历史中增加一个记录利用 hash 的以上特点，就可以来实现前端路由“更新视图但不重新请求页面”的功能了

> 特点：兼容性好但是不美观

##### **history 模式**

利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法。

这两个方法应用于浏览器的历史记录站，在当前已有的 back、forward、go 的基础之上，它们提供了对历史记录进行修改的功能。这两个方法有个共同的特点：当调用他们修改浏览器历史记录栈后，虽然当前 URL 改变了，但浏览器不会刷新页面，这就为单页应用前端路由“更新视图但不重新请求页面”提供了基础。

> 特点：虽然美观，但是刷新会出现 404 需要后端进行配置



#### 路由守卫

路由守卫是在路由导航过程中进行拦截和控制的功能。

Vue Router 提供了以下几种类型的路由守卫：

> 1. 全局前置守卫（Global Before Guards）：
>    - `beforeEach(to, from, next)`：在每个路由跳转之前调用，可以用来进行全局的前置验证或处理逻辑。
> 2. 路由独享的守卫（Per-Route Guards）：
>    - `beforeEnter(to, from, next)`：在某个特定路由配置中定义的守卫，只会对该路由生效。
> 3. 组件内的守卫（In-Component Guards）：
>    - `beforeRouteEnter(to, from, next)`：在进入路由对应的组件之前调用，可以访问组件实例，但此时组件实例还没有被创建。
>    - `beforeRouteUpdate(to, from, next)`：在当前路由组件复用时调用，例如在同一路由下切换不同的参数。
>    - `beforeRouteLeave(to, from, next)`：在离开当前路由组件时调用，可以阻止离开或在离开前进行一些处理。

这些守卫函数接收三个参数：

- `to`：即将进入的目标路由对象
- `from`：当前导航正要离开的路由对象
- `next`：函数，用于进入下一个守卫或确认导航

在守卫函数中，可以通过调用 `next()` 方法来进行导航控制：

- 调用 `next()` 进行正常导航
- 调用 `next(false)` 中止当前导航
- 调用 `next('/path')` 或 `next({ path: '/path' })` 进行重定向导航

使用路由守卫可以实现诸如登录验证、权限控制、页面访问限制等功能，提供了灵活且强大的路由导航控制机制。 [Vue Router 的官方文档](https://router.vuejs.org/guide/advanced/navigation-guards.html)



### Vue相关

####  vue 内置指令

![vue内置指令](../images/Vue.js%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0/vue-command.jpg)



#### v-if 和 v-show 的区别

`v-if` 和 `v-show` 都是 Vue 中的条件渲染指令，用于根据条件来控制元素的显示和隐藏。它们之间的区别如下：

> 1. 编译时机：`v-if` 是在编译阶段进行条件判断，如果条件为假，那么该元素及其子元素将不会被编译和渲染到 DOM 中。而 `v-show` 是在编译阶段将元素渲染到 DOM 中，然后通过 CSS 的 `display` 属性控制元素的显示和隐藏。
> 2. 切换消耗：由于 `v-if` 是在编译阶段进行条件判断，当条件发生变化时，会触发元素的创建或销毁，因此切换时的消耗较大。而 `v-show` 只是通过修改 CSS 的 `display` 属性来切换元素的可见性，所以切换时的消耗较小。
> 3. 初始渲染开销：由于 `v-if` 是在编译阶段进行条件判断，当条件为假时，元素及其子元素不会被编译和渲染，所以初始渲染的开销较小。而 `v-show` 在初始渲染时，会将元素及其子元素都渲染到 DOM 中，然后通过 CSS 控制其可见性，所以初始渲染的开销较大。
> 4. 条件切换频率：如果需要频繁切换元素的显示和隐藏，建议使用 `v-show`，因为它只是通过修改 CSS 属性来切换，性能更好。如果切换频率较低，可以使用 `v-if`，因为它在条件为假时会销毁元素，可以减少页面中的 DOM 元素数量。

​	`v-if` 适用于条件切换频率较低的情况，可以节省初始渲染开销和内存占用；`v-show` 适用于频繁切换元素的可见性，可以减少切换消耗。根据具体的需求和场景选择合适的条件渲染指令。



#### v-if 和 v-for 哪个优先级更高

​	在 Vue 中，`v-for` 指令的优先级高于 `v-if` 指令。这意味着当一个元素同时使用了 `v-if` 和 `v-for`，`v-for` 将首先被解析和执行，然后在每个迭代的元素上进行 `v-if` 的条件判断。

​	具体来说，当使用 `v-if` 和 `v-for` 同时存在于同一个元素上时，Vue 的编译器会先处理 `v-for`，根据数据集合生成对应的元素列表。然后，在每个生成的元素上，会再依次应用 `v-if` 的条件判断。这意味着，如果 `v-if` 的条件为假，对应的元素将不会被渲染到最终的 DOM 中。

​	`v-for` 会先根据 `items` 数据集合生成对应的元素列表，然后在每个元素上应用 `v-if` 的条件判断。只有当 `item.visible` 的值为真时，对应的元素才会被渲染到最终的 DOM 中。

​	需要注意的是，在某些特定情况下，使用 `v-if` 和 `v-for` 同时存在可能会导致性能问题，因为每次迭代都需要进行条件判断。在这种情况下，可以考虑使用计算属性或过滤器来预先筛选数据集合，以减少渲染的元素数量，从而提升性能。



#### slot（插槽）的作用

​	插槽（Slot）是 Vue 中一种用于扩展组件内容的机制。通过插槽，你可以在组件的模板中预留出一些位置，然后在使用该组件时，将内容插入到这些位置上。

插槽的作用主要有以下几个方面：

> 1. 内容分发：插槽允许组件的使用者向组件中传递内容，并在组件内部进行渲染。使用插槽可以将组件设计得更加灵活，使得组件可以接受不同的内容进行渲染，从而满足不同的需求。
> 2. 组件组合：通过插槽，你可以将多个组件组合在一起，形成更复杂的组合组件。插槽使得组件之间的组合变得简单，你可以将多个组件的内容组合在一起，并且可以在父组件中决定如何组合它们。
> 3. 默认内容：插槽可以设置默认内容，当使用组件时没有提供具体内容时，将会使用默认的插槽内容进行渲染。这样可以确保即使没有传入内容，组件仍然可以正常显示一些默认的内容。

​	在 Vue 中，有两种类型的插槽：具名插槽和默认插槽。具名插槽允许你为插槽指定名称，并在组件中根据名称进行内容分发。默认插槽是没有名称的，当组件中没有具名插槽时，会将内容分发到默认插槽中。

​	通过使用插槽，你可以将组件的结构和样式与具体的内容进行解耦，提高了组件的可复用性和灵活性。它是 Vue 中非常强大和常用的特性之一。



#### 关于 Vue 的单向数据流

​	数据总是从父组件传到子组件，子组件没有权利修改父组件传过来的数据，只能请求父组件对原始数据进行修改。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

> 注意：在子组件直接用 v-model 绑定父组件传过来的 prop 这样是不规范的写法 开发环境会报警告

​	如果实在要改变父组件的 prop 值 可以再 data 里面定义一个变量 并用 prop 的值初始化它 之后用$emit 通知父组件去修改。



#### computed 和 watch 的区别和运用的场景

​	computed 是[计算属性](https://juejin.cn/post/6956407362085191717)，依赖其他属性计算值，并且 computed 的值有缓存，只有当计算值变化才会返回内容，它可以设置 getter 和 setter。

​	[watch](https://juejin.cn/post/6954925963226382367)监听到值的变化就会执行回调，在回调中可以进行一些逻辑操作。

​	计算属性一般用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑计算属性原理详解 



#### vue 中使用了哪些设计模式

>1.工厂模式 - 传入参数即可创建实例
>
>虚拟 DOM 根据参数的不同返回基础标签的 Vnode 和组件 Vnode
>
>2.单例模式 - 整个程序有且仅有一个实例
>
>vuex 和 vue-router 的插件注册方法 install 判断如果系统存在实例就直接返回掉
>
>3.发布-订阅模式 (vue 事件机制)
>
>4.观察者模式 (响应式数据原理)
>
>5.装饰模式: (@装饰器的用法)
>
>6.策略模式 策略模式指对象有某个行为,但是在不同的场景中,该行为有不同的实现方案-比如选项的合并策略

#### 

#### Vue 模板编译原理

Vue 的模板编译原理主要包括以下步骤：

> 1. 解析：Vue 的模板编译器将模板字符串解析为抽象语法树（AST）。AST 是一个树状结构，表示了模板中的各个节点和它们之间的关系。
> 2. 优化：在解析完成后，编译器会对生成的 AST 进行优化。这个优化过程包括静态节点标记、静态根节点提升和事件侦听器的缓存等。这些优化可以提高渲染性能和减少运行时的开销。
> 3. 代码生成：在优化完成后，编译器会根据 AST 生成渲染函数。渲染函数是一个 JavaScript 函数，它接收数据作为参数，返回一个虚拟 DOM 树。渲染函数可以将模板中的数据和逻辑转换为实际的 DOM 操作。
> 4. 组件化编译：如果模板中包含组件，编译器会递归地对组件进行编译。这样可以将组件的模板编译为渲染函数，并生成组件的渲染逻辑。

​	在运行时，Vue 实例会通过编译后的渲染函数生成虚拟 DOM，并将其与实际的 DOM 进行比对，只更新需要改变的部分，以提高性能。

​	Vue 的模板编译原理将模板字符串解析为 AST，经过优化后生成渲染函数，然后在运行时使用渲染函数生成虚拟 DOM，并进行差异比对来更新视图。这种编译的过程使得 Vue 具有高效的渲染性能和灵活的组件化开发能力。

####  Vue 的性能优化

Vue 的性能优化可以从以下几个方面考虑：

> 1. 减少不必要的重新渲染：Vue 使用响应式系统来跟踪数据的变化并更新视图。为了减少不必要的重新渲染，可以使用合理的计算属性和侦听器，避免不必要的计算和更新。另外，使用 v-if 和 v-show 来条件渲染元素，只渲染当前需要显示的部分。
> 2. 列表渲染优化：在使用 v-for 渲染列表时，使用唯一的 key 属性来提高性能。Vue 使用 key 来跟踪每个节点的身份，以便在更新过程中进行重用和重新排序，而不是完全重新创建和销毁 DOM 节点。
> 3. 懒加载和异步组件：对于大型的页面或组件，可以使用懒加载和异步组件来延迟加载和渲染。这可以加快初始加载时间并减少首屏渲染的工作量。
> 4. 使用 v-if 和 v-for 的选择：在需要根据条件动态渲染的元素上使用 v-if，而不是在列表上使用 v-for。因为 v-if 在条件不满足时会完全销毁和重建元素，而 v-for 只是在数据发生变化时更新元素。
> 5. 使用虚拟列表或无限滚动：当需要处理大量数据列表时，可以使用虚拟列表或无限滚动的技术来提高性能。这样可以减少一次性渲染的节点数量，只渲染可见部分，从而减少内存占用和渲染时间。
> 6. 合理使用异步更新：Vue 提供了 nextTick 方法和 $nextTick 实例方法来在下次 DOM 更新周期之后执行回调。合理使用异步更新可以将多个更新合并成一次，减少不必要的计算和渲染。
> 7. 基于路由的代码分割：通过合理的路由配置和动态导入，将页面的代码拆分成更小的块，按需加载和渲染，提高初始加载速度和页面切换的响应性。
> 8. 合理使用缓存：对于一些计算开销较大的结果或静态数据，可以使用缓存来避免重复计算或请求。例如，使用计算属性的缓存选项或使用工具库进行数据缓存。

