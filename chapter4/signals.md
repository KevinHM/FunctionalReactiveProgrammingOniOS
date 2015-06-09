# 信号
信号是另一种类型的流。与序列流相反，信号是`push-driven`的。新的值能够通过管道发布但不能像`pull-driven`一样在管道中获取，他们所抽象出来的数据会在未来的某个时间递送过来。

这里需要理解两个概念:`pull-driven`和`push-driven`.
 > Push-driven means that values for the signal are not defined at the moment of signal creation and may become available at a later time (for example, as a result from network request, or any user input).

 > Push-driven : 在创建信号的时候，信号不会被立即赋值，之后才会被赋值(举个栗子：网络请求回来的结果或者是任意的用户输入的结果)

 > Pull-driven means that values in the sequence are defined at the moment of signal creation and we can query values from the stream one-by-one.

 > Pull-driven : 在创建信号的同时序列中的值就会被确定下来，我们可以从流中一个个地查询值。

信号发送三种类型的值：
