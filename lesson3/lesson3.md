# 第三章 微信小程序框架(自定义组件)
## 创建自定义组件
* 小程序组件需要4个文件(.js .json .wxml .wxss)，和页面类似。
* 首先在json中声明这是个自定义组件
  ```
    {
      "component": true
    }
  ```
* 然后在wxml文件中编写组件模版，在wxss文件中写组件样式
  * 在组件模板中提供了slot节点，用于承担组件引用时插入的子节点
  ```
    <!-- 组件模板 -->
    <view class="wrapper">
      <view>这里是组件的内部节点</view>
      <slot></slot>
    </view>

    <!-- 引用组件的页面模版 -->
    <view>
      <component-tag-name>
        <!-- 这部分内容将被放置在组件 <slot> 的位置上 -->
        <view>这里是插入到组件slot中的内容</view>
      </component-tag-name>
    </view>
  ```
  * 默认情况下只能有一个slot节点，如果需要多slot，需要在组件js中声明
  ```
    Component({
      options: {
        multipleSlots: true // 在组件定义时的选项中启用多slot支持
      },
      properties: { /* ... */ },
      methods: { /* ... */ }
    })
  ```
  * 声明后可以在组件wxml文件中使用多个slot节点，用name属性区分，在页面模板使用组件时，用slot属性对应不同的slot节点
  ```
  <!-- 组件模板 -->
  <view class="wrapper">
    <slot name="before"></slot>
    <view>这里是组件的内部细节</view>
    <slot name="after"></slot>
  </view>

  <!-- 引用组件的页面模版 -->
  <view>
    <component-tag-name>
      <!-- 这部分内容将被放置在组件 <slot name="before"> 的位置上 -->
      <view slot="before">这里是插入到组件slot name="before"中的内容</view>
      <!-- 这部分内容将被放置在组件 <slot name="after"> 的位置上 -->
      <view slot="after">这里是插入到组件slot name="after"中的内容</view>
    </component-tag-name>
  </view>
  ```
  * 在组件的 wxss 文件写样式时注意
    * 只对组件wxml内的节点生效
    * 不能使用id选择器、属性选择器([a])、标签名选择器，改用class选择器。
    * 组件和引用组件的页面中使用后代选择器在一些极端条件下会有非预期的表现，如遇，换选择器。
    * 子元素选择器只能用于view组件与其子节点之间，用于其他组件可能会出现非预期的情况。
    * 继承样式，如 font 、 color ，会从组件外继承到组件内，比如在app.wxss、引用组件的页面中定义的继承样式。
    * 除继承样式外， app.wxss 中的样式、引用组件的页面的样式对自定义组件无效。
  * 组件可以指定它所在节点的默认样式，在组件的wxss文件中使用 :host 选择器
  ```
  /* 组件 component.wxss */
  :host {
    color: yellow;
  }
  ```
* 最后在自定义组件的js文件中使用 Component() 来注册组件，并提供组件的属性定义、内部数据和自定义方法。
```
Component({
  properties: {
    // 这里定义了innerText属性，属性值可以在组件使用时指定，其实就是从页面传过来的值
    innerText: {
      type: String,
      value: 'default value',
    }
  },
  data: {
    // 这里是一些组件内部数据
    someData: {}
  },
  methods: {
    // 这里是一个自定义方法
    customMethod: function(){}
  }
})
```
* 注意: 
  * Component 构造器构造的组件也可以作为页面使用。
  * 使用 this.data 可以获取内部数据和属性值，但不要直接修改它们，应使用 setData 修改。
  * 生命周期函数无法在组件方法中通过 this 访问到。
  * 属性名应避免以 data 开头，即不要命名成 dataXyz 这样的形式，因为在 WXML 中， data-xyz="" 会被作为节点 dataset 来处理，而不是组件属性。
  * 在一个组件的定义和使用时，组件的属性名和data字段相互间都不能冲突（尽管它们位于不同的定义段中）。
* [Component构造器详细定义](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/custom-component/component.html)

