### 1. Vuex特点

#### 1.1 Vuex是一个插件

- 使用Vuex的时候需要用到Vue的use方法，use方法是用于注册插件的
- 所以Vuex的本质就是一个插件
- 所以实现Vuex就是在实现一个全局共享数据的插件

#### 1.2 Vuex是一个类

- 在使用Vuex的时候我们会通过new Vuex.Store创建一个仓库
- 所以需要在Vuex中新增Store属性，这个属性的取值是一个类

#### 1.3 $store属性

- 为了保证每个Vue实例中都能通过this.$store拿到仓库
- 我们需要给每个Vue实例都动态加添一个$store属性

### 2. 实现

#### 2.1 步骤

1. 三个特点
2. 实现共享数据（state：用于保存全局共享的数据）
3. 实现getters方法（getters: 保存全局共享的方法）
4. 实现mutations方法（mutations：用于保存修改全局共享数据的方法）
5. 实现actions方法（actions：用于保存 触发mutations中保存的方法 的方法）

#### 2.2 详细代码

```javascript
// 1-1 Vue.js 的插件应该暴露一个 install 方法。这个方法的第一个参数是 Vue 构造器，第二个参数是一个可选的选项对象
const install = (Vue, options)=> {
    /*
    1-3. 给每一个vue实例都添加一个$store属性:
    Vue的mixin方法：这个方法会在创建每一个Vue实例的时候执行
    通过mixin方法给每一个Vue实例都添加$store属性
    * */
    Vue.mixin({
        beforeCreate() {
            // console.log(this.$options.name);
            // Vue在创建实例的时候会先创建父组件，然后创建子组件
            // root --> App --> HelloWorld
            if (this.$options && this.$options.store) {
                // 根组件，有store，root
                this.$store = this.$options.store
            } else {
                // 不是根组件，没有$store
                // 将父组件的$store 赋值给它
                // App --> HelloWorld
                this.$store = this.$parent.$store
            }
        }
    });
};
// 1-2. 定义Store类
class Store{
    constructor(options) {
        // 2. 实现共享数据：new Store时,将进行共享的数据state添加到Store上面
        // this.state = options.state;（这个不能实时更新数据和界面）
        Vue.util.defineReactive(this, 'state', options.state);
        // 3. 实现getters方法：将传递进来的getters添加到Store上面
        this.initGetters(options);
        // 4.实现mutations方法：将传递进来的mutations添加到Store上面
        this.initMutations(options);
        // 5.实现actions方法：将传递进来的actions添加到Store上面
        this.initActions(options);
    }
        dispatch = (type, payload)=>{
        this.actions[type](payload);
    };
    initActions(options){
        // 拿到传递进来的actions
        let actions = options.actions || {};
        // 在Store上新增一个actions的属性
        this.actions = {};
        // 将传递进来的actions中的方法添加到当前Store的actions上
        for (let key in actions) {
            this.actions[key] = (payload)=> {
                actions[key](this, payload);
            }
        }
    }
    commit = (type, payload)=>{
        this.mutations[type](payload);
    };
    initMutations = (options)=>{
        // 拿到传递进来的mutations
        let mutations = options.mutations || {};
        // 在Store上新增一个mutations的属性
        this.mutations = {};
        // 将传递进来的mutations中的方法添加到当前Store的mutations上
        for (let key in mutations) {
            this.mutations[key] = (payload)=> {
                mutations[key](this.state, payload);
            }
        }
    };
    initGetters= (options)=>{
        // 拿到传递进来的getters
        let getters = options.getters || {};
        // 在Store上新增一个getters的属性
        this.getters = {};
        // 将传递进来的getters中的方法添加到当前Store的getters上
        for (let key in getters) {
            Object.defineProperty(this.getters, key, {
                get: ()=> {
                    return getters[key](this.state)
                }
            })
        }
    }
}
```



#### 2.3VueX中实现modules（模块化共享数据）

- 不同模块中可能有大量同名的数据
- 可以将不同模块中的数据放到不同的state中
- 注意点
  1. 多个模块中不能出现同名的getters方法，会报错
  2. 多个模块中的mutations，actions，中可以出现同名的方法，不会覆盖，会放到数组中依次执行
  3. 子模块中还可以嵌套子模块（main >> home >> login）

```javascript
	// store >> index.js
export default new Vuex.Store({
    // state：用于保存全局共享的数据
    state: {name: 'main'},
    // mutations：用于保存修改全局共享数据的方法
    mutations: mutations,
    // actions：用于保存 触发mutations中保存的方法 的方法
    actions: actions,
    // getters: 保存全局共享的方法
    getters: {getMainName(){}},
    // modules：用于模块化共享数据
    modules:{
        home: home,
        account: account
    }
})
let home = {
    state: {name: 'home'},
    mutations: {},
    actions: {},
    getters: {getHomeName(){}},
    modules: {}
}
```

- 使用方法和注意点：

  1. 如果获取的是模块中state共享的数据，需要加上模块的名称
  2. 如果获取的是模块中getters共享的数据，不需要加上模块的名称

  - ```javascript
    <p>{{this.$store.state.name}}</p>
    <p>{{this.$store.getters.getMainName}}</p>
    <hr>
    <p>{{this.$store.state.home.name}}</p>
    <p>{{this.$store.getters.getHomeName}}</p>
    ```

#### 2.4 安装模块数据

#### 2.5 安装模块方法