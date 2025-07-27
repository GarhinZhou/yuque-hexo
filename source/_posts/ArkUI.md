---
title: ArkUI
date: '2025-07-14 21:27:44'
updated: '2025-07-23 16:18:06'
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



## UIAbility
<font style="color:rgba(0, 0, 0, 0.9);">UIAbility划分原则与建议：</font>

<font style="color:rgba(0, 0, 0, 0.9);">UIAbility组件是系统调度的基本单元，为应用提供绘制界面的窗口。一个应用可以包含一个或多个UIAbility组件。例如，在支付应用中，可以将入口功能和收付款功能分别配置为独立的UIAbility</font>

<font style="color:rgba(0, 0, 0, 0.9);">每一个UIAbility组件实例都会在最近任务列表中显示一个对应的任务</font>

<font style="color:rgba(0, 0, 0, 0.9);">对于开发者而言，可以根据具体场景选择单个还是多个UIAbility，划分建议如下：</font>

+ <font style="color:rgb(36, 39, 40);">如果开发者希望在任务视图中看到一个任务，建议使用“一个UIAbility+多个页面”的方式，可以避免不必要的资源加载</font>
+ <font style="color:rgb(36, 39, 40);">如果开发者希望在任务视图中看到多个任务，或者需要同时开启多个窗口，建议使用多个UIAbility实现不同的功能</font>

<font style="color:rgb(36, 39, 40);">例如，即时通讯类应用中的消息列表与音视频通话采用不同的UIAbility进行开发，既可以方便地切换任务窗口，又可以实现应用的两个任务窗口在一个屏幕上分屏显示</font>

