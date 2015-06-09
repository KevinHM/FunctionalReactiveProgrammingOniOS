# 流和序列
流是值的序列化的抽象，你可以认为一个流就像一条水管，而值就是流淌在水管中的水，值从管道的一端流入从另一端流出。当值从管道的另一端流出的时候，我们可以读取过去所有的值，甚至是刚刚进入管道的值(即当前值)。接下来让我们拭目以待！
呃，值的序列化，那是什么鬼？以我们当前的认知水平来说，她就像是一个数组，一个列表。事实上，使用`rac_sequeuece`我们能够轻松地将数组转化为一个流:
```
NSArray *array = @[ @1, @2, @3 ];
RACSequence * stream = [array rac_sequence];
```
等一下！`Sequences`？我以为我们在处理`Stream`? 好吧，说明一下，`Sequences`是两种特定类型的流的一种，实际上，`RACSequence`是一个`RACStream`的子类。
我们能用流做什么呢?好吧，我将使用流来展示上一章中提到的例子。应用在平方数映射上:
```
[stream map:^id (id value){
    return @(pow([value integerValue], 2));
}];
```
注意，跟数组一样，流不能包含nil元素。[译者注:NSArray中以nil作为结束标示，stream也一样]。
非常好！但是流映射后还是流，我们怎么样才能得到数组呢？幸运的是,`RACSequence`有一个方法返回数组:`array`。
```
NSLog(@"%@",[stream array]);
```
这会打印映射后的数组。比起直接使用`RXCollections`这多出了几个步骤，但这里我只想说明使用流也可以达成任务。

当然，我们可以合并上面的方法调用来避免污染变量的作用域.

```
NSLog(@"%@",[[[array rac_sequence] map:^id (id value){
                    return @(pow([value integerValue], 2));
                }] array]);
```

总的来说,我们做了这样的事情:
 - 将数组转化成一个序列类型的流。
 - 对流进行映射得到一个新的流。
 - 将新的流转为数组。

序列，默认情况下是延迟加载的(也称：懒加载或被动加载)，是`pull-driven`的，在他们被生成的时候就会提供确切的值，而数组方法会强制给序列的每一个成员赋值。

我们来看一下`filtering`。为了使用ReactiveCocoa来过滤我们的数组，我们需要再一次把它序列化以便于使用过滤。
```
NSLog(@"%@", [[[array rac_sequence] filter:^BOOL (id value){
                        return [value integerValue] % 2 == 0;
                    }] array]);
```
最后看一下怎么让一个序列流合并为单个值(`folding`)：

```
NSLog(@"%@",[[[array rac_sequence] map:^id (id value){
                    return [value stringValue];
                }] foldLeftWithStart:@"" reduce:^id (id accumulator, id value){
                    return [accumulator stringByAppendingString:value];
            }]);
```
这种情况下，我们在序列上进行了链式调用，当我们讨论下一节'信号'的时候，(链式调用)是一个关键的概念。

ReactiveCocoa具有左折叠和右折叠的概念。左折叠时折叠算法将从头到尾遍历数组，反之称为右折叠。这样的命名(即左、右折叠)暗示了编程语言对列表的理解，这种概念在Objective-C中是没有的。

确定你现在已经理解了到此为止我们所说的内容，这对后面将要进行的讲解非常重要。


