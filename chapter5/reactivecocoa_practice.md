# ReactiveCocoa的实践
这一章中我们将第一次使用ReactiveCocoa来编写一个实际的应用。我们将创建一个叫做'[500px](https://itunes.apple.com/app/500px-discover-photos-from/id471965292?mt=8)'的简单应用。'500px'类似于'[Flickr](https://itunes.apple.com/us/app/flickr/id328407587?ls=1&mt=8)',但只有你满意的照片才会被存放在那里。我使用'500px'的API的原因有两点:
 - 照片看起来非常棒
 - 当我还在那里工作的时候，我为他们的API接口写了iOS的SDK，我很熟悉她。

这一章我们分三个部分来讲解：

- 首先将完成我们的App(FunctionalReactivePixels)的基本实现。
- 其次我们将添加一些新的视图控制器，做更多的数据加载，来进一步证实第一步的实现。
- 最后我们将重新审视这个应用程序，以消除更多的状态获取使用更多函数响应型编程的机会。

这一章非常有趣，当然由我亲笔完成。我们应用程序'FunctionalReactivePixels'最后的结果开源在[Github](https://github.com/ashfurrow/FunctionalReactivePixels)上,不幸的是，创作这个App的中间一些过程并不会展现在最后的结果里，但是如果你一章一章跟着我来的话，应该会很好。


