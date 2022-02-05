---
title: 聊聊Vue组件通信方式
date: 2020-07-09 22:07:47
tags:
  - Vue
  - 组件
---

> 组件是现代前端框架最强大的功能，但是组件的作用域是相互独立的，那么不同组件之间如何传递数据呢？

![image](http://img.yanyuanfe.cn/vue.png)

<!--more-->

为了构建大型Web应用，现代前端框架都引入了组件系统，将模版和逻辑抽象为可复用的组件，可以提高项目的开发效率和可维护性。而组件间的通信一直是一个老生常谈的问题，今天来讨论一下Vue中的组件通信。

### props
通过props可以实现父组件向子组件传递数据，prop是定义在组件上的一些自定义属性，需要提前在子组件中的props声明。

``` vue
// 子组件
<template>
    <div>{{text}}</div>
</template>

<script>
    export default {
        name: 'Confirm',
        props: {
            text: {
                type: String,
                default: ''
            },
        }
    }
</script>
```

``` vue
// 父组件
<template>
    <confirm :text="文字">
</template>
```

### 自定义事件

通过自定义事件的方式可以在子组件中向父组件传递数据。

``` vue
// 子组件
<template>
    <button @click="confirm">Click</button>
</template>

<script>
    export default {
        name: 'Confirm',
        methods: {
            confirm() {
              this.$emit('confirm',"data")
            }
        }
    }
</script>
```

``` vue
// 父组件
<template>
    <confirm @confirm="handleConfirm">
</template>

<script>
    export default {
        methods: {
            confirm(data) {
              console.log(data);
            }
        }
    }
</script>
```
子组件使用this.$emit可以触发自定义事件，第一个参数是自定义事件名称，第二个参数为传递的参数（可选）。父组件使用@监听子组件的自定义事件。

### 事件总线
props只适用于父子组件之间数据的传递，兄弟组件之间的通信可以使用eventBus的方式，具体使用方式如下：

新建eventBus.js:


``` vue
import Vue from 'vue';  
export default new Vue(); 
```

触发事件的组件：


``` vue
import eventBus from './eventBus.js'; 

methods: {  
   onClick(event) {  
       eventBus.$emit('clickEvent', event.target);   
   }  
}  
```

监听事件的组件：

``` vue
import eventBus from './eventBus.js'; 

methods: {  
   onClick(event) {  
       eventBus.$on('clickEvent', (data) => {
           console.log(data);
       });   
   }  
}  
```

eventBus的方法实现了一个全局的状态管理，在任意组件之间都能非常便捷的传递数据。

### Vuex

在大型Web应用中，上述组件通信方式已经不能很好的满足实际的需求，Vue官网提供了一个全局状态管理库：Vuex。具体可参考官方文档。

### $root和$parent

在每个 new Vue 实例的子组件中，其根实例可以通过 $root property 进行访问。
所有的子组件都可以将这个实例作为一个全局 store 来访问或使用。
和 $root 类似，$parent 可以用来从一个子组件访问父组件的实例。因此可以利用$root或者$root作为桥梁在子组件之间传递数据。

触发事件的组件：


``` vue
methods: {  
   onClick(event) {  
       this.$root.$emit('clickEvent', event.target);   
   }  
}  
```

监听事件的组件：

``` vue
methods: {  
   onClick(event) {  
       this.$root.$on('clickEvent', (data) => {
           console.log(data);
       });   
   }  
}  
```

### $children

父组件也可以使用$children来访问子组件实现父子组件的通信。

this.$children是当前实例的直接子组件。需要注意 $children 并不保证顺序，也不是响应式的。

> 慎重使用 $parent 和 $children，它们的主要目的是作为访问组件的应急方法。更推荐用 props 和 events 实现父子组件通信。

``` vue
//父组件

this.$children[0].xx = xx;
```

### $refs
$refs可以创建一个子元素或者子组件的引用，使用$refs可以很方便的调用子组件的方法或者获取子组件的数据。

``` vue
// 父组件
<template>
   <base-input ref="usernameInput"></base-input>
</template>

<script>
    export default {
        methods: {
            focus() {
              this.$refs.usernameInput.focus();
            }
        }
    }
</script>
```

``` vue
// 子组件
<template>
   <input ref="input">
</template>

<script>
    export default {
        methods: {
            focus: function () {
                this.$refs.input.focus()
          }
        }
    }
</script>
```
### $attrs和$listeners

$attrs中包含了父组件传入子组件，且未在子组件props中定义的属性（class和style除外），$attrs会被自动添加到子组件的根元素上，如果你不希望组件的根元素继承 attribute，你可以在组件的选项中设置 inheritAttrs: false。


``` vue
//子组件
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```


``` vue
//父组件

<base-input
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```

同理，$listeners可以将父组件传入的所有事件监听函数指向子组件的某个特定的子元素。


``` vue
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      return Object.assign({},
        {
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```

### provide/inject

provide和inject是Vue中依赖注入的实现，provide 选项允许我们指定我们想要提供给后代组件的数据/方法。


``` vue
// 提供数据的组件
<script>
export default {
  provide() {
    return {
      form: this
    };
  },
}
</script>
```

然后在任何后代组件里，我们都可以使用 inject 选项来接收指定的我们想要添加在组件上的属性：


``` vue
<script>

export default {
  inject: ["form"],
  methods: {
    validate() {
      console.log(this.form);
    }
}
</script>
```

Vue官方的状态管理库-Vuex的原理也是使用了provide/inject。

### 总结

上述就是Vue中的组件通信方案，不同的方案适用场景不同，没有银弹，在项目开发中，需要根据项目需求来选择适合的方案。



