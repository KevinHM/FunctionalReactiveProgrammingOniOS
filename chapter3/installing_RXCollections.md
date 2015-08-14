# 使用RXCollections

我的朋友RobRix使用OC写了一个优秀的高阶函数的库叫做[RXCollections](https://github.com/robrix/RXCollections) (译者注：目前这个项目作者已经停止维护，取而代之是RobRix的另外一个项目[Reducers](https://github.com/robrix/Reducers))

首先，我们需要一个可以展示的Xcode工程，创建一个新工程“Playground”。选择"Single View Application"作为模板。我们将在AppDelegate中展示绝大部分代码。在本书中，我将使用"FRP"作为类的前缀。

其次，我们需要在工程中导入RXCollections.我将使用Cocoapods导入这个库，这会让事情变得简单。使用如下命令以确保你的电脑里安装了最新的cocoapods。
```
sudo gem install cocoapods
```
终端出现提示的时候输入你的root密码。一旦cocoapods已经安装好了，使用`cd`导航到刚刚新建的工程目录下，并在终端输入如下指令:
```
pod init
```
这将会在当前目录下为你生成一个新的文件`Podfile`.内容大致如下:

```
#Uncomment this line to define a global platform for your project
#platform :ios, "6.0" (这里为m.n 取决于工程的设置)

target "Playground" do

end

target "PlaygroundTests" do

end

```
用你最习惯的文本编译器(我猜是Vim),取消`#platform :ios,"6.0" `的注释标示，并添加 `pod 'RXCollections' , '~> 1.0'`到"Playground"下。
```
platform :ios, "6.0"

target "Playground" do

pod 'RXCollections', '~> 1.0'

end

target "PlaygroundTests" do

end

```
好了！保存并退出编辑器，回到终端并输入:
```
pod install
```
这将导入RXCollections到工程中，同时为你提供一个新的xcode workspace文件。关闭当前xcode工程，用刚刚生成的workspace文件打开工程。

在Appdelegate.m文件中引入如下头文件:
```
#import <RXCollections/RXCollection.h>
```
在`application:didFinishLaunchingWithOptions:`方法中，创建一个我们之前讲到的数组。
```
NSArray * array = @[ @1, @2 , @3 ];
```
好了，万事具备，开始染手函数式编程！










