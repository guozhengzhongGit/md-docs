Reflect 是 es6 提供的操作对象的 api，其作用有两个

1. 将原本部署在 Object 对象上的一些明显属于语言内部的方法，放到 Reflect 上；当前某些方法将同时在 Object 对象和 Reflect 对象上同时部署，而未来的新方法将只部署在 Reflect 对象上
2. 修改某些 Object 对象方法的返回结果，让其变得更合理
3. 让 Object 操作变成函数行为而非以前的命令式，比如 key in obj 或者 delete obj[key]，取而代之的是 Reflect.has(obj, key) 和 Reflect.deleteProperty(obj, name) 等函数式方法
4. Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。所以当使用 Proxy 对某个对象进行代理拦截时，总是可以调用对应的 Reflect 对象方法确保完成原有的默认行为，然后再部署额外功能

```js
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target, name, value, receiver);
    if (success) {
      console.log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

上面代码 Proxy 方法拦截了对象 target 的属性赋值行为，于是在 set 方法内部，采用 Reflect.set 方法完成给对象属性赋值的原有行为，再部署额外的功能

与 Proxy 对象相对应，Reflect 对象也一共有 13 个静态方法，包括 Vue3 中用到的

- Reflect.get(target, key, receiver)
- Reflect.set(target, key, value, receiver)
- Reflect.has(obj, key)
- Reflect.deleteProperty(obj, key)

### Reflect.get(target, key, receiver)

查找并返回 target 对象的 key 属性，如果没有该属性，则返回 undefined

```js
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
};

var myReceiverObject = {
  foo: 4,
  bar: 4,
};

Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

### Reflect.set(target, key, value, receiver)

设置 target 对象的 key 属性值为 value

### Reflect.has(target, key)

对应 key in target 的 in 运算符

### Reflect.deleteProperty(target, key)

等同于 delete target[key]

### Reflect.ownKeys(target)

返回对象自身的所有属性