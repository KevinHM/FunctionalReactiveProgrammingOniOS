# 高阶过滤

谈到ReactiveCocoa，我们要使用的另一种关键的高阶函数就是过滤器。一个列表通过过滤能够返回一个只包含了原列表中符合条件的元素的新列表，具体我们来看实践中的例子:

```
NSArray *filteredArray = [array rx_filterWithBlock:^BOOL(id each){
    return ([each integerValue] % 2 == 0);
}]
```
过滤后，现在`filteredArray`等于`@[ @2 ]`.如果没有这样的抽象方法(即高阶过滤)，我们不得不像下面这样来完成工作:
```
NSMutableArray *mutableArray = [NSMutableArray arrayWithCapacity: array.count];
for ( NSNumber * number in array ){
    if ( [number integerValue] % 2 == 0 ){
        [mutableArray addObject:number];
    }
}
NSArray *filteredArray = [NSArray arrayWithArray:mutableArray];
```
有点明白了,对不对? 你可能像上面这样子写代码写了成百上千次。我们每一天的工作中涉及到类似这种高阶映射或者高阶过滤的事情有多少? 非常多！通过使用像高阶过滤、高阶映射类似的高阶函数，我们能够把这种繁琐又乏味的任务抽象出来，轻松工作，轻松生活。。。