### <font style="color:rgba(0, 0, 0, 0.9);">声明配置</font>
<font style="color:rgba(0, 0, 0, 0.9);">为使应用能够正常使用UIAbility，需要在</font>[<font style="color:rgb(10, 89, 247);">module.json5配置文件</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/module-configuration-file)<font style="color:rgba(0, 0, 0, 0.9);">的</font>[<font style="color:rgb(10, 89, 247);">abilities标签</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/module-configuration-file#abilities%E6%A0%87%E7%AD%BE)<font style="color:rgba(0, 0, 0, 0.9);">中声明UIAbility的名称、入口、标签等相关信息</font>

```json
{
  "module": {
    // ...
    "abilities": [
      {
        "name": "EntryAbility", // UIAbility组件的名称
        "srcEntry": "./ets/entryability/EntryAbility.ets", // UIAbility组件的代码路径
        "description": "$string:EntryAbility_desc", // UIAbility组件的描述信息
        "icon": "$media:icon", // UIAbility组件的图标
        "label": "$string:EntryAbility_label", // UIAbility组件的标签
        "startWindowIcon": "$media:icon", // UIAbility组件启动页面图标资源文件的索引
        "startWindowBackground": "$color:start_window_background", // UIAbility组件启动页面背景颜色资源文件的索引
        // ...
      }
    ]
  }
}
```

### 回调
![](/images/670d39fd7d2042987a429928d65bbed7.png)

以UIAbility实例的冷启动为例说明：

1. 当用户启动一个UIAbility时，系统首先触发onCreate()回调告知应用该UIAbility正在被启动。紧接着，系统触发onForeground()回调将UIAbility切换到与用户交互的前台状态
2. 当用户跳转到其他应用时，系统会触发onBackground()回调将UIAbility切换到后台状态
3. 当用户退出UIAbility时，系统会触发onDestroy()回调告知应用该UIAbility将被销毁、

#### <font style="color:rgba(0, 0, 0, 0.9);">onWindowStageCreate()</font>
<font style="background-color:#FBDE28;">WindowStage是HarmonyOS中用于管理窗口阶段的类。在HarmonyOS中，每个UIAbility实例都有一个与之对应的WindowStage，它们之间是一对一的关系</font>

```arkts
import { UIAbility } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';
import { hilog } from '@kit.PerformanceAnalysisKit';


const DOMAIN_NUMBER: number = 0xFF00;


export default class EntryAbility extends UIAbility {
  // ...
  onWindowStageCreate(windowStage: window.WindowStage): void {
    // 设置WindowStage的事件订阅（获焦/失焦、切到前台/切到后台、前台可交互/前台不可交互）
    try {
      windowStage.on('windowStageEvent', (data) => {
        let stageEventType: window.WindowStageEventType = data;
        switch (stageEventType) {
          case window.WindowStageEventType.SHOWN: // 切到前台
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage foreground.`);
            break;
          case window.WindowStageEventType.ACTIVE: // 获焦状态
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage active.`);
            break;
          case window.WindowStageEventType.INACTIVE: // 失焦状态
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage inactive.`);
            break;
          case window.WindowStageEventType.HIDDEN: // 切到后台
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage background.`);
            break;
          case window.WindowStageEventType.RESUMED: // 前台可交互状态
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage resumed.`);
            break;
          case window.WindowStageEventType.PAUSED: // 前台不可交互状态
            hilog.info(DOMAIN_NUMBER, 'testTag', `windowStage paused.`);
            break;
          default:
            break;
        }
      });
    } catch (exception) {
      hilog.error(DOMAIN_NUMBER, 'testTag',
                  `Failed to enable the listener for window stage event changes. Cause: ${JSON.stringify(exception)}`);
    }
    hilog.info(DOMAIN_NUMBER, 'testTag', `%{public}s`, `Ability onWindowStageCreate`);
    // 设置UI加载
    windowStage.loadContent('pages/Index', (err, data) => {
      // ...
    });
  }
}
```

#### <font style="color:rgba(0, 0, 0, 0.9);">onWindowStageWillDestroy()</font>
<font style="color:rgba(0, 0, 0, 0.9);">在</font>[<font style="color:rgb(10, 89, 247);">UIAbility</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-uiability)<font style="color:rgba(0, 0, 0, 0.9);">实例销毁之前，系统触发</font>[<font style="color:rgb(10, 89, 247);">onWindowStageWillDestroy()</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-app-ability-uiability#onwindowstagewilldestroy12)<font style="color:rgba(0, 0, 0, 0.9);">回调。该回调在WindowStage销毁前执行，此时WindowStage可以使用。开发者可以在该回调用释放通过WindowStage获取的资源、注销WindowStage事件订阅等</font>

```arkts
import { UIAbility } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { hilog } from '@kit.PerformanceAnalysisKit';


const DOMAIN_NUMBER: number = 0xFF00;


export default class EntryAbility extends UIAbility {
  windowStage: window.WindowStage | undefined = undefined;
  // ...
  onWindowStageCreate(windowStage: window.WindowStage): void {
    this.windowStage = windowStage;
    // ...
  }


  onWindowStageWillDestroy(windowStage: window.WindowStage) {
    // 释放通过windowStage对象获取的资源
    // 在onWindowStageWillDestroy()中注销WindowStage事件订阅（获焦/失焦、切到前台/切到后台、前台可交互/前台不可交互）
    try {
      if (this.windowStage) {
        this.windowStage.off('windowStageEvent');
      }
    } catch (err) {
      let code = (err as BusinessError).code;
      let message = (err as BusinessError).message;
      hilog.error(DOMAIN_NUMBER, 'testTag', `Failed to disable the listener for windowStageEvent. Code is ${code}, message is ${message}`);
    }
  }
}
```

### 组件启动模式
在`module.json5`配置文件中的`launchType`字段配置，例如：

```json
{
  "module": {
    // ...
    "abilities": [
      {
        "launchType": "multiton",
        // ...
      }
    ]
  }
}
```

#### singleton
一个 UIAbility 就只能额外调用一个任务窗口，调用 `startAbility()`时复用已经有了的 UIAbility 实例

此时不会回调` onCreate()`，而是 `onNewWant()`

![](/images/67be915b5560b115de71512828dacb7f.gif)

#### multiton
每次调用`startAbility()`方法时，都会在应用进程中创建一个新的该类型UIAbility实例

![](/images/c6c6bc8cc18ac8cb9cbb3319ab8474fb.gif)

#### specified
指定实例模式，针对一些特殊场景使用（例如文档应用中每次新建文档希望都能新建一个文档实例，重复打开一个已保存的文档希望打开的都是同一个文档实例）

![](/images/bedb78688658b1f0b69d031997b8d3dd.png)![](/images/fc80c7d99c9f7ebe98ead45fd4d585fe.gif)

### 组件基本用法
#### 指定 UIAbility 启动页面
应用中的`UIAbility`在启动过程中，需要指定启动页面，否则应用启动后会因为没有默认加载页面而导致白屏。可以在`UIAbility`的`onWindowStageCreate()`生命周期回调中，通过`WindowStage`对象的`loadContent()`方法设置启动页面

```arkts
import { UIAbility } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';


