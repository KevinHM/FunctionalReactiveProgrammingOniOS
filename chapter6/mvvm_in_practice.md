# MVVM的具体实践

&nbsp;&nbsp;本章的其他部分将把Functional Reactive Pixels Demo的其他代码迁移到MVVM架构中。我们将添加一个新的库到Podfile文件里。Github上创作了ReactiveCocoa的黑客，也同时创建了一个ViewModel的基类:ReactiveViewModel.我们将要使用它的0.1.1版本。更新Podfile之后立即运行`pod install`以安装该库。

&nbsp;&nbsp;重构的第一个类是高清图片视图控制器。从这儿开始是因为它的业务逻辑比较少，抽象成viewModel时相对简单。我们循序渐进，慢慢来。

&nbsp;&nbsp;目前，我们的`FRPFullSizePhotoViewController`包含一个图片数组和当前图片(在数组中)的下标值。我们将把他们抽象到我们的视图模型中来。

&nbsp;&nbsp;从头文件中移除自定义初始化，追加`FRPFullSizePhotoViewModel`的预申明。然后在这个新类中追加一个属性。

```
@property (nonatomic ,strong ) FRPFullSizePhotoViewModel *viewModel;
```

&nbsp;&nbsp;在实现文件里，#import这个新的视图模型(别担心，我们很快就会创建它)，

```
#import "FRPFullSizePhotoViewModel.h"
```

&nbsp;&nbsp;然后，移除`photoModelArray`私有属性的申明。重写我们的初始化方法以移除对`photoModelArray`实例的引用。代码看起来应该像下面这样:

```Objective-C
- (instancetype)init {
	self = [super init];
	if(!self) return nil;

	//ViewControllers
	self.pageViewController = [UIPageViewController alloc]
					initWithTransitionStyle:UIPageViewControllerTransitionStyleScroll
					navigationOrientation:UIPageViewControllerNavigationOrientationHorizontal
					   options:@{ UIPageViewControllerOptionInterPageSpacingKey : @30 };

	self.pageViewController.dataSource = self;
	self.pageViewController.delegate	  = self;
	[self addChildViewController:self.pageViewController];

	return self;
}
```

&nbsp;&nbsp;在你的`ViewDidLoad:`中添加如下代码：

```Objective-C
//Configure child view controllers
[self.pageViewController \
		setViewControllers: @[ [self photoViewControllerForIndex:self.viewModel.initialPhotoIndex] ]
		direction:UIPageViewControllerNavigationDirectionForward
		animated:NO
		completion:nil ];

//Configure self
self.title = [self.viewModel.initialPhotoModel photoName];

```

我们将要写的这个我们提到的方法，对于veiwModel中发生的事情，给你一种XX感。最后，进到`photoViewControllerForIndex`方法中，它应用了已经解除分配的`photoModelArray`，用下面的实现替代它。

```Objective-C
- (FRPPhotoViewController *)photoViewControllerForIndex:(NSInteger)index {
	if (index >= 0 && index < self.viewModel.photoArray.coung ) {
		FRPPhotoModel *photoModel = self.viewModel.model[index];

		FRPPhotoViewController *photoViewController = \
			[[FRPPhotoViewController alloc] initWithPhotoModel:photoModel index:index];

		return photoViewController;
	}

	// Index was out of bounds, return nil
	return nil;
}

```

&nbsp;&nbsp;好了！现在轮到我们的视图模型本身了。创建一个新的`RVMViewModel`的子类，并将其命名为`FRPFullSizedPhotoViewModel`.基于它将要封装的信息，以及我们在视图控制器中的需求，我们知道，我们的头文件看起来应该是下面这样：

```Objective-C
@class FRPPhotoModel;

@interface FRPFullSizePhotoViewModel : RVMViewModel

- (instancetype)initWithPhotoArray:(NSArray *)photoArray initialPhotoIndex:(NSInteger)initialPhotoIndex;
- (FRPPhotoModel *)photoModelAtIndex:(NSInteger)index;

@property (nonatomic , readonly, strong) NSArray *model;
@property (nonatomic, readonly) NSInteger initialPhotoIndex;
@property (nonatomic, readonly) NSString *initialPhotoName;

@end

```

