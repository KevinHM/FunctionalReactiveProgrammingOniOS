# 和FunctionalReactivePixels一起实践
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
我们也可以在`self`上调用`rac_signalForSelector:`，使用同样的block块。然而，我们有必要在视图控制器实现里提供一个空存根方法以避免编译器发出"实现不完全"之类的警告。

> 空存根方法：源于C++的一个非常不错的函数设计方法。在设计整个程序时，一般会先编写完所有的代码，然后开始编译和测试，但这样有时候会出现一大堆错误而不知从哪里入手，这时我们可以采用空存根技术。

> 存根是一个仅仅返回某个意义不大的值的空函数。存根可以用来测试整个程序的逻辑关系，以及分块实现程序的不同部分。

> 设计一个程序时，先分析设计程序的各个函数完成的功能；然后直接设计函数的存根并编译，编译通过，证明程序的逻辑关系没有问题的情况下，再来分别实现各个不同的函数(存根)。

接下来，我们有更多的机会来抽象这个类中的方法。`loadPopularPhotos`方法除了改变我们的状态之外，并没有什么卵用。如果`ReactiveCocoa`能够很好地监控这些状态，让我们不在这方面担心的话，那肯定是极好的！幸运的是，我恰好知道这个~

我们移除这个方法，在viewDidLoad中键入下面的代码来代码这个方法的调用：

```
RACSignal *photoSignal = [FRPPhotoImporter importPhotos];
RACSignal *photosLoaded = [photoSignal catch:^RACSignal *(NSError *error) {
    NSLog(@"Couldn't fetch photos from 500px : %@",error);
    return [RACSignal empty];
}];
RAC(self, photosArray) = photosLoaded;
[photosLoaded subscribeCompleted: ^{
    @strongify(self);
    [self.conllectionView reloadData];
}];

```

一开始我们只是进行了`importPhotos`方法调用，不同的是，我们用`signal`来存放其返回值。
然后，我们“捕抓”这个信号上的错误并将它打印出来(跟我们之前做的一样，只不过语法不同而已)。比起`subscribeError:`方法，`catch:`方法处理的更为巧妙：它允许无错误值的信号穿透它，仅在信号有错误事件发生时才会调用它的block并发送其在发生错误时的返回值。这里我们使用`catch:`方法，来过滤无错误的值。这个`catch:`块仅仅返回一个空信号。更多关于这方面知识的细节[请参考StackOverFlow的问题](http://stackoverflow.com/questions/19439636/difference-between-catch-and-subscribeerror)。

上面的方式，有一点点污染了我们的局部变量作用域，这可以用下面的更简洁的等效方法：

```Objective-C
RAC(self, photosArray) = [[[[FRPPhotoImporter importPhotos]
        doCompleted:^{
            @strongify(self);
            [self.collectionView reloadData];
        }] logError] catchTo:[RACSignal empty]];

```
使用RAC宏，我们创建了`photosLoaded`信号的最新值到`photoArray`属性的单向绑定。太好了，保持状态!

我们来看一下，我们的collectionViewCell的子类实现：

```Objective-C
@interface FRPCell ()

@property (nonatomic, weak) UIImageView *imageView;
@property (nonatomic, strong) RACDisposable *subscription;

@end

@implementation FRPCell

- (instancetype)initWithFrame:(CGRect)frame {
	...
}

- (void)perpareForReuse {
	[super perpareForReuse];
	
	[self.subscription dispose], self.subscription = nil;
}

- (void)setPhotoModel:(FRPPhotoModel *)photoModel {
	self.subscription = [[[RACObserve(photoModel, thumbnailData) filter:^BOOL(id value) {
		return value != nil;
	}] map:^id(id value) {
		return [UIImage imageWithData:value];
	}] setKeyPath:@keypath(self.imageView, image) onObject:self.imageView];
}

@end

```

这里有两个标志性的点表明了一个使用ReactiveCocoa来抽象的机会。

 1. 我们有状态(`subscription`属性)
 2. 我们手动处理`RACDisposable`的生命周期

无论何时调用一个`RACDisposable`对象的`dispose`方法，就是一个**"这里有更加响应式的方法来作某件事"**的好信号。在我们的例子中，这种嗅觉是对的。

通过在`FRPCell`创建一个新的属性，我们能够抽象掉使用`prepareForReuse`方法的必要性。这个属性就是`photoModel`(我们之前的行为就像是一个只写的属性，现在它将变为可读写的了)。把属性放在文件顶部：

```
@property (nonatomic, strong ) FRPPhotoModel *photoModel;
```

下一步我们将彻底摆脱`setPhotoModel:`方法。我们将为`photoModel的thumbnailData`观察我们自己的关键路径。将下面的代码添加到cell的初始化函数中。

```Objective-C

RAC(self.imageView, image) = [[RACObserve(self, photoModel.thumbnailData) ignore:nil] 
							map:^(NSData *data){
								return [UIImage imageWithData:data];
							}];

```

注意看我们观察的是`self`的`photoModel.thumbnailData`的关键路径，而非`self.photoModel`的`thumbnailData`的关键路径。这点微妙的区别，作用却大大不同。当`self`的属性`photoModel`或者`photoModel`的`thumbnailData`属性改变时，关键路径`photoModel.thumbnailData`将会收到一个被(这种变化所)引发的KVO消息。

现在我们总算彻底摆脱了`subscription`属性！