export default class EntryAbility extends UIAbility {
  onWindowStageCreate(windowStage: window.WindowStage): void {
    // Main window is created, set main page for this ability
    windowStage.loadContent('pages/Index', (err, data) => {
      // ...
    });
  }
  // ...
}
```

#### 获取UIAbility的上下文信息
UIAbility类拥有自身的上下文信息，该信息为`UIAbilityContext`类的实例，`UIAbilityContext`类拥有`abilityInfo`、`currentHapModuleInfo`等属性。通过`UIAbilityContext`可以获取`UIAbility`的相关配置信息，如包代码路径、Bundle名称、Ability名称和应用程序需要的环境状态等属性信息，以及可以获取操作UIAbility实例的方法（如`startAbility()`、`connectServiceExtensionAbility()`、`terminateSelf()`等）

如果需要在页面中获得当前Ability的Context，可调用`getHostContext`接口获取当前页面关联的`UIAbilityContext`或`ExtensionContext`

+ 在UIAbility中可以通过this.context获取UIAbility实例的上下文信息：

```arkts
import { UIAbility, AbilityConstant, Want } from '@kit.AbilityKit';


export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 获取UIAbility实例的上下文
    let context = this.context;
    // ...
  }
}
```

+ 在页面中获取UIAbility实例的上下文信息，包括导入依赖资源context模块和在组件中定义一个context变量两个部分：

```arkts
import { common, Want } from '@kit.AbilityKit';


@Entry
@Component
struct Page_EventHub {
  private context = this.getUIContext().getHostContext() as common.UIAbilityContext;

  startAbilityTest(): void {
    let want: Want = {
      // Want参数信息
    };
    this.context.startAbility(want);
  }

  // 页面展示
  build() {
    // ...
  }
}
```

终止当前UIAbility实例，可以通过调用`terminateSelf()`方法实现

#### <font style="color:rgba(0, 0, 0, 0.9);">获取UIAbility拉起方的信息</font>
在UIAbilityB的onCreate生命周期中，获取并打印UIAbilityA的Pid、BundleName和AbilityName

```arkts
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';


