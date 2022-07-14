# vuex

解决应用遇到多个组件共享状态时，单向数据流被破坏的问题，比如

- 多个视图共享一个状态
- 来不同的视图行为需要变更同一状态

这时我们把组件的共享状态抽取出来以一个**全局单例模式**去管理

# 概念

vuex 应用的核心就是 `store`，里面存放了状态 `state`

改变 store 中 state 的唯一途径是显式的提交(commit) mutation

可以通过 `store.state` 来获取状态对象

可以通过 `store.commit` 方法触发状态变化

在组件中可以通过 `this.$store` 来访问

```js
methods: {
  increment() {
    this.$store.commit('increment')
    console.log(this.$store.state.count)
  }
}
```

### state

state 中的数据是响应式的，在组件中读取 store 中的数据最简单的方法就是在计算属性中返回某个状态

```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

每当 count 变化时，计算属性都会重新求取，并更新相关的 DOM 视图

当 store 在根组件中注入后，所有的子组件都可以使用 `this.$store` 访问到 store 实例

```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

但组件里不可能只需要一个状态，通常来说都是需要读取各种状态完成业务逻辑，此时将每个状态都声明为计算属性是重复和冗余的，可以使用 `mapState` 辅助函数来帮助我们生成计算属性

```js
import { mapState } from 'vuex'

export default {
  // ...
  computed: {
    localComputed () { /* ... */ },
    // 使用对象展开运算符将此对象混入到外部对象中
    ...mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
  }
}
```

mapState 返回的是一个对象，但组件本身有其自己的计算属性，使用对象展开符即可将二者合并

### getter

有时候我们需要从 store 的 state 中派生一些状态，假设有多个组件都会用到这个状态，要么复制这个函数，要么写一个公共的工具函数，在各个组件里导入；但这两种方式都不好

我们可以在 vuex 中定义 `getters`，==可以认为是 store 的计算属性== getter 接受 state 作为第一个参数

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

与 state 类似，getter 会暴露为 `store.getters` 对象

getter 也可以接受其他 getter 作为第二个参数

在组件中同样通过 `this.$store.getters` 来访问

可以使用 `mapGetters` 辅助函数将 store 中的 getter 映射到局部计算属性

### mutation

vuex 中的 mutation 非常类似于事件，每个 mutation 都有一个字符串的事件类型（type）和一个回调函数（handler）

回调函数这里就是进行状态变更的地方，接受 state 作为第一个参数

可以向 store.commit 传入额外的参数作为载荷（payload）



