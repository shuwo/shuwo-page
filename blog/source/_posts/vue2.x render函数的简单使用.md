# vue2.x render函数的简单使用
vue render函数[官方链接](https://cn.vuejs.org/v2/guide/render-function.html).
## 简介
我们写的模板最终都会转化为render函数，render函数即为vue模板渲染函数,简单用法如下：

有个通过 **level** prop 动态生成heading 标签的组件

```
<test-component :level="1">Hello world!</test-component>
```
很容易联想到如下简单的实现方式：


```/Users/Eric/Desktop/blog/vue2.x render函数-2017.8.3/vue2.x render函数的简单使用.md
<template>
    <div>
        <h1 v-if="level === 1">
            <slot></slot>
        </h1>
        <h2 v-else-if="level === 2">
            <slot></slot>
        </h2>
        <h3 v-else-if="level === 3">
            <slot></slot>
        </h3>
        <h4 v-else-if="level === 4">
            <slot></slot>
        </h4>
        <h5 v-else-if="level === 5">
            <slot></slot>
        </h5>
        <h6 v-else-if="level === 6">
            <slot></slot>
        </h6>
    </div>
</template>

<script>
export default {
    name:'TestComponent',
    props:{
    	level:{
    		type:Number,
    		required:true
    	}
    }
}
</script>

```
但是这样看起来有点繁琐，现在通过render函数简化以上代码

```
export default {
    props: {
        level: {
            type: Number,
            required: true
        }
    },
    render: function (createElement) {
        return createElement(
            'h' + this.level,   // tag name 标签名称
            this.$slots.default // 子组件中的阵列
        )
    }
}
```
## 实战篇
拿element ui 1.3.7 版本的 el-col组件来讲解，[具体用法](http://element.eleme.io/1.3/#/zh-CN/component/layout)

属性参数作用如下：

| 参数      | 说明          | 类型      | 可选值                           | 默认值  |
|---------- |-------------- |---------- |--------------------------------  |-------- |
| span | 栅格占据的列数 | number | — | — |
| offset | 栅格左侧的间隔格数 | number | — | 0 |
| push |  栅格向右移动格数 | number | — | 0 |
| pull |  栅格向左移动格数 | number | — | 0 |
| xs | `<768px` 响应式栅格数或者栅格属性对象 | number/object (例如： {span: 4, offset: 4}) | — | — |
| sm | `≥768px` 响应式栅格数或者栅格属性对象 | number/object (例如： {span: 4, offset: 4}) | — | — |
| md | `≥992` 响应式栅格数或者栅格属性对象 | number/object (例如： {span: 4, offset: 4}) | — | — |
| lg | `≥1200` 响应式栅格数或者栅格属性对象 | number/object (例如： {span: 4, offset: 4}) | — | — |
| tag | 自定义元素标签 | string | * | div |
源码如下：

```javascript
export default {
  name: 'ElCol',
  
  props: {
    span: {                 //栅格占据的列数
      type: Number,
      default: 24
    },
    tag: {
      type: String,
      default: 'div'
    },
    offset: Number,         //栅格左侧的间隔格数
    pull: Number,           //栅格向左移动格数
    push: Number,           //栅格向右移动格数
    xs: [Number, Object],   //响应式栅格
    sm: [Number, Object],
    md: [Number, Object],
    lg: [Number, Object]
  },

  computed: {
  	//gutter是栅格间隔，由于是父组件的props
  	//所以通过$parent访问父组件获取gutter值
    gutter() {
      let parent = this.$parent;
      //一直往上查询父组件是不是ElRow
      while (parent && parent.$options.componentName !== 'ElRow') {
        parent = parent.$parent;
      }
      return parent ? parent.gutter : 0;
    }
  },
  //render就是模版渲染函数
  render(h) {
  	 //栅格的很多属性都是通过class和style来实现的
  	 //classList：存储栅格的class
  	 //style：存储栅格的style
    let classList = [];
    let style = {};
    
    //如果存在gutter，就设置栅格两边间距为gutter的一半
    if (this.gutter) {
      style.paddingLeft = this.gutter / 2 + 'px';
      style.paddingRight = style.paddingLeft;
    }
	 
	 //'span', 'offset', 'pull', 'push'
	 //这四个props值最终作用是通过设置class来实现的
    ['span', 'offset', 'pull', 'push'].forEach(prop => {
      if (this[prop]) {
        classList.push(
          prop !== 'span'
          ? `el-col-${prop}-${this[prop]}`
          : `el-col-${this[prop]}`
        );
      }
    });
		
	 //'xs', 'sm', 'md', 'lg'
	 //这四个响应式也是通过class来实现
    ['xs', 'sm', 'md', 'lg'].forEach(size => {
      if (typeof this[size] === 'number') {
        classList.push(`el-col-${size}-${this[size]}`);
      } else if (typeof this[size] === 'object') {
        let props = this[size];
        Object.keys(props).forEach(prop => {
          classList.push(
            prop !== 'span'
            ? `el-col-${size}-${prop}-${props[prop]}`
            : `el-col-${size}-${props[prop]}`
          );
        });
      }
    });
    
    //render函数第一个参数是要生成哪种标签，
    //第二个参数是一个包含模板相关属性的数据对象
    //第三个参数是子节点 (VNodes)
    return h(this.tag, {
      class: ['el-col', classList],
      style
    }, this.$slots.default);
  }
};
```