export default class UIAbilityB extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 调用方无需手动传递parameters参数，系统会自动向Want对象中传递调用方信息。
    console.log(`onCreate, callerPid: ${want.parameters?.['ohos.aafwk.param.callerPid']}.`);
    console.log(`onCreate, callerBundleName: ${want.parameters?.['ohos.aafwk.param.callerBundleName']}.`);
    console.log(`onCreate, callerAbilityName: ${want.parameters?.['ohos.aafwk.param.callerAbilityName']}.`);
  }


  onDestroy(): void {
    console.log(`UIAbilityB onDestroy.`);
  }


  onWindowStageCreate(windowStage: window.WindowStage): void {
    console.log(`Ability onWindowStageCreate.`);


    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load the content, error code: ${err.code}, error msg: ${err.message}.`);
        return;
      }
      console.log(`Succeeded in loading the content.`);
    });
  }
}
```

### UIAbility组件与UI的数据同步
使用EventHub进行数据通信：

EventHub为UIAbility组件提供了事件机制，使它们能够进行订阅、取消订阅和触发事件等数据通信能力

在基类Context中，提供了EventHub对象，可用于在UIAbility组件实例内通信。使用EventHub实现UIAbility与UI之间的数据通信需要先获取EventHub对象：

1. 在UIAbility中调用`eventHub.on()`方法注册一个自定义事件“event1”，`eventHub.on()`有如下两种调用方式，使用其中一种即可

```arkts
import { hilog } from '@kit.PerformanceAnalysisKit';
import { UIAbility, Context, Want, AbilityConstant } from '@kit.AbilityKit';


const DOMAIN_NUMBER: number = 0xFF00;
const TAG: string = '[EventAbility]';


export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 获取eventHub
    let eventhub = this.context.eventHub;
    // 执行订阅操作
    eventhub.on('event1', this.eventFunc);
    eventhub.on('event1', (data: string) => {
      // 触发事件，完成相应的业务操作
    });
    hilog.info(DOMAIN_NUMBER, TAG, '%{public}s', 'Ability onCreate');
  }


  // ...
  eventFunc(argOne: Context, argTwo: Context): void {
    hilog.info(DOMAIN_NUMBER, TAG, '1. ' + `${argOne}, ${argTwo}`);
    return;
  }
}
```

2.  在UI中通过`eventHub.emit()`方法触发该事件，在触发事件的同时，根据需要传入参数信息

```arkts
import { common } from '@kit.AbilityKit';


@Entry
@Component
struct Page_EventHub {
  private context = this.getUIContext().getHostContext() as common.UIAbilityContext;


  eventHubFunc(): void {
    // 不带参数触发自定义“event1”事件
    this.context.eventHub.emit('event1');
    // 带1个参数触发自定义“event1”事件
    this.context.eventHub.emit('event1', 1);
    // 带2个参数触发自定义“event1”事件
    this.context.eventHub.emit('event1', 2, 'test');
    // 开发者可以根据实际的业务场景设计事件传递的参数
  }


  build() {
    Column() {
      // ...
      List({ initialIndex: 0 }) {
        ListItem() {
          Row() {
            // ...
          }
          .onClick(() => {
            this.eventHubFunc();
            this.getUIContext().getPromptAction().showToast({
              message: 'EventHubFuncA'
            });
          })
        }


        // ...
        ListItem() {
          Row() {
            // ...
          }
          .onClick(() => {
            this.context.eventHub.off('event1');  //取消事件订阅
            this.getUIContext().getPromptAction().showToast({
              message: 'EventHubFuncB'
            });
          })
        }
        // ...
      }
      // ...
    }
    // ...
  }
}
```

3.  在自定义事件“event1”使用完成后，可以根据需要调用eventHub.off()方法取消该事件的订阅

使用AppStorage/LocalStorage进行数据同步：

ArkUI提供了AppStorage和LocalStorage两种应用级别的状态管理方案，可用于实现应用级别和UIAbility级别的数据同步。使用这些方案可以方便地管理应用状态，提高应用性能和用户体验。其中，AppStorage是一个全局的状态管理器，适用于多个UIAbility共享同一状态数据的情况；而LocalStorage则是一个局部的状态管理器，适用于单个UIAbility内部使用的状态数据。通过这两种方案，开发者可以更加灵活地控制应用状态，提高应用的可维护性和可扩展性

<font style="color:rgba(0, 0, 0, 0.9);">详细请参见</font>[<font style="color:rgb(10, 89, 247);">应用级变量的状态管理</font>](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-application-state-management-overview)

### UIAbility组件间交互（设备内）
#### 启动应用内的UIAbility
1. 在EntryAbility中，通过调用`startAbility()`方法启动UIAbility，want为UIAbility实例启动的入口参数，其中bundleName为待启动应用的Bundle名称，abilityName为待启动的Ability名称，moduleName在待启动的UIAbility属于不同的Module时添加，parameters为自定义信息参数

```arkts
.onClick(() => {
  // context为Ability对象的成员，在非Ability对象内部调用需要
  // 将Context对象传递过去
  let wantInfo: Want = {
    deviceId: '', // deviceId为空表示本设备
    bundleName: 'com.samples.stagemodelabilitydevelop',
    moduleName: 'entry', // moduleName非必选
    abilityName: 'FuncAbilityA',
    parameters: {
      // 自定义信息
      info: '来自EntryAbility Page_UIAbilityComponentsInteractive页面'
    },
  };
  // context为调用方UIAbility的UIAbilityContext
  this.context.startAbility(wantInfo).then(() => {
    hilog.info(DOMAIN_NUMBER, TAG, 'startAbility success.');
  }).catch((error: BusinessError) => {
    hilog.error(DOMAIN_NUMBER, TAG, 'startAbility failed.');
  });
```

2. 在FuncAbility的`onCreate()`或者onNewWant()生命周期回调文件中接收EntryAbility传递过来的参数

```arkts
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit';

export default class FuncAbilityA extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    // 接收调用方UIAbility传过来的参数
    let funcAbilityWant = want;
    let info = funcAbilityWant?.parameters?.info;
  }
  //...
}
```

3. 在FuncAbility业务完成之后，如需要停止当前UIAbility实例，在FuncAbility中通过调用`terminateSelf()`方法实现

```arkts
.onClick(() => {
  let context = this.getUIContext().getHostContext() as common.UIAbilityContext; // UIAbilityContext
  // context为需要停止的UIAbility实例的AbilityContext
  context.terminateSelf((err) => {
    if (err.code) {
      hilog.error(DOMAIN_NUMBER, TAG, `Failed to terminate self. Code is ${err.code}, message is ${err.message}`);
      return;
    }
  });
})
```

4. 如需要关闭应用所有的UIAbility实例，可以调用ApplicationContext的`killAllProcesses()`方法实现关闭应用所有的进程

#### 启动应用内的UIAbility并获取返回结果
1. 在EntryAbility中，调用`startAbilityForResult()`接口启动FuncAbility，异步回调中的data用于接收FuncAbility停止自身后返回给EntryAbility的信息

```arkts
.onClick(() => {
  let context = this.getUIContext().getHostContext() as common.UIAbilityContext; // UIAbilityContext
  const RESULT_CODE: number = 1001;
  let want: Want = {
    deviceId: '', // deviceId为空表示本设备
    bundleName: 'com.samples.stagemodelabilitydevelop',
    moduleName: 'entry', // moduleName非必选
    abilityName: 'FuncAbilityA',
    parameters: {
      // 自定义信息
      info: '来自EntryAbility UIAbilityComponentsInteractive页面'
    }
  };
  context.startAbilityForResult(want).then((data) => {
    if (data?.resultCode === RESULT_CODE) {
      // 解析被调用方UIAbility返回的信息
      let info = data.want?.parameters?.info;
      hilog.info(DOMAIN_NUMBER, TAG, JSON.stringify(info) ?? '');
      if (info !== null) {
        this.getUIContext().getPromptAction().showToast({
          message: JSON.stringify(info)
        });
      }
    }
    hilog.info(DOMAIN_NUMBER, TAG, JSON.stringify(data.resultCode) ?? '');
  }).catch((err: BusinessError) => {
    hilog.error(DOMAIN_NUMBER, TAG, `Failed to start ability for result. Code is ${err.code}, message is ${err.message}`);
  });
})
```

2. 在FuncAbility停止自身时，需要调用`terminateSelfWithResult()`方法，入参abilityResult为FuncAbility需要返回给EntryAbility的信息

```arkts
.onClick(() => {
  let context = this.getUIContext().getHostContext() as common.UIAbilityContext; // UIAbilityContext
  const RESULT_CODE: number = 1001;
  let abilityResult: common.AbilityResult = {
    resultCode: RESULT_CODE,
    want: {
      bundleName: 'com.samples.stagemodelabilitydevelop',
      moduleName: 'entry', // moduleName非必选
      abilityName: 'FuncAbilityB',
      parameters: {
        info: '来自FuncAbility Index页面'
      },
    },
  };
  context.terminateSelfWithResult(abilityResult, (err) => {
    if (err.code) {
      hilog.error(DOMAIN_NUMBER, TAG, `Failed to terminate self with result. Code is ${err.code}, message is ${err.message}`);
      return;
    }
  });
})
```

3. FuncAbility停止自身后，EntryAbility通过`startAbilityForResult()`方法回调接收被FuncAbility返回的信息，RESULT_CODE需要与前面的数值保持一致

```arkts
.onClick(() => {
  let context = this.getUIContext().getHostContext() as common.UIAbilityContext; // UIAbilityContext
  const RESULT_CODE: number = 1001;

  let want: Want = {
    deviceId: '', // deviceId为空表示本设备
    bundleName: 'com.samples.stagemodelabilitydevelop',
    moduleName: 'entry', // moduleName非必选
    abilityName: 'FuncAbilityA',
    parameters: {
      // 自定义信息
      info: '来自EntryAbility UIAbilityComponentsInteractive页面'
    }
  };
  context.startAbilityForResult(want).then((data) => {
    if (data?.resultCode === RESULT_CODE) {
      // 解析被调用方UIAbility返回的信息
      let info = data.want?.parameters?.info;
      hilog.info(DOMAIN_NUMBER, TAG, JSON.stringify(info) ?? '');
      if (info !== null) {
        this.getUIContext().getPromptAction().showToast({
          message: JSON.stringify(info)
        });
      }
    }
    hilog.info(DOMAIN_NUMBER, TAG, JSON.stringify(data.resultCode) ?? '');
  }).catch((err: BusinessError) => {
    hilog.error(DOMAIN_NUMBER, TAG, `Failed to start ability for result. Code is ${err.code}, message is ${err.message}`);
  });
})
```

#### 启动UIAbility的指定页面
<font style="color:rgba(0, 0, 0, 0.9);">UIAbility的启动分为两种情况：UIAbility冷启动和UIAbility热启动。</font>

+ <font style="color:rgb(36, 39, 40);">UIAbility冷启动：指的是UIAbility实例处于完全关闭状态下被启动，这需要完整地加载和初始化UIAbility实例的代码、资源等</font>
+ <font style="color:rgb(36, 39, 40);">UIAbility热启动：指的是UIAbility实例已经启动并在前台运行过，由于某些原因切换到后台，再次启动该UIAbility实例，这种情况下可以快速恢复UIAbility实例的状态</font>

##### <font style="color:rgb(36, 39, 40);">调用方UIAbility指定启动页面</font>
<font style="color:rgb(36, 39, 40);">调用方UIAbility启动另外一个UIAbility时，通常需要跳转到指定的页面。例如FuncAbility包含两个页面（Index对应首页，Second对应功能A页面），此时需要在传入的want参数中配置指定的页面路径信息，可以通过want中的parameters参数增加一个自定义参数传递页面跳转信息</font>

```arkts
.onClick(() => {
  let context = this.getUIContext().getHostContext() as common.UIAbilityContext; // UIAbilityContext
  let want: Want = {
    deviceId: '', // deviceId为空表示本设备
    bundleName: 'com.samples.stagemodelabilityinteraction',
    moduleName: 'entry', // moduleName非必选
    abilityName: 'FuncAbility',
    parameters: { // 自定义参数传递页面信息
      router: 'funcA'
    }
  };
  // context为调用方UIAbility的UIAbilityContext
  context.startAbility(want).then(() => {
    hilog.info(DOMAIN_NUMBER, TAG, 'Succeeded in starting ability.');
  }).catch((err: BusinessError) => {
    hilog.error(DOMAIN_NUMBER, TAG, `Failed to start ability. Code is ${err.code}, message is ${err.message}`);
  });
})
```

##### 目标UIAbility冷启动
目标UIAbility冷启动时，在目标UIAbility的`onCreate()`生命周期回调中，接收调用方传过来的参数

然后在目标UIAbility的`onWindowStageCreate()`生命周期回调中，解析调用方传递过来的want参数，获取到需要加载的页面信息url，传入`windowStage.loadContent()`方法

```arkts
onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
  // 接收调用方UIAbility传过来的参数
  this.funcAbilityWant = want;
}
onWindowStageCreate(windowStage: window.WindowStage): void {
  // Main window is created, set main page for this ability
  hilog.info(DOMAIN_NUMBER, TAG, '%{public}s', 'Ability onWindowStageCreate');
  // Main window is created, set main page for this ability
  let url = 'pages/Index';
  if (this.funcAbilityWant?.parameters?.router && this.funcAbilityWant.parameters.router === 'funcA') {
    url = 'pages/Page_ColdStartUp';
  }
  windowStage.loadContent(url, (err, data) => {
    // ...
  });
}
```

##### 目标UIAbility热启动
在应用开发中，会遇到目标UIAbility实例之前已经启动过的场景，这时再次启动目标UIAbility时，不会重新走初始化逻辑，只会直接触发`onNewWant()`生命周期方法。为了实现跳转到指定页面，需要在`onNewWant()`中解析参数进行处理

1. 冷启动短信应用的UIAbility实例时，在`onWindowStageCreate()`生命周期回调中，通过调用`getUIContext()`接口获取UI上下文实例UIContext对象

```arkts
onWindowStageCreate(windowStage: window.WindowStage): void {
  // Main window is created, set main page for this ability
  hilog.info(DOMAIN_NUMBER, TAG, '%{public}s', 'Ability onWindowStageCreate');
  let url = 'pages/Index';
  if (this.funcAbilityWant?.parameters?.router && this.funcAbilityWant.parameters.router === 'funcA') {
    url = 'pages/Page_ColdStartUp';
  }

  windowStage.loadContent(url, (err, data) => {
    if (err.code) {
      return;
    }

    let windowClass: window.Window;
    windowStage.getMainWindow((err, data) => {
      if (err.code) {
        hilog.error(DOMAIN_NUMBER, TAG, `Failed to obtain the main window. Code is ${err.code}, message is ${err.message}`);
        return;
      }
      windowClass = data;
      this.uiContext = windowClass.getUIContext();
    });
    hilog.info(DOMAIN_NUMBER, TAG, 'Succeeded in loading the content. Data: %{public}s', JSON.stringify(data) ?? '');
  });
}
```

2. 在短信应用UIAbility的`onNewWant()`回调中解析调用方传递过来的want参数，通过调用UIContext中的`getRouter()`方法获取Router对象，并进行指定页面的跳转。此时再次启动该短信应用的UIAbility实例时，即可跳转到该短信应用的UIAbility实例的指定页面

```arkts
funcAbilityWant: Want | undefined = undefined;
uiContext: UIContext | undefined = undefined;

onNewWant(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    if (want?.parameters?.router && want.parameters.router === 'funcA') {
      let funcAUrl = 'pages/Page_HotStartUp';
      if (this.uiContext) {
        let router: Router = this.uiContext.getRouter();
        router.pushUrl({
          url: funcAUrl
        }).catch((err: BusinessError) => {
          hilog.error(DOMAIN_NUMBER, TAG, `Failed to push url. Code is ${err.code}, message is ${err.message}`);
        });
      }
    }
  }
```

