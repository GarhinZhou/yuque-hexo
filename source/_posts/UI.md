---
title: UI
date: '2025-07-14 21:27:44'
updated: '2025-07-15 21:58:48'
---
![](/images/a23ede7aa56ae7ceb205a389a9ff7abe.png)

在鸿蒙（HarmonyOS）开发中，用于标记页面组件入口的装饰器是`@Entry`。这个装饰器用于指明当前页面的入口组件，它只能被用于被`@Component`修饰的子组件上。每个页面最多只能有一个`@Entry`装饰器

页面基础格式：

```arkts
@Component //定义组件
struct Pagename {
	Build(){
		...
	}
}
```

手动创建 page.ets 文件的话要手动给页面加路由

路由文件在：`**<font style="color:rgb(36, 39, 40);">entry > src > main > resources > base > profile > main_pages.json</font>**`

```arkts
{
  "src": [
    "pages/Index",
    "pages/Second"
  ]
}
```

<font style="color:rgb(36, 39, 40);">行高与列宽（占屏幕比例）：</font>

```arkts
Row(){
	Column(){
		...
	}.width('100%')
}.height('100%')
```

控件间距离：

```arkts

```



导入模块要在`@Entry`之前：

```arkts
import { BusinessError } from '@kit.BasicServicesKit';
```



## 页面路由
相当于用一个 url 跳转到对应页面

跳转指定界面：

```arkts
// 获取UIContext
let uiContext: UIContext = this.getUIContext();
let router = uiContext.getRouter();
// 跳转到另外一页
router.pushUrl({ url: 'pages/Second' })
```

按钮返回上一级：

```arkts
.onClick(() => {
  let uiContext: UIContext = this.getUIContext();
	let router = uiContext.getRouter();
	router.back()
})
```

## 预览
### 页面预览
在页面代码前加上：

```arkts
@Entry
```

### 组件预览
在组件前加上：

```arkts
@Preview({
  title: 'Component1',  //预览组件的名称
  deviceType: 'phone',  //指定当前组件预览渲染的设备类型，默认为Phone
  width: 1080,  //预览设备的宽度，单位：px
  height: 2340,  //预览设备的长度，单位：px
  colorMode: 'light',  //显示的亮暗模式，当前支持取值为light
  dpi: 480,  //预览设备的屏幕DPI值
  locale: 'zh_CN',  //预览设备的语言，如zh_CN、en_US等
  orientation: 'portrait',  //预览设备的横竖屏状态，取值为portrait或landscape
  roundScreen: false  //设备的屏幕形状是否为圆形
})
```

