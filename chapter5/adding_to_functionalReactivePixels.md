# 添加FunctionalReactivePixels
一个简单的画廊弄好了，但是我们是不是想看一下高清图呢？当用户点击画廊中的某一个单元格时，我们创建一个新的视图控制器并将其推入到导航堆栈中。

```
- (void)collectionView:(UICollectionView *)collectionView 
	didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
	FRPFullSizePhotoViewController * viewController = [[FRPFullSizePhotoViewController alloc] initWithPhotoModels:self.photosArray currentPhotoIndex:indexPath.item];
	
	viewController.delegate = self;
	[self.navigationController pushViewController:viewController animated:YES];
	
}
```

这个方法没有任何特殊的，只是些一般的OC方法。当然别忘了在当前实现文件里加载视图控制器(FRPFullSizePhotoViewControler)的头文件.现在让我们来创建这个视图控制器(FRPFullSizePhotoViewControler).

创建一个UIViewController的子类FRPFullSizePhotoViewControler,这不会是一个特别的‘Reactive’的视图控制器，实际上大部分只是`UIPageViewController`子视图控制器的模版。

```
@class FRPFullSizePhotoViewController;

@protocol FRPFullSizePhotoViewControllerDelegate <NSOject>
- (void)userDidScroll:(FRPFullSizePhotoViewController *)viewController toPhotoAtIndex:(NSInteger)index;

@end

@interface FRPFullSizePhotoViewController : UIViewController

- (instancetype)initWithPhotoModels:(NSArray *)photoModelArray currentPhotoIndex:(NSInteger)photoIndex;

@property (nonatomic , readonly) NSArray *photoModelArray;
@property (nonatomic, weak) id<FRPFullSizePhotoViewControllerDelegate> delegate;

@end

```

回到画廊视图控制器实现必要的代理方法：

```
- (void)userDidScroll:(FRPFullSizePhotoViewController *)viewController toPhotoAtIndex:(NSInteger)index{
	[self.collectionView scrollToItemAtIndexPath:[NSIndexPath indexPathForItem:index inSection:0] 
				atScrollPosition:UICollectionViewScrollPositionCenteredVertically 
						animated:NO];
}
```

当我们滑到一个新的图像去查看其高清图片时，这个方法将更新collectionView滑动的位置。这样一来，当用户查看完高清图回到这个界面的时候，高清图所对应的缩略图将会显示在界面上，方便用户获知自己浏览的位置以及继续往下浏览。

`#import`这些必要的数据模型的头文件并追加一下两个私有属性：

```
@interface FRPFullSizePhotoViewController () <UIPageViewControllerDataSource, UIPageViewControllerDelegate>
//Private assignment
@property (nonatomic, strong) NSArray *photoModeArray;
//Private properties
@property (nonatomic, strong) UIPageViewController *pageViewController;
@end
```

`photoModelArray`是共有的只读属性，但是内部可读写。第二个属性是我们的子视图控制器。我们这样来初始化：

```
- (instancetype)initWithPhotoModels:(NSArray *)photoModelArray currentPhotoIndex:(NSInteger)photoIndex{
	self = [self init];
	if (!self) return nil;
	
	//Initialized, read-only properties
	self.photoModelArray = photoModelArray;
	
	//Configure self
	self.title = [self.photoModelArray[photoIndex] photoName];
	
	//ViewControllers
	self.pageViewController = [UIPageViewController alloc] 
									initWithTransitionStyle:UIPageViewControlerTransitionStyleScroll 
									navigationOrientation:UIPageViewControllerNavigationOrientationHorizontal 
									options:@{ UIPageViewControllerInterPageSpacingKey: @(30)};
	self.pageViewController.dataSource = self;
	self.pageViewController.delegate = self;
	[self addchildViewController:self.pageViewController];
	
	[self.pageViewController setViewController:@[[self photoViewControllerForIndex:photoIndex]] 
							direction:UIPageViewControllerNavigationDirectionForward 
							animated:NO completion:nil ];
							
	return self;
}
```

赋值属性、设置标题、配置我们的`pageViewController`，一切都非常无聊，我们的viewDidLoad方法也同样简单。

```
- (void)viewDidLoad{
	[super viewDidLoad];
	
	self.view,backGroundColor = [UIColor blackColor];
	
	self.pageViewController.view.frame = self.view.bounds;
	
	[self.view addSubView:self.pageViewController.view];
}
```
我要指出的是，简便起见，在我的应用里我禁用了横向展示，因为这不是一本关于`autoresizingMask`或者`autoLayout`的书。你可以通过[Eria Sadun的书]()了解更多关于`autoLayout`方面的细节。