## 使用自定义组件
* 使用已注册的组件时，需要在页面的json文件中声明。
```
{
  "usingComponents": {
    "component-tag-name": "path/to/the/custom/component"
  }
}
```
* 这样在页面的wxml中就可以使用自定义组件。
```
<view>
  <!-- 以下是对一个自定义组件的引用 -->
  <component-tag-name inner-text="Some text"></component-tag-name>
</view>
```

* 注意
  * 自定义组件的标签名只能是小写字母、中划线和下划线的组合。
  * 自定义组件也可以引用自定义组件，类似页面引用组件，使用 usingComponents 字段。
  * 自定义组件和使用自定义组件的页面所在的项目的根目录名不能以"wx-"为前缀，否则报错。
  * 旧版本的基础库（1.5.x）不支持自定义组件，此时，引用自定义组件的节点会变为默认的空节点。

## 组件事件
* 事件系统是组件间交互的主要形式。自定义组件可以触发任意的事件，引用组件的页面可以监听这些事件。
```
<!-- page.wxml -->
<!-- 当自定义组件触发“myevent”事件时，调用“onMyEvent”方法 -->
<component-tag-name bindmyevent="onMyEvent" />
<!-- 或者可以写成 -->
<component-tag-name bind:myevent="onMyEvent" />

// page.js
Page({
  onMyEvent: function(e){
    e.detail // 自定义组件触发事件时提供的detail对象
  }
})
```

* 自定义组件触发事件时，需要使用 triggerEvent 方法，指定事件名、detail对象和事件选项
  * 就是在自定义组件里触发page页面的事件
  ```
    <!-- 在自定义组件wxml中 -->
    <button bindtap="onTap">点击这个按钮将触发“myevent”事件</button>

    // 在自定义组件js
    Component({
      properties: {}
      methods: {
        onTap: function(){
          var myEventDetail = {} // detail对象，提供给事件监听函数
          var myEventOption = {} // 触发事件的选项
          this.triggerEvent('myevent', myEventDetail, myEventOption)
        }
      }
    })
  ```
  * 触发事件的选项(myEventOption): 

  | 选项名 | 类型 | 必填 | 默认值 | 描述 |
  | :------ | :------ | :------ | :------ | :------ |
  | bubbles | Boolean | 否 | false | 事件是否冒泡 |
  | composed | Boolean | 否 | false | 事件是否可以穿越组件边界，为false时，事件将只能在引用组件的节点树上触发，不进入其他任何组件内部 |
  | capturePhase | Boolean | 否 | false | 事件是否拥有捕获阶段 |

  * bubbles: 为true时事件从下到上在节点树上传递
  * composed: 为true时事件会传递到其他组件内部

## behaviors
### 定义和使用 behaviors
* behaviors 是用于组件间代码共享的特性，类似于一些编程语言中的“mixins”或“traits”。
* 每个 behavior 可以包含一组属性、数据、生命周期函数和方法，组件引用它时，它的属性、数据和方法会被合并到组件中，生命周期函数也会在对应时机被调用。每个组件可以引用多个 behavior 。 behavior 也可以引用其他 behavior 。
* behavior 需要使用 Behavior() 构造器定义。
```
// my-behavior.js
module.exports = Behavior({
  behaviors: [],
  properties: {
    myBehaviorProperty: {
      type: String
    }
  },
  data: {
    myBehaviorData: {}
  },
  attached: function(){},
  methods: {
    myBehaviorMethod: function(){}
  }
})
```
* 组件使用behavior时需要在behaviors字段中将它们逐个列出。
```
// my-component.js
var myBehavior = require('my-behavior')
Component({
  behaviors: [myBehavior],
  properties: {
    myProperty: {
      type: String
    }
  },
  data: {
    myData: {}
  },
  attached: function(){},
  methods: {
    myMethod: function(){}
  }
})
```
* 这个时候，这个组件就合并了myBehavior里的属性、数据和方法。
  * 注意: 当组件触发 attached 生命周期时，会依次触发 my-behavior 中的 attached 生命周期函数和 my-component 中的 attached 生命周期函数。

