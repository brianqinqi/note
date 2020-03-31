### 1 Vue响应式原理

#### 1.1 双向数据绑定
- 通过原生JS的defineProperty方法，[文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- defineProperty方法可以直接在一个对象上定义一个新属性，或者修改一个对象的属性，并返回这个对象
- Object.defineProperty(obj, prop, descriptor)
	- obj: 需要操作的对象
	- prop: 需要操作的属性
	- descriptor: 属性描述符
- defineProperty 可以给某个属性添加 get/set 方法，以后获取这个属性的值就自动调用get方法，设置就自动调用set方法
	- 注意点：如果设置了 get/set 方法，就不能通过value直接赋值，不能编写writable
```javascript
let obj = {};
Object.defineProperty(obj, 'name', {
	// value: 'lnj', // 通过value确定新增属性的取值
    // 修改的属性能修改/删除/遍历
    // 新增的属性默认不能修改/删除/遍历
    // writable: true, // 此时可以修改
    configurable: true, // 此时可以删除
    enumerable: true, // 此时可以遍历
    get(){
    	return oldValue;
    },
    set(newValue){
    	if(oldValue !== newValue){
    		oldValue = newValue
    		// 此时属性发生了改变，可以编写修改UI代码
    	}
    }
})
console.log(obj) // {name: 'lnj'}
```

#### 1.2 实现响应式
- 需求：监听对象中所有属性的变化
- 思路：将需要监听的对象传递给Observer这个类，这个类就给传入的对象所有属性都添加get/set 方法
``` javascript
let obj = {
	name: 'qq',
	age: 26
};
class Observer{
    // 只要将需要监听的那个对象传递给这个类
    // 这个类就能快速给传入对象的所有属性都添加get/set方法
	constructor(data){
		this.observer(data);
	}
	observer(obj){
		if(obj && typeof obj === 'object'){
			for(let key in obj){
				this.defineReactive(obj, key, obj[key]);
			}
		}
	}
    // obj:需要操作的对象
    // arrt:需要新增get/set方法的属性
    // value:需要新增get/set方法的属性取值
	defineReactive(obj, attr, value){
		// 当对象的属性取值是一个对象时，也需要给这个对象的所有属性添加get/set 方法
		this.observer(value);
		Object.defineProperty(obj, attr, {
			get(){
				return value;
			},
			set:(newValue)=>{
				if(value !== newValue){
					value = newValue;
					console.log('监听到数据变化，修改UI');
					// 将对象的属性从基本类型改成对象时，需要给新对象添加get/set 方法
					this.observer(value);
				}
			}
		})
	}
}
new Observer(obj); // 此时就能监听obj属性的变化
```

---

### 2. 构建Vue实例

#### 2.1 思路

1. 创建Vue的实例是通过new来创建，说明Vue是一个类。
   - 所以我们想使用自己的Vue，就必须定义一个名称叫做Vue的实例
2. 创建好了Vue实例，Vue就会根据指定的区域和数据，去编译渲染这个区域。
   - 所以我们需要在自己编写的Vue实例中拿到数据和控制区域，去编译渲染这个区域
   - 注意点：创建Vue实例的时候指定的控制区域可以是一个ID名称，也可以是一个Dom元素
   - 注意点：Vue实例会将传递的控制区域和数据都绑定到创建出来的实例对象上（$el/$data）

#### 2.2 步骤

##### 2.2.1 定义一个自己的Nue类

##### 2.2.2 保存创建时候传过来的控制区域和数据

```javascript
class Nue {
    constructor(options){
        // 1.保存创建时候传过来的控制区域和数据
        if (this.isElement(options.el)){
            this.$el = options.el;
        } else {
            this.$el = document.querySelector(options.el);
        }
        this.$data = options.data;
        this.proxyData();
        this.$methods = options.methods;
        // 2.根据区域和数据去编译渲染界面
        if (this.$el){
            // 定义一个新的类去编译渲染
            new Complier(this);
        }
    }
    // 将数据保存到Vue实例中
    proxyData(){
        for (let key in this.$data){
            Object.defineProperty(this, key, {
                get: () => {
                    return this.$data[key];
                }
            })
        }
    }
    // 判断是否是一个元素
    isElement(node){
        return node.nodeType === 1;
    }
}
```

##### 2.2.3 根据区域和数据去渲染界面（Compiler）

###### 1.将网页上的元素放到内存中

   ```javascript
   node2fragment(app){
       // 1.创建一个空的文档碎片对象
       let fragment = document.createDocumentFragment();
       // 2.遍历循环取到每一个元素
       let node = app.firstChild;
       while (node) {
           // 只要将元素添加到文档碎片对象中，那么这个元素会自动从网页上消失
           fragment.appendChild(node);
           node = app.firstChild;
       }
       // 3.返回存储了所以元素的文档对象
       return fragment;
   }
   ```

   - [关于文档片段接口DocumentFragment](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment)

###### 2.利用指定的数据编译内存中的元素

   ```javascript
   buildTemplate(fragment){
   	let nodeList = [...fragment.childNodes];
       nodeList.forEach(node=>{
           // 1.判断当前遍历到的节点是元素还是文本
           // 1.1元素：判断有没有v-model属性
           // 1.2文本：判断有没有{{}}的内容
           if (this.vm.isElement(node)){
               // 元素
               this.buildElement(node);
               // 处理子元素(后代)
               this.buildTemplate(node);
           } else {
               // 文本
               this.buildText(node);
           }
       })
   }
   // 处理元素
   buildElement(node){
       // 取出传入元素所有属性，并通过解构赋值转换成数组
   	let attrs = [...node.attributes];
       // 遍历数组
       attrs.forEach(attr=>{
           let {name, value} = attr; // v-model="name" --> name:v-model/value:name
           // 判断是否是vue的属性
           if (name.startsWith('v-')){
               // 取出v-后面的属性名称
               let [ , directive] = name.split('-'); // v-model >> [v, model]
               // 利用this.vm中的数据$data，替换对应元素node的值value
               CompilerUtil[directive](node, value, this.vm);
           }
       })
   }
   // 处理文本
   buildText(node){
       let content = node.textContent;
       let reg = /\{\{.+?\}\}/gi;
       if (reg.test(content)){
           CompilerUtil['content'](node, content, this.vm);
       }
   }
   // 定义工具方法
   let CompilerUtil = {
       // 根据名称拿到对应的数据
       getValue(vm, value){
           // time.h --> [time, h]
           // reduce() 方法对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。
           return value.split('.').reduce((data, currentKey)=>{
               // 第一次执行：data=$data, currentKey=time
               // 第一次执行：data=time, currentKey=h
               return data[currentKey];
           }, vm.$data);
       },
       getContent(vm, value){
           let reg = /\{\{(.+?)\}\}/gi; //{{name}} --> name
           let val = value.replace(reg, (...args) => {
               // console.log(args); // 数组第二个元素为name
               return this.getValue(vm, args[1]); // 根据name拿到数据并返回
           });
           return val;
       },
       // 节点，要赋值的属性，包含了数据的对象
       model: function (node, value, vm) {     
           // value = time.h
           // node.value = vm.$data[value];  
           // vm.$data[time.h] --> vm.$data[time] --> time[h]
           // 所以需要getValue来处理名称拿到对应的数据
           let val = this.getValue(vm, value);
           node.value = val;
       },
       content: function (node, value, vm) {
           // console.log(value); // {{name}} --> name --> $data[data]
           let val = this.getContent(vm, value);
           node.textContent = val;
       }
   }
   ```

######  3.将编译好的内容重新渲染回网页上


	```javascript
	this.vm.$el.appendChild(fragment);
	```
---

### 3. 数据驱动界面

- 想要实现数据变化之后更新UI界面，我们可以使用发布订阅模式来实现

- 先定义一个观察者类，再定义一个发布订阅类，然后通过发布订阅的类来管理观察者类

  ```javascript
  class Watcher {
      constructor(vm, attr, cb){
          this.vm = vm;
          this.attr = attr;
          this.cb = cb;
          // 在创建观察者的时候就去获取当前的旧值
          this.oldValue = this.getOldValue();
      }
      getOldValue(){
          // 将当前创建的观察者保存到全局上
          Dep.target = this;
          let oldValue = CompilerUtil.getValue(this.vm, this.attr);
          // 使用完后，清空
          Dep.target = null;
          return oldValue;
      }
      // 定义一个更新的方法，判断新值和旧值是否相同
      update(){
          let newValue = CompilerUtil.getValue(this.vm, this.attr);
          if (this.oldValue !== newValue) {
              // 当值改变时，就是数据发生改变，执行回调
              this.cb(newValue, this.oldValue);
          }
      }
  }
  // 发布订阅类
  class Dep {
      constructor(){
          // 这个数组用来管理某个属性所有的观察者对象
          this.subs = [];
      }
      // 订阅观察的方法
      addSub(watcher){
          this.subs.push(watcher);
      }
      // 发布订阅的方法
      notify(){
          this.subs.forEach(watcher => watcher.update());
      }
  }
  ```

1. 在第一次渲染的时候，给每一个属性添加观察者
2. 在get方法当中，把当前属性的观察者添加到发布订阅对象中管理起来
3. 当前属性发生变化之后，让发布订阅对象发布一个通知
4. 发布通知后，遍历所有的观察者对象，调用更新方法，更新UI界面

---

### 4. 界面驱动数据

1. 监听输入框的输入事件，实现界面驱动数据

```javascript
node.addEventListener('input', (e)=>{
    let newValue = e.target.value;
    // time.h 多层
    this.setValue(vm, value, newValue);
})
```

2. 有多层属性时（time.h），需要分割取到最后一个值

```javascript
setValue(vm, attr, newValue){
    attr.split('.').reduce((data, currentAttr, index, arr)=>{
        // 分割取到最后一个值
        if (index === arr.length - 1){
            data[currentAttr] = newValue;
        }
    }, vm.$data)
},
```

---

### 5. 实现事件相关指令

1. 渲染元素的时候，判断取到需要实现的事件指令和方法名称

```javascript
buildElement(node){
    let attrs = [...node.attributes];
    attrs.forEach(attr=>{
        let {name, value} = attr; // v-on:click="myFn" >> name:v-on:click/value:myFn
        if (name.startsWith('v-')){
            let [directiveName, directiveType] = name.split(':'); // [v-on, click]
            let [ , directive] = directiveName.split('-'); // v-on >> [v, on]
            // 利用this.vm中的方法$methods，
            CompilerUtil[directive](node, value, this.vm, directiveType);
            // CompilerUtil[on]
        }
    })
}
```

2. 在工具类中根据方法名称实现事件指令

```javascript
on: function (node, value, vm, type) {
    node.addEventListener(type, (e)=>{
        // Vue中所有方法的this都是Vue实例
        vm.$methods[value].call(vm, e);
    })
}
```

---

### 6. 实现相关属性

- 实现Vue中的计算属性

```javascript
this.$computed = options.computed;
// 将computed 中的方法添加到$data中
// 方便将来渲染时从$data中获取到computed中定义的计算属性
this.computed2data();
// 将computed 中的方法添加到$data中
computed2data(){
    for (let key in this.$computed){
        Object.defineProperty(this.$data, key, {
            get: () => {
                return this.$computed[key].call(this)
            }
        })
    }
}
```









