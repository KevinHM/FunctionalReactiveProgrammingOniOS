# 跟随FunctionalReactivePixels的脚步
上一节，我们很多次使用了`ReactiveCocoa`的关键部分，这里有更多的机会来使用`ReactiveCocoa`整个代码库。开始吧！

首先在我们的画廊视图控制器中实现三个不同的代理方法：`CollectionViewDataSource`、`CollectionViewDelegate`、高清图视图控制器的`PhotoViewControllerDelegate`

使用一个称之为`RACDelegateProxy`的实例，我们可以抽象委托类型的协议的任何方法实现(比如：那些返回void类型的)。

委托代理是一个称为`rac_signalForSelector:`对象的‘白板’，获取当Selector被调用时发送的新值的信号。

注意：你必须retain这个delegate对象，否则他们将会被释放，你将会得到一个`EXC_BAD_ACCESS`异常。添加下列私有属性到画廊视图控制器：

```
@property (nonatomic, strong) id collectionViewDelegate;
```
同时你也需要导入`RACDelegateProxy.h`，因为他不是ReactiveCocoa的核心部分，不包含在`ReactiveCocoa.h`中。移除`UICollectionViewDelegate`以及`FRPFullsizePhotoViewControllerDelegate`方法，追加下面的代码到`viewDidLoad`.

```
RACDelegateProxy *viewControllerDelegate = [[RACDelegateProxy alloc] 
									initWithProtocol:@protocol(FRPFullSizePhotoViewControllerDelegate)];
									
[[viewControllerDelegate rac_signalForSelector:@selector(userDidScroll:toPhotoAtIndex:) 	fromProtocol:@protocol(FRPFullSizePhotoViewControllerDelegate)] 
		subscribeNext:^(RACTuple *value){
			@strongify(self);
			[self.collectionView 
				scrollToItemAtIndexPath:[NSIndexPath indexPathForItem:[value.second integerValue] inSection:0] 
				atScrollPosition:UICollectionViewScrollPositionCenteredVertically 
				animated:NO];
		}];
		
self.collectionViewDelegate = [[RACDelegateProxy alloc] initWithProtocol:@protocol(UICollectionViewDelegate)];

[[self.collectionViewDelegate rac_signalForSelector:@selector(collectionView:didSelectItemAtIndexPath:)]
		subscribeNext:^(RACTuple *arguments) {
			@strongify(self);
			FRPFullSizePhotoViewController *viewController = [[FRPFullSizePhotoViewController alloc] initWithPhotoModels:self.photosArray currentPhotoIndex:[(NSIndexPath *)arguments.second item]];
			viewController.delegate = (id<FRPFullSizePhotoViewControllerDelegate>)viewControllerDelegate;
			
			[self.navigationController pushViewController:viewController animated:YES];
			
		}];
```

