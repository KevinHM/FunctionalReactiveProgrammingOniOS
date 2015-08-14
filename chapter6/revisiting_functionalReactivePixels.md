# 重温FunctionalReactivePixels

&nbsp;&nbsp;在我们继续研究使用MVVM来重构我们的`FunctionalReactivePixels`Demo之前，我们需要做一些准备工作。他们的登陆系统不支持我们使用500px_iOS_SDK的方式。

&nbsp;&nbsp;我们将从AppDelegate的头文件中移除`apiHelper`属性，并用下面的代码替换实现文件里执行初始化的那行代码，填上你的消费者Key和Secret.

```Objective-C
[PXRequest setConsumerKey:consumerKey consumerSecret:consumerSecret];
```

&nbsp;&nbsp;现在，任何调用`AppDelegate.apiHelper`来创建500px_API请求的地方，全部必须替换为`[PXRequest apiHelper]`.

&nbsp;&nbsp;最后，请更新你的CocoaPods文件中500px_iOS_SDK的版本号到`1.0.5`