&nbsp;&nbsp;`model`属性在`RVMViewModel`中被定义为`id`类型，我们把它重定义为`NSArray`. 我们也勾住了(即使用全局变量记录)我们最初照片的索引(下标)并且给我们最初的照片名属性定义了只读属性。这种微不足道的逻辑我们可以放到我们的视图控制器中，但很快我们就会看到更为复杂的情况。

&nbsp;&nbsp;我们来完成实现文件里的东西。第一件事就是：我们需要#import `FRPPhotoModel`类的头文件。然后，我们将打开私有属性的读写访问权限。

```Objective-C
//Model
#import "FRPPhotoModel.h"

@interface FRPFullSizePhotoViewModel ()
//private access
@property (nonatomic, assign) NSInteger initialPhotoIndex;

@end

```
&nbsp;&nbsp;好！下一步处理我们的初始化方法

```Objective-C
- (instancetype)initWithPhotoArray:(NSArray *)photoArray initialPhotoIndex:(NSInteger)initialPhotoIndex {
	self = [super initWithModel:photoArray];
	if(!self) return nil;

	self.initialPhotoIndex = initialPhotoIndex;

	return self;
}
```

初始化方法中，先调用超类的`initWithModel:`实现，然后设置自己的`initialPhotoIndex`属性。剩下的两个只读属性的获取逻辑微不足道。

```Objective-C
- (NSString *)initialPhotoName {
	return [[self photoModelAtIndex:self.initialPhotoIndex] photoName];
}

- (FRPPhotoModel *)photoModelAtIndex:(NSInteger)index {
	if(index < 0 || index > self.model.count - 1) {
		//Index was out of bounds, return nil
		return nil;
	}
	else {
		return self.model[ index ];
	}
}

```

&nbsp;&nbsp;这样做的另一个优点是：业务逻辑不需要重复书写，而且也使得业务逻辑非常好进行单元测试。

&nbsp;&nbsp;最后，我们需要在高清视图控制器中设置该视图模型，否则屏幕上将不会显示任何东西。导航到我们的画廊视图控制器（那个我们实例化并推出高清视图控制器的地方）。用下面的代码来替换这个业务逻辑：

```Objective-C
[[self rac_signalForSelector:@selector(collectionView:didSelectItemAtIndexPath:)
	fromProtocol:@protocol(UIcollectionViewDelegate)] subscribeNext:^(RACTuple *arguments) {
		@strongify(self);

		NSIndexPath *indexPath = arguments.second;
		FRPFullSizePhotoViewModel *viewModel = [[FRPPhotoViewModel alloc]
			initWithPhotoArray:self.viewModel.model initialPhotoIndex:indexPath.item];

		FRPFullSizePhotoViewController *viewController = [[FRPFullSizePhotoViewController alloc] init];

		viewController.viewModel = viewModel;
		viewController.delegate = (id<FRPFullSizePhotoViewControllerDelegate>)self;

		[self.navigationController pushViewController:viewController animated:YES];
	}];
```
在下一节开始之前，我们没有计划为视图模型撰写单元测试。下一节我们看到在视图模型上如何运行测试驱动开发的概念。现在我们来完成`FRPGalleryViewModel`吧，很基础。我们想要从视图控制器中抽象出来的逻辑是通过API加载`model`的数据内容。我们来看一下应该怎么做：

```Objective-C

@interface FRPGalleryViewModel : RVMViewModel

@property (nonatomic, readonly, strong) NSArray *model;

@end

```

&nbsp;&nbsp;基本的接口：将`model`申明为数组`NSArray`.接下来，我们简单实现它：

