# 高阶折叠

Flod 是一个有趣的高阶函数－她把列表中的所有元素变成一个值。一个简单的高阶折叠能够用来给数值数组求和。

```
NSNumber * sum = [array rx_foldWithBlock:^ id (id memo , id each){
    return @([memo integerValue] + [each integerValue]);
}];
```
输出的值为@6.数组中的每一个元素按顺序执行上述合并规则:`[memo integerValue] + [each integerValue]`,其中memo参数纪录的是上一次合并后的结果，其初始值为零。这还不是很有趣，有趣的是我们还能给`memo`(这个参数的泛称)赋初始值:
```
[[array rx_mapWithBlock:^id (id each){
        return [each stringValue];
    }] rx_foldInitialValue:@"" block:^id (id memo , id each){
        return [memo stringByAppendingString:each];
}];
```
代码的结果:@“123”. 我们来分析一下这是怎么做到的. 首先我们对数组中的所有NSNumber对象做了映射，把他们变成了NSString对象，然后我们实现了一个高阶折叠，并给了`memo`变量一个空字符串。

在没有RXCollections的情况下能得到这样的结果吗？当然可以。但这是一个明确的"是什么，而不是如何"的解决问题的方法。这种方法可以让我们不必跟CPU一样去想"这一步要如何，下一步要如何"类似这样的事情。写代码的时候如此，读代码的时候更是如此(意:更多地关注任务是什么，要达成什么目标)
