# 指令

上一节，我们绑定UIButton的enabled属性并不是最佳实践，因为UIButton增加了一个ReactiveCocoa的类和一条指令。在这一节中我们将介绍ReactiveCocoa的指令。实际上button的rac_command可以为我们监控enabled属性。
应用一段ReactiveCocoa的文档:
> 指令，RACCommand类的代表，创建并订阅动作的信号响应，可以很容易地实现一些用户与应用交互时的边界效果。

>指令(行为触发的)通常是UI驱动的，比如按键的点击。指令也可以通过信号自动禁用，这种禁用状态呈现在UI上就是禁用与该指令相关联的任何操作。

当你想要一次用户交互发送一个信号来响应的时候指令就很有用。指令信号对订阅了指令的这个信号而言，她之后的输出都被指令信号所处理。这有一点点混乱，在第五章我们会看到一些指令相关的实践。

现在我们用下面的代码来替代之前的在button上绑定enabled属性的代码

```
self.button.rac_command = [[RACCommand alloc] initWithEnabled:validEmailSignal
                                                signalBlock:^RACSignal *(id input){
                                                    NSLog(@"Button was pressed.");
                                                    return [RACSignal empty];
                                                }];
```

任何时候button被点击就会执行signalBlock，rac_command属性会监控使能信号validEmailSignal和button的enabled属性。(实际上，如果我们保留原来的代码，新加这一段会引起重复绑定一个属性的错误)。

另外，这里返回的[RACSignal empty]是什么东西? 呃。。。这里我们需要返回一个信号让属于RACCommand的`executionSignal`管道(pipe)下发出去。这个信号代表button按下时一些任务需要被处理。在这个处理信号没有返回一个'complete value'('empty '会立即返回一个'complete value')之前button将会保持不可用状态.因为这个例子中我们只是打印了一下，所以这里我们只返回一个empty信号。在第五章我们将继续讨论RACCommand及其用途。