```Objective-C

//Utilities

#import "FRPPhotoImporter.h"

@interface FRPGalleryViewModel ()

@end

@implementation FRPGalleryViewModel

- (instancetype)init {
	self = [super init];
	if(!self) return nil;

	RAC(self, model) = [[[FRPPhotoImporter importPhotos] logError] catchTo:[RACSignal empty]];

	return self;
}

@end

```

&nbsp;&nbsp;有争议的是，我们应该把从API加载数据的(RAC绑定的)逻辑放在初始化方法中，还是放在视图模型被激活的地方。接下来我们会讨论更多的关于激活的内容，但我想要展示给你们看这个视图模型到底能做到多简单。将直接在画廊视图控制器中加载数据内容的逻辑迁移到画廊的视图模型中是非常简单的：在视图控制器的初始化中初始化视图模型===》任何引用试图控制`self.model`属性的地方使用`self.viewModel.model`来代替即可。

&nbsp;&nbsp;我们可以进一步深挖视图模型的构造，甚至可以通过一系列的访问器把`model`的访问逻辑抽象出来，但在这个例子里就有点过多‘抽象’了。更重要的是你可以根据你的喜好将更多的或者更少的业务逻辑抽象到视图模型中。我发现，就我个人而言，这个架构使用的越多，业务逻辑抽象出来的越多，就意味着更轻量级的视图控制器以及高内聚和可测试的代码。

&nbsp;&nbsp;把注意力移到单元测试之前，我们来做多一次用视图模型来抽象业务逻辑的实践。

&nbsp;&nbsp;我们的最后一个例子是`FRPPhotoViewController`上的`FRPPhotoViewModel`:创建一个`RVMViewModel`的视图模型子类并放置在视图控制器中(很快我们会回到视图模型中)。

&nbsp;&nbsp;视图控制器的新的初始化方法如下：

```Objective-C
- (instancetype)initWithViewModel:(FRPPhotoViewModel *)viewModel index:(NSInteger)photoIndex {

	self = [self init];//NS_DESIGNATED_INITIALIZER
	if(!self) return nil;

	self.viewModel = viewModel;
	self.photoIndex = photoIndex;

	return self;
}

```

&nbsp;&nbsp;确定导入必要的头文件并为视图模型申明私有属性。现在我们需要使用新的初始化方法初始化视图控制器。看一看视图控制器到页面视图控制器的方法`photoViewControllerForIndex:`.

```Objective-C
- (FRPPhotoViewController *)photoViewControllerForIndex:(NSInteger)index {
	FRPPhotoModel *photoModel = [self.viewModel photoModelAtIndex:index];
	if(photoModel) {
		FRPPhotoViewModel *photoViewModel = [[FRPPhotoViewModel alloc] initWithModel:photoModel];
		FRPPhotoViewController *photoViewController = [[FRPPhotoViewController alloc] \
							 initWithViewModel:photoViewModel
									      index:index];

		return photoViewController;
	}

	return nil;
}

```

&nbsp;&nbsp;新的初始化过程中我们创建了一个视图模型。

&nbsp;&nbsp;在我们的`viewDidLoad:`方法里，我们将使用这个新的视图模型为我们的图片视图提供数据，并且为用户显示图片的下载进度。这里有个貌似冲突的地方：图片的下载是视图的模型的业务逻辑之一，但视图什么时候显示开始加载数据(这个业务逻辑)视图模型中没有体现---记住一个好的视图模型不应该引用视图本身。那么我们如何来混合地使用这两个业务逻辑？

&nbsp;&nbsp;答案是我们借助视图模型的`active`状态来对付（上面的情况）。`RVMViewModel`提供了一个布尔属性`active`，当试图控制器变得"活跃"时(不管在语义的上下文里这是啥意思)，在这里，我们可以在`viewWillAppear:`和`viewDidDisappear:`这些方法来设置这个属性。

```Objective-C
- (void)viewWillAppear:(BOOL)animated {
	[super viewWillAppear:animated];

	self.viewModel.active = YES;
}

- (void)viewDidDisappear:(BOOL)animated {
	[super viewDidDisappear:animated];

	self.viewModel.active = NO;
}
```
相当简单吧，我们来看一下我们新的`viewDidLoad`方法：

