# 第一章 微信小程序的代码构成
## 微信小程序基本上由4种类型的文件构成：
  1. .json后缀的json配置文件
  2. .wxml后缀的WXML模板文件
  3. .wxss后缀的WXSS样式文件
  4. .js后缀的JS脚步逻辑文件

## JSON配置
### 项目根目录下的json
* 项目根目录下的json一般有app.json和project.config.json。
* app.json就是当前小程序的全局配置，包括小程序的全部页面路径，界面表现，网络超时事件，tab表现等。
* app.json 配置项列表:

  | 属性  | 类型  | 必填  | 描述  |
  | :------ | :------ | :------ | :------ |
  | pages           | StringArray | 是 | 设置页面路径，数组第一项就是初始页面，数组的每一项都是 路径+文件名 |
  | window          | Object      | 否 | 设置默认的页面表现 |
  | tabBar          | Object      | 否 | 设置底部 tab 的表现 |
  | networkTimeout  | Object      | 否 | 设置网络超时时间 |
  | debug           | Boolean     | 否 | 设置是否开启 debug 模式 |

* [app.json详细配置](https://mp.weixin.qq.com/debug/wxadoc/dev/framework/config.html)

* project.config.json是工具配置，方便保存你的个性化设置，比如编辑器的界面颜色，编译配置等等。方便你在另外的电脑上使用个性化配置。
* [project.config.json详细配置](https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/edit.html#项目配置文件)

### page目录下各个页面目录的json
* page.json让开发者能定义各个不同页面的特定属性，比如每个页面的顶部颜色，是否允许下拉等，和app.json有点像，只不过一个是全局，一个是局部。
* **注意1**：页面的.json只能设置 window 相关的配置项，以决定本页面的窗口表现，所以无需写 window 这个键
* **注意2**：页面中配置项会覆盖 app.json 的 window 中相同的配置项。

## WXML 模板
### wxml就是html，只不过微信自己做了一些封装
#### 1. 标签名字不一样
  * 例如把原来div、span、p等标签都封装成了view、button、text等等基本标签。
  * 微信还封装了不少组件，像地图、视频、音频等组件。使用时直接写上对应的组件标签名就行，比如你需要在界面上显示地图，只需这样写:
  ```
  <map></map>
  ```
  * [组件详细内容](https://mp.weixin.qq.com/debug/wxadoc/dev/component/?t=2018228)

#### 2. 多了 wx:if 这样的属性和 {{}} 表达式
* 以前的前端开发流程中，都是用js操作DOM，做用户的交互效果，现代前端一般都是用mvvm开发模式，把渲染和逻辑分开，js只需要管理状态，通过模板语法来描述状态和页面结构的关系，小程序的框架就是这个思路。
* 我们可以通过 {{}} 将数据绑定到页面上，还可以通过if/else、for等来控制页面显示。
  * 数据绑定
  ```
    <!--wxml-->
    <view> {{message}} </view>
    // page.js
    Page({
      data: {
        message: 'Hello World!'
      }
    })
  ```
  * 列表渲染
  ```
    <!--wxml-->
    <view wx:for="{{array}}" wx:for-item="item"> {{item}} </view>
    // page.js
    Page({
      data: {
        array: [1, 2, 3, 4, 5]
      }
    })
  ```

  * 条件渲染
  ```
    <!--wxml-->
    <view wx:if="{{flag == 'WEBVIEW'}}"> WEBVIEW </view>
    <view wx:elif="{{flag == 'APP'}}"> APP </view>
    <view wx:else="{{flag == 'AppService'}}"> AppService </view>
    // page.js
    Page({
      data: {
        flag: 'AppService'
      }
    })
  ```

  * 模板 
  ```
  <!--wxml-->
  <template name="Names">
    <view>
      FirstName: {{FirstName}}, LastName: {{LastName}}
    </view>
  </template>

  <template is="Names" data="{{...staffName}}"></template>
  // page.js
  Page({
    data: {
      staffName: {FirstName: 'Jim', LastName: 'Hu'}
    }
  })
  ```

## WXSS 样式
### 和wxml一样，也是微信对css做了封装
#### 1. 新的尺寸单位rpx
* 用来适应移动端不同设备的屏幕宽度和像素比。小程序底层会根据不同设备来做换算。
#### 2. 提供了全局的样式和局部的样式
* 在根目录下的app.wxss会作用于当前小程序的所有页面，局部page.wxss仅对当前页面生效。
#### 3. 只支持部分css选择器（挺少的，程序员何苦为难程序员 TAT）
  | 选择器 | 样例 | 样例描述 |
  | :---------- | :---------- | :---------- |
  | .class           | .intro         | 选择所有拥有 class="intro" 的组件 |
  | #id              | #firstname     | 选择拥有 id="firstname" 的组件 |
  | element          | view           | 选择所有 view 组件 |
  | element, element | view, checkbox | 选择所有文档的 view 组件和所有的 checkbox 组件 |
  | ::after          | view::after    | 在 view 组件后边插入内容 |
  | ::before         | view::before   | 在 view 组件前边插入内容 |

## JS 交互逻辑
#### 1. 首先，我们要使用App()来注册一个小程序,接受一个Object参数,其中指定了小程序的生命周期函数等。
* 示例代码
```
App({
  // 监听小程序初始化
  onLaunch: function(options) {
    // Do something initial when launch.
  },
  // 监听小程序显示
  onShow: function(options) {
      // Do something when show.
  },
  // 监听小程序隐藏
  onHide: function() {
      // Do something when hide.
  },
  // 错误监听函数
  onError: function(msg) {
    console.log(msg)
  },
  // 任意函数或数据，可以用this访问
  globalData: 'I am global data'
})
```

#### 2. 然后使用Page()函数用来注册一个页面
* 示例代码
```
Page({
  data: {
    text: "This is page data."
  },
  onLoad: function(options) {
    // Do some initialize when page load.
  },
  onReady: function() {
    // Do something when page ready.
  },
  onShow: function() {
    // Do something when page show.
  },
  onHide: function() {
    // Do something when page hide.
  },
  onUnload: function() {
    // Do something when page close.
  },
  onPullDownRefresh: function() {
    // Do something when pull down.
  },
  onReachBottom: function() {
    // Do something when page reach bottom.
  },
  onShareAppMessage: function () {
   // return custom share data when user share.
  },
  onPageScroll: function() {
    // Do something when page scroll
  },
  onTabItemTap(item) {
    console.log(item.index)
    console.log(item.pagePath)
    console.log(item.text)
  },
  // Event handler.
  viewTap: function() {
    this.setData({
      text: 'Set some data for updating view.'
    }, function() {
      // this is setData callback
    })
  },
  customData: {
    hi: 'MINA'
  }
})
```

#### 3. 使用bind+事件名在wxml上绑定事件
```
<!-- wxml -->
<view>{{ msg }}</view>
<button bindtap="clickMe">点击我</button>
// page.js
Page({
  clickMe: function() {
    this.setData({ msg: "Hello World" })
  }
})
```
* **注意**: 视图层的数据必须使用 this.setData() 函数更新，直接通过 this.data 赋值来修改数据是不会反馈到视图层的，且会造成视图层和逻辑层数据不一致。