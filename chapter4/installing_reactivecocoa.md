# 引入ReactiveCocoa

&nbsp;&nbsp;ReactiveCocoa有两种引入的方式:使用CocoaPods或者作为项目的一个字模块(直接拽入项目中)。ReactiveCocoa官方是不支持CococaPods的，但是开源社区提供了这样的服务，我们可以使用她。如果你乐于让ReactiveCocoa作为一个子模块引入到项目中，你可以下载2.x版本并根据官方的介绍来配置她。

&nbsp;&nbsp;使用CocoaPods来引入ReactiveCocoa：打开前面我们创建的`Podfile`文件，并删除`RXCollections`行，用`pod 'ReactiveCocoa', '2.0'`替代掉。你的`Podfile`文件看起来应该是这样的:
```
platform :ios, "6.0"
target "Playground" do
pod 'ReactiveCocoa' , '2.0'
end

target "PlaygroundTests" do

pod 'ReactiveCocoa' , '2.0'
end
```

注意：我们使用的是'2.0'版的ReactiveCocoa而非最新的。重新运行`pod install`,将从项目中移除`RXCollections`并引入`ReactiveCocoa`。项目中任何`#import <RXCollections/XXXX>`的地方都会编译报错，请把他们也移除。

这一章里面，我们将把代码写在`ViewController`的实现文件中，而不是在`AppDelegate`中，所以现在请打开ViewController的实现文件。不要忘记把ReactiveCocoa引入进来 `#import <ReactiveCocoa/ReactiveCocoa.h>`。