```Objective-C
- (void)viewDidLoad {
	[super viewDidLoad];

	//Configure self's view
	self.view.backgroundColor = [UIColor blackColor];

	//Configure subViews
	UIImageView *imageView = [[UIImageView alloc] initWithFrame:self.view.bounds];
	RAC(imageView, image) = RACObserve(self.viewModel,photoImage);
	imageView.contentModel = UIViewContentModelScaleAspectFit;
	[self.view addSubView:imageView];
	self.imageView = imageView;

	[RACObserve(self.viewModel, loading) subscribeNext:^(NSNumber *loading) {
		if(loading.boolValue) {
			[SVProgressHUD show];
		}
		else {
			[SVProgressHUD dismiss];
		}
	}];
}

```

&nbsp;&nbsp;该图片视图的图片属性的绑定是标准的ReactiveCocoa方式,有趣的是下面(我们要提到的)我们使用`loading`的时刻。当加载信号发送`YES`的时候我们展示进度HUD，发送`NO`的时候，让进度HUD消失。我们将看到该`loading`信号本身如何依赖于`didBecomeActiveSignal`。现在只是视图模型通过网络请求获取图像数据的序幕。

&nbsp;&nbsp;接口的申明如下：

```Objective-C
@class FRPPhotoModel;

@interface FRPPhotoViewModel : RVMViewModel

@property (nonatomic, readonly) FRPPhotoModel *model;
@property (nonatomic, readonly) UIImage *photoImage;
@property (nonatomic, readonly, getter = isLoading) BOOL loading;

- (NSString *)photoName;

@end

```

&nbsp;&nbsp;该`model`和`photoImage`属性的用法已经解释过了。`photoName`事实上作为属性在代码库的其他地方被用来设置一些东西，类似于分页视图控制器的标题这样。你可以下载Github的代码库了解详情。我们来看一下实现：

```Objective-C
#import "FRPPhotoViewModel.h"

//Utilities
#import "FRPPhotoImporter.h"
#import "FRPPhotoModel.h"

@interface FRPPhotoViewModel ()

@property (nonatomic, strong) UIImage *photoImage;
@property (nonatomic, assign, getter = isLoading) BOOL loading;

@end

@implementation FRPPhotoViewModel

- (instancetype)initWithModel:(FRPPhotoModel *)photoModel {
	self = [super initWithModel:photoModel];
	if(!self) return nil;

	@weakify(self);
	[self.didBeComeActiveSignal subscribeNext:^(id x) {
		@strongify(self);
		self.loading = YES;
		[[FRPPhotoImporter fetchPhotoDetails:self.model] subscribeError:^(NSError *error) {
			NSLog(@"Could not fetch photo details: %@",error);
		} completed:^{
			self.loading = NO;
			NSLog(@"Fetched photoDetails.");
		}];
	}];

	RAC(self, photoImage) = [RACObserve(self.model, fullsizedData) map:^id (id value) {
		return [UIImage imageWithData:value];
	}];

	return self;
}

- (NSString *)photoName {
	return self.model.photoName;
}

@end


```

&nbsp;&nbsp;该`didBecomeActive`信号订阅带有"函数副作用"的加载照片详情包括它的高清图片的数据。然后`photoImage`属性与模型的映射结果绑定。

&nbsp;&nbsp;使用`didBecomeActiveSignal`这种方法来启动一些像网络操作这样昂贵的任务，远远优于我们早前在初始化方法中启动他们的方法。

&nbsp;&nbsp;这就是在本书中我们将要涉及的全部内容，更多详情请参考[functional reactive pixels](https://github.com/ashfurrow/FunctionalReactivePixels)，这个代码库包含了更多的在图片详情视图控制器和登陆视图控制器中使用视图模型的例子。这些Demo将向你展示如何有效地使用`ReactiveCocoa`执行网络操作和使用`RACCommands`响应用户界面交互。





