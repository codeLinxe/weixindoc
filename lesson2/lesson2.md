# 第二章 微信小程序的框架(逻辑层)
## 1. 目录结构
* 小程序包含一个描述整体程序的 app 和多个描述各自页面的 page。
* 一个小程序主体部分由三个文件组成，必须放在项目的根目录，如下：
  1. app.js： 小程序逻辑
  2. app.json： 小程序公共设置
  3. app.wxss： 小程序公共样式
* 一个小程序页面由四个文件组成，如下：
  1. js: 逻辑处理
  2. wxml: 页面结构
  3. wxss: 页面样式
  4. json: 页面设置

* **注意**：描述页面的四个文件必须具有相同的路径与文件名。

## 2. 逻辑层（App Service）
### 2.1 简介
* 小程序的逻辑层由JavaScript编写。
* 逻辑层处理数据后发送给视图层，同时接收视图层的事件反馈。
* 和通常JavaScript相比，微信做了修改：
  * 增加了 App() 和 Page() 方法，分别用来注册程序和页面。
  * 增加 getApp 和 getCurrentPages 方法，分别用来获取App实例和当前页面栈。
  * 提供微信的API，如微信用户信息、扫一扫、支付等。
  * 每个页面有独立的作用域，并提供模块化能力。
  * 由于不是浏览器环境，如 document 和 window 等会无法使用。
  * 开发者写的所有代码最终将会打包成一份 JavaScript，并在小程序启动的时候运行，直到小程序销毁。类似 ServiceWorker，所以逻辑层也称之为 App Service。
### 2.2 注册程序
* App() 函数用来注册一个小程序。接受一个 object 参数，其指定小程序的生命周期函数等。
* object 参数说明：

  | 属性 | 类型 | 描述 |
  | :------ | :------| :------|
  | onLaunch | Function | 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）|
  | onShow | Function | 当小程序启动，或从后台进入前台显示，会触发 onShow |
  | onHide | Function | 当小程序从前台进入后台，会触发 onHide |
  | onError | Function | 当小程序发生脚本错误，或者 api 调用失败时，会触发 onError 并带上错误信息 |
  | 其他 | Any | 任意的函数或数据，用this访问 |

* 前台、后台定义： 当用户点击左上角关闭，或者按了设备 Home 键离开微信，小程序并没有直接销毁，而是进入了后台；当再次进入微信或再次打开小程序，又会从后台进入前台。需要注意的是：只有当小程序进入后台一定时间，或者系统资源占用过高，才会被真正的销毁。

* onLaunch, onShow 参数，如下：

  | 字段 | 类型 | 说明 |
  | :------ | :------| :------|
  | path | String | 打开小程序的路径 |
  | query | Object | 打开小程序的query |
  | scene | Number | 打开小程序的场景值 |
  | shareTicket | String | 获取转发信息 |
  | referrerInfo | Object | 当场景为由从另一个小程序或公众号或App打开时，返回此字段 |
  | referrerInfo.appId | String | 来源小程序或公众号或App的 appId |
  | referrerInfo.extraData | Object | 来源小程序传过来的数据，scene=1037或1038时支持 |

* getApp(): 全局的 getApp() 函数可以用来获取到小程序实例。
* 注意：
  * App() 必须在 app.js 中注册，且是唯一的。
  * 不要在 App() 内定义的函数中使用 getApp() ，应该使用 this 拿到App实例。
  * 不要在生命周期函数 onLaunch 中使用 getCurrentPages() ，此时page还未生成。
  * 通过 getApp() 获取实例之后，不要私自调用生命周期函数。

### 2.3 注册页面
* Page() 函数用来注册一个页面。接受一个 object 参数，其指定页面的初始数据、生命周期函数、事件处理函数等。
* object 参数说明：

  | 属性 | 类型 | 描述 |
  | :------ | :------| :------|
  | data | Object | 页面的初始数据 |
  | onLoad | Function | 监听页面加载 |
  | onReady | Function | 监听页面初次渲染完成 |
  | onShow | Function | 监听页面显示 |
  | onHide | Function | 监听页面隐藏 |
  | onUnload | Function | 监听页面卸载 |
  | onPullDownRefresh | Function | 监听用户下拉动作 |
  | onReachBottom | Function | 页面上拉触底事件的处理函数 |
  | onShareAppMessage | Function | 用户点击右上角转发的处理函数 |
  | onPageScroll | Function | 页面滚动触发事件的处理函数 |
  | onTabItemTap | Function | 当前是tab页时，点击tab触发的函数 |
  | 其他 | Any | 任意的函数或数据，用this访问 |

* 生命周期函数
  * onLoad: 一个页面只会调用一次，可以在参数中获取当前页面调用的query参数。
  * onShow: 每次页面打开都会调用一次。
  * onReady: 一个页面只会调用一次，代表逻辑层可以和视图层进行交互。
  * onHide: 当wx.navigateTo或tab切换时调用.
  * onUnload: 当wx.redirectTo或wx.navigateBack的时候调用。

