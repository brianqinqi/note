#### vue的methods中的方法调方法，this指向问题

```javascript
methods: {
	fn1(){
        console.log(this)//this:Vue对象
    },
    fn2(){
        this.$options.methods.fn1()//此时fn1中的this为this.methods
        this.$options.methods.fn1.call(this)()//修改fn1中的this为Vue对象
    }
}
```



