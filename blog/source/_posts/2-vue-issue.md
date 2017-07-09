---
title: Vue踩坑之路
---
使用 vue + webpack 开发项目快接近一年了，踩过大大小小的坑，一直没时间去整理，其实就是懒，今天应我们老大要求，写一篇踩坑总结，以下踩坑是基于项目实战过程中经常遇到的。

  
***
### 1. 直接用下标修改数组值时，视图没更新
实际开发，数组类型是很常用的类型，比如这里定义一个布尔值数组，绑定在一组 li 列表上，当值为 true 时为该元素绑定一个类名 active
    
    // 声明在data对象上
    arr: [true, false, false]

    // 当元素被点击，对应下标数组元素置为true，通常情况下会这么做：
    this.arr[index] = true;

此时，并未看见元素有任何变化，类 active 没被添加到该元素上。一开始以为是代码写的有问题，debugger时，数组元素的值确实被置为true，但视图并未更新，查看了vue文档，才知道由于 javascript 的限制，当利用索引直接设置一个项时，Vue 不能检测到变动的数组，以下两种方式都可以实现想要的效果：

    Vue.set(this.arr, indexOfItem, newValue);

    this.arr.splice(indexOfItem, 1, newValue)

### 2. 对象新增属性后，视图没更新
不仅数组，对象新增属性也存在视图更新问题，有时候我们需要在已经存在的对象 object 上添加一个新的属性，一种习惯性的写法就是 `this.object.newKey = 'newValue'`

如果只是单纯的想要添加一个不用显示在视图上的新属性（例如只是想将数据发送到后台），这种写法没毛病；但若想更新到视图上，该写法是无效的。Vue 不能检测到对象属性的添加或删除，由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的。

Vue 不允许在已经创建的实例上动态添加新的根级响应式属性，但可以使用 `Vue.set(object, key, value)` 方法将响应属性添加到嵌套的对象上：

    Vue.set(this.object, 'newKey', 'newValue');
    // 或者 Vue 的实例方法
    this.$set(this.object, 'newKey', 'newValue');

上述方法每次只能在对象里新增一个属性，但有时我们需要同时新增多个属性，可以使用`Object.assign()`,这种情况下需要注意:

    // 错误写法,添加到对象上的新属性不会触发更新，与上面提到的习惯性写法一样
    Object.assign(this.object, { a: 1, b: 2 })

    // 正确写法，创建一个新的对象，让它包含原对象的属性和新的属性
    this.object = Object.assign({}, this.object, { a: 1, b: 2 })

顺便一提，vue 也提供了`Vue.delete`方法来确保删除后能触发更新视图
### 3. 父组件通过prop传递对象给子组件
子组件通过 props 选项来使用父组件的数据，prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是不会反过来。这是为了防止子组件无意修改了父组件的状态，这也是 vue2.0 移除了 `.sync` 的理由(但在2.3.0+ 2又重新引入了 `.sync`修饰符)

但单向绑定的前提是传递的数据是基本数据类型或数组，我们都知道，ECMAScript 变量包含两种不同数据类型的值：基本类型和引用数据类型，如果通过 props 传递的是引用类型的 object ，当在子组件中修改object的属性值，也会同时修改父组件的数据，因为他们引用的是同个地址的变量。

如果不希望子组件修改父组件的状态，可以对该对象进行深复制，再赋值给一个计算属性：

    computed: {    
        newObject () {
            return JSON.parse(JSON.stringify(this.object));
        }
    }
### 4. 父组件传递给子组件的数据延迟问题 
用户的界面操作，可能会不断改变父组件的数据，并传递给子组件，eg: 当用户点击某一个按钮后，修改父组件的数据并`立即`调用子组件里的方法。这时候我们可能会理所当然的认为：被调用的子组件方法执行时，获得的是跟父组件同步的最新数据。事实上此时得到的是旧数据。

Vue 异步执行props数据的更新，当观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际（已去重的）工作。

为了能在数据更新之后才执行相应的回调函数，可以使用 vue 提供的 `Vue.nextTick(callback)` 方法，或者使用 `setTimeout(fn, 0)`
### 5. 访问子组件的ref属性报错：undefined
有时我们需要在 js 中直接访问子组件，为此可以使用 ref 为子组件指定一个索引 ID。

```html
    <el-dialog title="提示" v-model="visible" size="tiny">
        <el-form :model="ruleForm" ref="ruleForm">
            <el-form-item label="测试">
                <el-input v-model="ruleForm.user"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">查询</el-button>
            </el-form-item>
        </el-form>
    </el-dialog>
```
（此处以element组件为例）曾经碰到这种问题：想在每次弹框时重置表单

    this.visible = true;
    this.$refs.ruleForm.resetFields();

第一次显示弹框时，控制台报错 `Uncaught TypeError: Cannot read property 'resetFields' of undefined`，之后就没报这个错了。主要因为当 visible 设为 true 后立即访问子组件的索引时，初次渲染，组件还没挂载完成，$refs 只在组件渲染完成后才填充，并且它是非响应式的，可以使用 Vue.nextTick()，待渲染完成后再调用。

### 6. ref 与 v-for
通常我们会为子组件指定一个 String 类型的索引 id，该索引指向单独一个组件。但如果将 ref 与 v-for 合在一起使用，那么该 ref 是一个数组，且会将 ref 值相同的组件放在同一个数组里面，通过数组访问v-for出来的每个组件。还有一点需要注意：vue 会判断 ref 属性值，当为 true 时才会将索引值注册在组件上，而数字 0 的布尔值为 false，无效的索引值。

### 7. 组件上直接监听dom事件
不能在组件标签上面直接监听dom事件，要在 html 标签上面监听，组件标签上监听的事件只有在组件内部有手动 emit 才会触发。
例如 element 的组件 `<el-button @click="onSubmit"></el-button>`，click 会触发是因为组件 el-button 监听了里面的 button 标签，然后再调用 `$emit('click')` 才会触发

### 8. 使用自定义事件的表单输入组件
自定义组件使用 v-model，可以将父组件的数据传到子组件的 props 的 value 属性。然后子组件再根据 value，填充自己的内容，当输入内容更改时，再触发一个 input 事件告知父组件（不要监听数据变化进行触发，不然父组件有更改的话，会死循环），父组件会用 input 事件的数据更新父组件数据。

### 9. 自定义指令
hook方法中的el元素参数，如果模板中的组件是reuse的（例如v-for活着v-if），那么在不同组件中的指令hook方法中的el元素是同一个。

### 10. 使用字面量语法传递数值
当我们想通过 props 传递数字时，初学者可能会这样写: `<comp some-prop="1"></comp>`

因为它是一个字面 prop，得到的值是字符串 "1" 而不是 number。如果想传递一个实际的 number，需要使用 v-bind，从而让它的值被当作 JavaScript 表达式计算：`<comp :some-prop="1"></comp>`

### 11. key的使用
Vue 的设计理念之一就是尽可能高效地渲染元素，所以通常会复用已有的元素而不是从头开始渲染。当我们使用 v-if 和 v-else 指令，且两个模板使用了相同的元素，切换条件时，原先的数据不会被替换。这有时候往往不是我们想要的结果，这时只需添加一个具有唯一值的 key 属性，来声明“这两个元素是完全独立的——不要复用它们”，可以实现在数据发生变化时候重新渲染整个元素。
***

以上是这一段时间使用 vue 开发过程中遇到的坑，由于水平有限，如有补充或者写错的地方欢迎指出。