### 字段的覆盖和组合规则
* 组件和它引用的 behavior 中可以包含同名的字段，处理规程如下:
  * 对于同名的属性和方法，组件本身的属性和方法会覆盖behavior。如果有多个behavior，在组件的behaviors字段里定义的数组中，靠后的behavior的属性和方法会覆盖靠前的。
  * 对于同名的数据字段，如果是对象，会进行合并，如果不是对象，则相互覆盖。
  * 生命周期函数不会相互覆盖，而是在对应触发时机被逐个调用。如果同一个 behavior 被一个组件多次引用，它定义的生命周期函数只会被执行一次。

### 内置 behaviors
* 自定义组件可以通过引用内置的 behavior 来获得内置组件的一些行为。

```
Component({
  behaviors: ['wx://form-field']
})
```

* wx://form-field 代表一个内置 behavior ，它使得这个自定义组件有类似于表单控件的行为。
* 内置 behavior 往往会为组件添加一些属性。在没有特殊说明时，组件可以覆盖这些属性来改变它的 type 或添加 observer 。

## 组件间关系
### 定义和使用组件间关系
* 有时需要组件嵌套组件，相互间的通讯就比较复杂，为了解决这个问题，微信在 Component() 里加入了 relations 字段。
```
<!-- page.wxml -->
<custom-ul>
  <custom-li> item 1 </custom-li>
  <custom-li> item 2 </custom-li>
</custom-ul>


// 父组件custom_ul
Component({
  relations: {
    './custom_li': {
      type: 'child', // 当前是父组件,关联目标是子组件
      linked: function(target) {
        // 每次有custom-li被插入时执行，target是该节点实例对象，触发在该节点attached生命周期之后
      },
      linkChanged: function(target) {
        // 每次有custom-li被移动后执行，target是该节点实例对象，触发在该节点moved生命周期之后
      },
      unlinked: function(target) {
        // 每次有custom-li被移除时执行，target是该节点实例对象，触发在该节点detached生命周期之后
      }
    }
  ,
  methods: {
    _getAllLi: function(){
      // 使用getRelationNodes可以获得nodes数组，包含所有已关联的custom-li，且是有序的
      var nodes = this.getRelationNodes('path/to/custom-li')
    }
  },
  ready: function(){
    this._getAllLi()
  }
})

// 子组件custom_li
Component({
  relations: {
    './custom-ul': {
      type: 'parent', // 当前是子组件,关联目标是父组件
      linked: function(target) {
        // 每次被插入到custom-ul时执行，target是custom-ul节点实例对象，触发在attached生命周期之后
      },
      linkChanged: function(target) {
        // 每次被移动后执行，target是custom-ul节点实例对象，触发在moved生命周期之后
      },
      unlinked: function(target) {
        // 每次被移除时执行，target是custom-ul节点实例对象，触发在detached生命周期之后
      }
    }
  }
})

``` 

### 关联一类组件
* 有时有这样的结构
```
<custom-form>
  <view>
    input
    <custom-input></custom-input>
  </view>
  <custom-submit> submit </custom-submit>
</custom-form>
```

* 这个时候如果 custom-input 组件和 custom-submit 组件有同一个behavior，则在 custom-form 组件的 relations 中可以用这个behavior来代替组件路径作为关联的目标节点
```
// custom-form-controls.js
module.exports = Behavior({
  // ...
})

// custom-input.js 
var customFormControls = require('./custom-form-controls')
Component({
  behaviors: [customFormControls],
  relations: {
    './custom-form': {
      type: 'ancestor', // 关联的目标节点应为祖先节点
    }
  }
})

// custom-submit.js
var customFormControls = require('./custom-form-controls')
Component({
  behaviors: [customFormControls],
  relations: {
    './custom-form': {
      type: 'ancestor', // 关联的目标节点应为祖先节点
    }
  }
})

// custom-form.js
var customFormControls = require('./custom-form-controls')
Component({
  relations: {
    'customFormControls': {
      type: 'descendant', // 关联的目标节点应为子孙节点
      target: customFormControls
    }
  }
})

```

* [组件间关系详细设置](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/custom-component/relations.html)