# MVVM的具体实践

本章的其他部分将把Functional Reactive Pixels Demo的其他代码迁移到MVVM架构中。我们将添加一个新的库到Podfile文件里。Github上创作了ReactiveCocoa的黑客，也同时创建了一个ViewModel的基类:ReactiveViewModel.我们将要使用它的0.1.1版本。更新Podfile之后立即运行`pod install`以安装该库。

重构的第一个类是高清图片视图控制器。从这儿开始是因为它的业务逻辑比较少，抽象成viewModel时相对简单。我们循序渐进，慢慢来。

目前，我们的`FRPFullSizePhotoViewController`包含一个图片数组和当前图片(在数组中)的下标值。我们将把他们抽象到我们的视图模型中来。

从头文件中移除自定义初始化，追加`FRPFullSizePhotoViewModel`的预申明。然后在这个新类中追加一个属性。

```
@property (nonatomic ,strong ) FRPFullSizePhotoViewModel *viewModel;
```

在实现文件里，#import这个新的视图模型(别担心，我们很快就会创建它)，

```
#import "FRPFullSizePhotoViewModel.h"
```

然后，移除`photoModelArray`私有属性的申明。重写我们的初始化方法以移除对`photoModelArray`实例的引用。代码看起来应该像下面这样:

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

在你的`ViewDidLoad:`中添加如下代码：

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
好了！现在轮到我们的视图模型本身了。创建一个新的`RVMViewModel`的子类，并将其命名为`FRPFullSizedPhotoViewModel`.基于它将要封装的信息，以及我们在视图控制器中的需求，我们知道，我们的头文件看起来应该是下面这样：

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

`model`属性在`RVMViewModel`中被定义为`id`类型，我们把它重定义为`NSArray`. 我们也勾住了(即使用全局变量记录)我们最初照片的索引(下标)并且给我们最初的照片名属性定义了只读属性。这种微不足道的逻辑我们可以放到我们的视图控制器中，但很快我们就会看到更为复杂的情况。

我们来完成实现文件里的东西。第一件事就是：我们需要#import `FRPPhotoModel`类的头文件。然后，我们将打开私有属性的读写访问权限。

```Objective-C
//Model
#import "FRPPhotoModel.h"

@interface FRPFullSizePhotoViewModel ()
//private access
@property (nonatomic, assign) NSInteger initialPhotoIndex;

@end

```
好！下一步处理我们的初始化方法

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

这样做的另一个优点是：业务逻辑不需要重复书写，而且也使得这个非常好进行单元测试。



