# vue3-learn

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

### 学习总结点

#### 1. setup
1. `setup`函数包含两个几个参数-`props`和`context`，`props`参数不能进行使用解构赋值，否则会使得该`props`变成非响应式对象，如果需要解构的话，可以使用`toRefs`这个API
2. `context`表示组件上下文，包含`attrs,slots,emit`三个属性
*  `setup`函数执行之后，组件的实例并没有创建，因此在该函数内只能访问`props,attrs,slots,emit`几个属性对象，而像实例初始化之后才能访问的对象`data,methods,computed`，无权限访问，`this`在该函数中也并不是代表组件实例对象。
```js
import { toRefs, ref, reactive } from 'vue'
export default {
  setup(props, context) {
    const { title } = toRefs(props)
    const refNumber = ref(0)
    const reactiveNum = reactive({
      number: 99
    })
    return {
      refNumber,
      reactiveNum
    }
  }
  // context不是响应式对象，因此可以直接解构
  setup(props, { attrs, slots, emits }) {

  }
}
```
3. `setup`也可以返回一个`render`函数
```js
import { h, ref, reactive } from 'vue'

export default {
  setup() {
    const readersNumber = ref(0)
    const book = reactive({ title: 'Vue 3 Guide' })
    // Please note that we need to explicitly expose ref value here
    return () => h('div', [readersNumber.value, book.title])
  }
}
```

#### 2. lifecycle-生命周期
* 由于vue3.0现在的写法偏向于函数式，因此生命周期du变成了函数，在原来的基础上加了一个`On`前缀，并且去除了`beforeCreated`和`created`两个生命周期，因为`setup`函数基本上就是在这两个生命周期阶段执行的

```js
onBeforeMount
onMounted
onBeforeUpdate
onUpdated
onBeforeUnmount
onUnmounted
onErrorCaptured
onRenderTracked
onRenderTriggered

export default {
  setup() {
    onMounted(() => {
      console.log('component mounted')
    })
  }
}
```

#### 3. provide-inject——依赖注入
* vue2.2.0提供的一个新的api，主要用于上级组件向层层嵌套的子组件传递数据，类似于react中的`context`上下文的概念，主要的语法是：
1. `provide`选项是一个对象或者一个返回对象的函数，这个对象包含可以注入子孙组件的属性，这个对象的属性名可以用`Symbol`对象代替，value就是需要传递的数据；
2. `inject`选项是一个字符串数组，或者是一个对象，对象的key是本地的绑定名，而value是`provide选项提供的key的名称`或`{ from: provide提供的key值,default: 降级情况下使用的value }`
* 使用方式
```js
// 父级组件provider
var Fprovider = {
  provide: {
    foo: 'bar'
  }
}
// 子组件注入inject
var Cinject = {
  inject: ['foo'],
  created() {
    console.log(this.foo) // 'bar'
  }
}
// Symbol作为key
const s = Symbol()

const Provider = {
  provide () {
    return {
      [s]: 'foo'
    }
  }
}

const Child = {
  inject: { s },
  // ...
}
```
3. 在Vue2.2.1以上版本，注入的值会在`props`和`data`初始化之前得到，因此可以使用注入值初始化这两个选项里面的数据
```js
const Child = {
  inject: ['foo'],
  props: {
    bar: {
      default () {
        return this.foo
      }
    }
  }
}
```
4. 为`inject`选项提供默认值，非基础类型的数据需要提供工厂方法进行包裹返回
```js
const Child = {
  inject: {
    foo: {
      from: 'bar',
      default: () => [1, 2, 3]
    }
  }
}
```
* 3.0中对原有的api进行了封装，参数传递和之前类似
```js
import { InjectionKey, provide, inject } from 'vue'

const key: InjectionKey<string> = Symbol()

provide(key, 'foo') // providing non-string value will result in error

const foo = inject(key) // type of foo: string | undefined
// 如果没有使用InjectionKey定义key的类型，需要在inject时显示的定义key的类型
const foo = inject<string>(key)
```

#### 4. getCurrentInstance——用于获取当前的组件实例
* 该api允许获取内部组件的实例对象，并且只能在`setup`和`lifecycles`中生效执行
```js
import { getCurrentIntance } from 'vue'

const MyComponent = {
  setup() {
    const internalIntance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // access to globalProperties
  }
}
```