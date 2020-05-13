### 1. 导入方法

1. 传统下载导入
   - 下载、导入框架
   - 创建Vue实例，指定实例对象控制的区域和数据等
2. Vue-CLI安装导入
   - 安装Vue-CLI
   - vue create project-name

### 2. 创建Vue实例

```javascript
let vue = new Vue({
    el: '#app'
});
```

### 3. 实例的数据与方法

```javascript
let vue = new Vue({
  el: '#example',
  data: data
})

vue.$data === data // => true
vue.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vue.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vue.a` 改变后调用，用于监听a的变化
})
```

### 4. 实例生命周期钩子

- 每个 Vue 实例在被创建时都要经过一系列的初始化过程，在这个过程中也会运行一些叫做**生命周期钩子**的函数，这给了用户在不同阶段添加自己的代码的机会。

- 初始化(new Vue()时)：beforeCreate/created
- 创建vue>$el：beforeMount/mounted
- data被修改时：beforeUpdate/updated
- 调用vue.$destroy时：beforeDestroy/destroyed

![生命周期](images/%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

### 5. 模板语法

#### 5.1 插值

- {{}}，对应data中的数据
- 可以写文本，{{msg}}
- 可以写原始HTML，但是不安全，不建议使用
- 可以写JavaScript表达式，{{number + 1}}
- 不能写代码片段

#### 5.2 指令

- v-bind

  - 用于响应式地更新HTML 属性

  - ```html
    <a v-bind:href="url">...</a>
    // 简写
    <a :href="url">...</a>
    ```

- v-on

  - 用于监听 DOM 事件

  - ```html
    <a v-on:click="doSomething">...</a>
    // 简写
    <a @click="doSomething">...</a>
    ```

### 6. 计算属性(computed)

- 模板中插值{{}}没有代码提示，不建议放逻辑运算与JS代码

- 可以用计算属性先做逻辑运算

- ```javascript
  <div id="example">
    <p>"{{ message }}"</p>
    <p>"{{ reversedMessage }}"</p>
  </div>
  
  var vue = new Vue({
    el: '#example',
    data: {
      message: 'Hello'
    },
    computed: {
      // 计算属性的 getter
      reversedMessage: function () {
        // `this` 指向 vue 实例
        return this.message.split('').reverse().join('')
      }
    }
  })
  ```

#### 6.1 计算属性与方法(methods)

- 计算属性能做的，方法也能做，但是计算属性能缓存，节约性能
- 计算属性所计算的属性未发生改变，就不会重新计算
- 如果不希望缓存时，要用方法

#### 6.2 计算属性与侦听属性(watch)

- 当数据改变时，可以用watch监听，也可以用computed监听

- 当要监听的数据随着其他数据变化时，用computed

- ```javascript
  <div id="demo">{{ sum }}</div>
  
  let vue = new Vue({
  	el: '#demo',
  	data: {
  		a: 1,
          b: 2,
          sum: a + b
  	},
      watch: {
          //需要同时监听a和b达到监听sum的目的
      },
      computed:{
          //当a/b发生变化时，计算属性都会重新计算
          //用这个需要将data中的sum删除
          sum: function(){return this.a + this.b}
      }
  });
  ```

