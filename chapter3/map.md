# 高阶映射

我们要学习的第一个高阶函数是'映射[map]'.映射是在函数的层次上把一个列表变成相同长度的另一个列表，原始列表中的每一个值，在新的列表中都有一个对应的值。如下所示是一个平方数的映射：
```
map(1,2,3) => (1,4,9)
```
当然，这只是一个伪代码，一个高阶函数会返回另外一个函数而不是一个列表。那么我们要如何利用RXCollections呢?

我们这么来用rx_mapWithBlock:方法：
```
NSArray * mappedArray = [array rx_mapWithBlock:^id(id each){
    return @(pow([each integerValue],2));
}];
```
这将会达成上面伪代码所完成的任务，如果我们打印出`array`的日志，我们将会看到如下内容:
```
(
    1，
    4，
    9
)
```
简直完美!请注意`rx_mapWithBlock:` 并不是一个真正的函数映射，因为他不是技术上的高阶函数(她没有返回一个函数)。后面提到的库(RAC)已经解决了这一点,在下一章我们将看到映射是如何在ReactiveCocoa的上下文中工作的。

注意`rx_mapWithBlock:`在没有对原数组元素进行任何修改的前提下返回了一个新的数组，这里Foundation的类真的是非常好用的一个例子，因为他们的类默认就是不可变的。

想象一下，往常(命令式编程)为了完成这个任务，我们不得不写下这样的代码:
```
NSMutableArray *mutableArray = [NSMutableArray arryaWithCapacity:array.count];
for (NSNumber *number in array) [mutableArray addObject:@(pow([number integerValue], 2))];

NSArray *mappedArray = [NSArray arrayWithArray: mutableArray];

```
代码显然更多，而且还有一个无用的局部变量`mutableArray`污染了我们的作用域，简直是个毛线！

所以当你想把一个列表里的元素转化为另一个列表的元素时，你就能体会到映射的强大。