* 页面相关事件处理函数
  * onPullDownRefresh: 下拉刷新
    * 需要在app.json的window选项中或page.json中开启enablePullDownRefresh。
    * 当处理完数据刷新后，可以调用wx.stopPullDownRefresh来停止当前页面的下拉刷新。
  * onReachBottom: 上拉触底
    * 可以在app.json的window选项中或page.json中设置触发距离onReachBottomDistance。
    * 在触发距离内滑动期间，本事件只会被触发一次。
  * onPageScroll: 页面滚动
    * 监听用户滑动页面事件。
    * 参数为 Object ，包括 scrollTop 字段，值为页面在垂直方向已滚动的距离（单位px）。
  * onShareAppMessage: 用户转发
    * 只有定义了此事件处理函数，右上角菜单才会显示“转发”按钮。
    * 用户点击转发按钮的时候会调用。
    * 此事件需要 return 一个 Object，用于自定义转发内容
    * 自定义转发字段

      | 字段 | 说明 | 默认值 |
      | :------ | :------ | :------ |
      | title | 转发标题 | 当前小程序名称 |
      | path | 转发路径 | 当前页面 path ，必须是以 / 开头的完整路径 |

* 事件处理函数
  * 可以在视图层通过bind+事件名绑定事件，在事件触发后，就会执行Page()中定义的事件处理函数

* 一些特殊函数
  * Page.prototype.route
    * route 字段可以获取到当前页面的路径。
  * Page.prototype.setData()
    * setData 函数用于将数据从逻辑层发送到视图层（异步），更新视图层的数据，同时改变对应的 this.data 的值（同步）。
    * setData(data, callback)
    * 注意: 
      * 直接修改 this.data 而不调用 this.setData 是无法改变页面的状态的，还会造成数据不一致。
      * 单次设置的数据不能超过1024kB，请尽量避免一次设置过多的数据。
      * 请不要把 data 中任何一项的 value 设为 undefined ，否则这一项将不被设置并可能遗留一些潜在问题。

### 2.4 页面路由
* 在小程序中所有页面的路由全部由框架进行管理。
#### 页面栈
* 框架以栈的形式维护了当前的所有页面。 当发生路由切换的时候，页面栈的表现如下:

  | 路由方式 | 页面栈表现 |
  | :------ | :------ |
  | 初始化 | 新页面入栈 |
  | 打开新页面 | 新页面入栈 |
  | 页面重定向 | 当前页面出栈，新页面入栈 |
  | 页面返回 | 页面不断出栈，直到目标返回页，新页面入栈 |
  | Tab 切换 | 页面全部出栈，只留下新的 Tab 页面 |
  | 重加载 | 页面全部出栈，只留下新的页面 |

* getCurrentPages()
  * getCurrentPages() 函数用于获取当前页面栈的实例，以数组形式按栈的顺序给出，第一个元素为首页，最后一个元素为当前页面。
  * 注意: 不要尝试修改页面栈，会导致路由以及页面状态错误。

* 路由触发方式
  * 初始化
    * 小程序打开的第一个页面
  * 打开新页面
    * 调用 API wx.navigateTo 或使用组件 <navigator open-type="navigateTo"/>
  * 页面重定向
    * 调用 API wx.redirectTo 或使用组件 <navigator open-type="redirectTo"/>44
  * 页面返回
    * 调用 API wx.navigateBack 或使用组件<navigator open-type="navigateBack">或用户按左上角返回按钮
  * Tab 切换
    * 调用 API wx.switchTab 或使用组件 <navigator open-type="switchTab"/> 或用户切换 Tab
  * 重启动
    * 调用 API wx.reLaunch 或使用组件 <navigator open-type="reLaunch"/>
* 注意
  * navigateTo, redirectTo 只能打开非 tabBar 页面。
  * switchTab 只能打开 tabBar 页面
  * reLaunch 可以打开任意页面
  * 页面底部的 tabBar 由页面决定，即只要是定义为 tabBar 的页面，底部都有 tabBar。
  * 调用页面路由带的参数可以在目标页面的onLoad中获取。

  ### 2.5 文件作用域
  * 在 JavaScript 文件中声明的变量和函数只在该文件中有效；不同的文件中可以声明相同名字的变量和函数，不会互相影响。

  * 通过全局函数 getApp() 可以获取全局的应用实例，如果需要全局的数据可以在 App() 中设置，如：
    ```
    // app.js
    App({
      globalData: 1
    })

    // a.js
    var app = getApp()
    app.globalData++

    // b.js
    var app = getApp()
    console.log(app.globalData)
    ```

### 2.6 模块化
#### 建立模块
* 可以将一些公共js代码抽离出来作为一个模块。模块通过module.exports或exports对外界暴露接口。
* 注意
  * 推荐使用 module.exports 来暴露接口，因为 exports 是 module.exports 的一个引用，在模块里边随意更改 exports 的指向会造成未知的错误
  * 小程序不支持直接引入node_modules, 使用到 node_modules 时候拷贝出相应的代码到小程序目录
  ```
  // common.js
  function sayHello(name) {
    console.log(`Hello ${name} !`)
  }
  function sayGoodbye(name) {
    console.log(`Goodbye ${name} !`)
  }

  module.exports.sayHello = sayHello
  exports.sayGoodbye = sayGoodbye
  ```

#### 模块的使用
​* 在需要使用这些模块的文件中，使用 require(path) 将公共代码引入
```
var common = require('common.js')
Page({
  helloMINA: function() {
    common.sayHello('MINA')
  },
  goodbyeMINA: function() {
    common.sayGoodbye('MINA')
  }
})
```
* 注意: require 暂时不支持绝对路径

### 2.7 API
[API 文档](https://mp.weixin.qq.com/debug/wxadoc/dev/api/)