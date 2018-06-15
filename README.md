# Wepy 小程序vue框架

### 目录结构：
````
  --dist
  --src
   | --components
   | --mixins
   | --pages
   | --store
      | --actions
      | --reducers
      | --types
      index.js
  
  wepy.config.js  
````

### 框架使用

> `.way` 格式文件
- html：
  - `<template />` 解析成为 .wxml文件
  
- javascript:
  - `config` 对象：微信文档 页面配置
  - `js` 代码

- style:
  - `<style />` 解析为 .wxss文件
  
> 实例：
````
  import wepy from 'wepy';
  
  // App小程序实例
  export default class MyAPP extends wepy.app {
    ...
    config = {}  // 对应 app.json 文件
    ....
  }
  
  // Page页面实例
  export default class IndexPage extends wepy.page {}
  
  // Component组件实例
  export default class MyComponent extends wepy.component {}
````

> 页面实例、组件实例：

WePY中的 `methods` 属性只能声明页面 **wxml标签的bind、catch事件**，不能声明自定义方法，这与Vue中的用法是不一致的。
````
  export default class MyComponent extends wepy.component {
      methods = {
          bindtap () {
              let rst = this.commonFunc();
              // doSomething
          },
  
          bindinput () {
              let rst = this.commonFunc();
              // doSomething
          },
          
          //错误：普通自定义方法不能放在methods对象中
          customFunction () {
              return 'sth.';
          }
      }
  
      //正确：普通自定义方法在methods对象外声明，与methods平级
      customFunction () {
          return 'sth.';
      }
  
  }
````
<br/>
<br/>

---------------------------------

>数据绑定 **`.sync` && `twoWay: true`**

````
/*----------------- parent.wpy ----------------------*/
  <child 
    :title="parentTitle" 
    :syncTitle.sync="parentTitle" 
    :twoWayTitle="parentTitle"
    :syncTwoWayTitle="parentTitle"
  />
  
  data = {
      parentTitle: 'p-title'
  };
  
/*----------------- child.wpy ----------------------*/
  props = {
      // 静态传值
      title: String,
  
      // 父向子 单向 动态传值
      syncTitle: {
          type: String,
          default: 'null'
      },
  
      // 子组件 动态改变 父组件props
      twoWayTitle: {
          type: String,
          default: 'nothing',
          twoWay: true
      }
      
      // 子组件 动态改变 父组件props
      syncTwoWayTitle: {
          type: String,
          default: 'nothing',
          twoWay: true
      }
  };
  
  onLoad () {
      /**** this.props 访问属性 ****/
      console.log(this.title); // p-title
      console.log(this.syncTitle); // p-title
      console.log(this.twoWayTitle); // p-title
      console.log(this.syncTwoWayTitle); // syncTwoWayTitle
  
      /* this.$parent.parentTitle 访问父组件属性方法 */
      this.title = 'c-title';
      console.log(this.$parent.parentTitle); // p-title.
      
      /* twoWay为true时，子组件props中的属性值改变时，会同时改变父组件对应的值 */
      this.twoWayTitle = 'two-way-title';
      this.$apply();
      console.log(this.$parent.parentTitle); // two-way-title
      
      /* 直接触发父组件props改变 */
      this.$parent.parentTitle = 'p-title-changed';
      this.$parent.$apply();
      console.log(this.title); // 'c-title';
      console.log(this.syncTitle); // 'p-title-changed' --- 有.sync修饰符的props属性值，当在父组件中改变时，会同时改变子组件对应的值。
  }

````

> 组件通信与交互 `$broadcast、$emit、$invoke` 3个 API

**用于监听组件之间的通信与交互事件的事件处理函数需要写在组件和页面的events对象中**
````
  /* 触发some-event */
  this.$emit('some-event', 1, 2, 3, 4);
  
  /* event 里监听 */
  import wepy from 'wepy'
  export default class Com extends wepy.component {
      components = {};
      data = {};
      methods = {};
  
      // events对象中所声明的函数为用于监听组件之间的通信与交互事件的事件处理函数
      events = {
          'some-event': (p1, p2, p3, $event) => {
                 console.log(`${this.$name} receive ${$event.name} from ${$event.source.$name}`);
          }
      };
      // Other properties
  }
````

- 3个触发event API
  
  - $broadcast:
  
    广播event，全部下级组件（顺序请参考官方文档）
    
  - $emit:
       
    广播event，由下到上（顺序请参考官方文档）
       
  - $invoke:  
  
    是一个页面或组件对另一个组件中的方法的直接调用，通过传入组件路径找到相应的组件，然后再调用其方法。
    
    ````
    // 调用 ComA someMethod 方法，@params: someArgs
    this.$invoke('ComA', 'someMethod', 'someArgs');
    ````
