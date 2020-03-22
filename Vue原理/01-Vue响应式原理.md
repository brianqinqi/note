### Vue响应式原理
#### 双向数据绑定
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

#### 实现响应式
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