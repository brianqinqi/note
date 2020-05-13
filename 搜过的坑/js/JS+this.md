#### forEach中的this

```javascript
array.forEach(function(currentValue, index, arr), thisValue){
    console.log(this);
}
// this指向thisValue
// 没有thisValue，this指向windows
// 可以用箭头函数“继承”this
array.forEach((currentValue, index, arr), thisValue) => {
    console.log(this);
}
```

