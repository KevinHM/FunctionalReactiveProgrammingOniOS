# 信号
信号是另一种类型的流。与序列流相反，信号是`push-driven`的。新的值能够通过管道发布但不能像`pull-driven`一样在管道中获取，他们所抽象出来的数据会在未来的某个时间传送过来。

这里需要理解两个概念:`pull-driven`和`push-driven`.
 > Push-driven means that values for the signal are not defined at the moment of signal creation and may become available at a later time (for example, as a result from network request, or any user input).

 > Push-driven : 在创建信号的时候，信号不会被立即赋值，之后才会被赋值(举个栗子：网络请求回来的结果或者是任意的用户输入的结果)

 > Pull-driven means that values in the sequence are defined at the moment of signal creation and we can query values from the stream one-by-one.

 > Pull-driven : 在创建信号的同时序列中的值就会被确定下来，我们可以从流中一个个地查询值。

信号发送三种类型的值：`Next Values`代表了下一个发送到管道内的值。`Error Value`代表`signal`无法成功完成,一般很少见，我们会在下一章学习怎么使用她们。`Completion Values`代表`signal`成功完成，我们也会在下一章来学习。这里要注意的是：
 > 一个事情响应中，一个`signal`(信号)发送了一个`Error value`或者一个`Completion Value`后，就不会再发送任何其他的`value`.
 错误或者成功将只会发送其中一个，绝不会有两个同时发送的情况！


信号是ReactiveCocoa的核心组件之一。ReactiveCocoa为UIKit的每一个控件内置了一套信号选择器。例如，UITextField就有一个`rac_textSignal`,UITextField中每一次按键的响应都会通过它发送出去。下一章我们会学习如何使用信号来执行任务。

信号也可以被链接(链式调用)和转化。通过映射或者过滤一个流得到的新的流也可以随后被映射、被过滤，进行所有你能想到的各种操作。下一章我们将了解更多这方面的内容。

