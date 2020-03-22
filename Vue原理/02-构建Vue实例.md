### 思路

1. 创建Vue的实例是通过new来创建，说明Vue是一个类。
   - 所以我们想使用自己的Vue，就必须定义一个名称叫做Vue的实例
2. 创建好了Vue实例，Vue就会根据指定的区域和数据，去编译渲染这个区域。
   - 所以我们需要在自己编写的Vue实例中拿到数据和控制区域，去编译渲染这个区域
   - 注意点：创建Vue实例的时候指定的控制区域可以是一个ID名称，也可以是一个Dom元素
   - 注意点：Vue实例会将传递的控制区域和数据都绑定到创建出来的实例对象上（$el/$data）

### 步骤

#### 1. 定义一个自己的Nue类

#### 2. 保存创建时候传过来的控制区域和数据

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
        // 2.根据区域和数据去编译渲染界面
        if (this.$el){
            // 定义一个新的类去编译渲染
            new Complier(this);
        }
    }
    // 判断是否是一个元素
    isElement(node){
        return node.nodeType === 1;
    }
}
```

#### 3. 根据区域和数据去渲染界面（Complier）

##### 3.1 将网页上的元素放到内存中

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

##### 3.2 利用指定的数据编译内存中的元素

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

##### 3.3 将编译好的内容重新渲染回网页上


	```javascript
	this.vm.$el.appendChild(fragment);
	```