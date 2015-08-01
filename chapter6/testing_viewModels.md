# 测试ViewModels

本书的最后一节，我们谈谈测试，尤其是单元测试。在iOS的开发社区里，这是一个有争议的话题，这也是为什么我要把它放在最后的原因。理想的情况下。你应该在编写视图模型的同时为它编写单元测试。然而学习如何使用这种新的模式来编码已经很困难，尝试去测试这些你没有吃透的东西，多你来说压力太大，所以我把它放在了最后（学到这里我相信你已经理解了这种编码方式）。

当然我也注意到，并不是每个人都以相同的方式来测试，或者能够测试到相同的程度。我有.Net编程背景，在.net中使用mocks来测试系统的实现细节是最平常不过的了。其他平台背景的开发者较少使用mocks来做，甚至从来没有这样的经验。本节我只将我的单元测试方法分享给大家，如果你觉得合适就采用。

确保你的`Podfile`文件包含下面这些库：

```Objective-C
target "FRPTests" do

pod 'ReactiveCocoa', '2.1.4'
pod 'ReactiveViewModel', '0.1.1'
pod 'libextobjc', '0.3'
pod '500px-iOS-api', '1.0.5'
pod 'Specta', '~> 0.2.1'
pod 'Expecta', '~> 0.2'
pod 'OCMock', '~> 2.2.2'

end

```

然后运行`pod install`.

首先我们来看看`FRPFullSizePhotoViewModel`，因为它最具Objective-C风范(没有太多ReactiveCocoa).

```Objective-C
@interface FRPFullSizePhotoViewModel ()
//Private access
@property (nonatomic, assign) NSInteger initialPhotoIndex;

@end

@implementation FRPFullSizePhotoViewModel

- (instancetype)initWithPhotoArray:(NSArray *)photoArray initialPhotoIndex:(NSInteger)initialPhotoIndex {
	self = [self initWithModel:photoArray];
	if(!self) return nil;
	
	self.initialPhotoIndex = initialPhotoIndex;
	
	return self;
}

- (NSString *)initialPhotoName {
	return [self.model[self.initialPhotoIndex] photoName];
}

- (FRPPhotoModel *)photoModelAtIndex:(NSInteger)index {
	if(index < 0 || index > self.model.count - 1) {
		//Index was out of bounds, return nil
		return nil;
	}
	else {
		return self.model[index];	
	}
}

@end

```
好了，我们先来测试这个初始化方法，然后在转移到其他两个方法上。

我们想印证初始化我们的视图模型时，它的两个属性`model`和`initialPhotoIndex`被正确地赋值了。

```Objective-C
#import <Specta/Specta.h>
#define EXP_SHORTHAND
#import <Expecta/Expecta.h>
#import <OCMock/OCMock.h>
#import "FRPPhotoModel.h"

#import "FRPFullSizePhotoViewModel.h"

SpecBegin(FRPFullSizePhotoViewModel)

describe(@"FRPFullSizePhotoModel", ^{
	it (@"Should assign correct attributes when initialized", ^{
		NSArray *model = @[];
		NSInteger initialPhotoIndex = 1337;
		
		FRPFullSizePhotoViewModel *viewModel =\
		 [[FRPFullSizePhotoViewModel alloc] initWithPhotoArray:model 
		 										    initialPhotoIndex: initialPhotoIndex];
		
		expect(model).to.equal(viewModel.model);
		expect(initialPhotoIndex).to.equal(viewModel.initialPhotoIndex);
		
	});
});

SpecEnd

```
在该代码段顶部，我们导入了一些头文件，包括一个奇怪的预定义`EXP_SHORTHAND`,我们把他放在那里以便于可以使用类似`expect()`这样的shorthand matchers（速记匹配）的语法。然后我们引入我们的私有接口`SpecBegin(...)/SpecEnd`来为我们正在测试的视图模型屏蔽编译警告，最后的部分就是我们的单元测试本身。`Specta`的测试规范相当简单，你可以阅读更多的关于这方面的信息，但本书不会深入讲解它的一些细节。总之你的测试始于`SpecBegin`并终止于`SpecEnd`，测试例程用类似于`@"应该。。。",^{ 预测正常的情况应该如何 }`写在中间。

好了，停止模拟器中正在运行的应用，按下`cmd+U`快捷键，你就可以运行这段单元测试了。如果一切正常，你就能通过测试。

接下来我们来看看`photoModelAtIndex:`方法

```Objective-C
- (FRPPhotoModel *)photoModelAtIndex:(NSInteger)index {
	if(index < 0 || index > self.model.count - 1 ) {
		// Index was out of bounds ,return nil
		return nil;
	}
	else {
		return self.model[ index ];
	}
}
```
这里面没有太多的业务逻辑，但是我们看到其他地方都要使用它，所以我们的测试应该是健壮的。

```Objective-C
it(@"Should return nil for an out-of-bounds photo index", ^{
	NSArray *model = @[[NSobject new]];
	NSInteger initialPhotoIndex = 0;
	
	FRPFullSizePhotoViewModel *viewModel = \
		[[FRPFullSizePhotoViewModel alloc] initWithPhotoArray:model initialPhotoIndex:initialPhotoIndex];
	
	id subzeroModel = [viewModel photoModelAtIndex:-1];
	expect(subzeroModel).to.beNil();
	
	id aboveBoundsModel = [viewModel photoModelAtIndex:model.count];
	expect(aboveBoundsModel).to.beNil();
});

it(@"Should return the correct model for photoModelAtIndex:",^{
	id photoModel = [NSObject new];
	NSArray *model = @[photoModel];
	NSInteger initialPhotoIndex = 0;
	
	FRPFullSizePhotoViewModel *viewModel = \
		[[FRPFullSizePhotoViewModel alloc] initWithPhotoArray:model initialPhotoIndex:initialPhotoIndex];
	
	id returnModel = [viewModel photoModelAtIndex:0];
	expect(returnModel).to.equal(photoModel);
	
});

```
太棒了！我们这个新的测试保证了我们的代码具有完全的代码覆盖率。它检测了`photoModelAtIndex:`参数的三种可能的情况：少于0、在作用范围内以及越界。

最后，我们来看下`initialPhotoName`方法：

```Objective-C
- (NSString *)initialPhotoName {
	return [self.model[self.initialPhotoIndex] photoName];
}

```
方法看起来很简单，但实际上这里面包含了更深层级的东西。恰当地重构一些代码并为它写一点不一样的更小的测试代码，来严格地测试这个方法。

```Objective-C
- (NSString *)initialPhotoName {
	FRPPhotoModel *photoModel = [self initialPhotoModel];
	return [photoModel photoName];
}

- (FRPPhotoModel *)initialPhotoModel {
	return [self photoModelAtIndex:self.initialPhotoIndex];
}

```

这更清晰简单了，一个方法确切地只做一件事情，就像一棵树的树皮，层层叠叠相互依存。只要我们一路下来所有的代码都测试，那么最后我们就可以很确切地保证代码的健壮性。

`initialPhotoModel`是一个私有方法，所以测试它我们需要在测试文件中申明它。

```Objective-C
@interface FRPFullSizePhotoViewModel ()

- (FRPPhotoModel *)initialPhotoModel;

@end
```

你看到的所有我们的测试代码都非常简单。

```Objective-C
it (@"Should return the correct initial photo model", ^{
	NSArray *model = @[[NSobject new]];
	NSInteger initialPhotoIndex = 0;
	
	FRPFullSizePhotoViewModel *viewModel = \
		[[FRPFullSizePhotoViewModel alloc] initWithPhotoArray:model initialPhotoIndex:initialPhotoIndex];
	
	id mockViewModel = [OCMockObject partialMockForObject:viewModel];
	[[[mockViewModel expect] andReturn:model[0]] photoModelAtIndex:initialPhotoIndex];
	
	id returnedObject = [mockViewModel initialPhotoModel];
	
	expect(returnedObject).to.equal(model[0]);
	
	[mockViewModel verify];
});
```

这个测试是用来确认当`initialPhotoModel`被调用时，接下来它应该调用`photoModelAtIndex:`方法并将`initialPhotoIndex`作为参数传入。这个测试是否简单取决于我们测试`photoModelAtIndex:`是否充分。

接下来，就让我们一起来看看`FRPGalleryViewModel`,这看似非常简单：

```Objective-C
- (instancetype)init {
	self = [super init];
	if(!self) return nil;
	
	RAC(self, model) = [[[FRPPhotoImporter importPhotos] logError] catchTo:[RACSignal empty]];
	
	return self;
}

```

然而，它可测性不高，需要重构。

我们简单地重构下视图模型。新的实现如下：

```Objective-C
@implementation FRPGalleryViewModel

- (instancetype)init {
	self = [super init];
	if(!self) return nil;
	
	RAC(self, model) = [self importPhotosSignal];
	
	return self;
}

- (RACSignal *)importPhotosSignal {
	return [[[FRPPhotoImporter importPhotos] logError] catchTo:[RACSignal empty]];
}

@end

```

我们把`importPhotos`的调用抽出来，以方便测试这个方法是否被调用。我们不会测试`FRPPhotoImporter`，关于它的测试(即单例测试)已经超出了本书的范畴。

这部分的测试代码如下：

```Objective-C
#import "Specta.h"
#import <OCMock/OCMock.h>

#import "FRPGalleryViewModel.h"

@interface FRPGalleryViewModel ()

- (RACSignal *)importPhotosSignal;

@end

SpecBegin(FRPGalleryViewModel)

describe(@"FRPGalleryViewModel",^{
	it(@"should be initialized and call importPhotos", ^{
		id mockObject = [OCMockObject mockForClass:[FRPGalleryViewModel class]];
		[[[mockObject expect] andReturn:[RACSignal empty]] importPhotosSignal];
		
		mockObject = [mockObject init];
		
		[mockObject verify];
		[mockObject stopMocking];
	});
});

```

为了测试一个方法，测试代码也太多了吧！ 我知道，我知道~ 这是OCMock没落的原因之一，它竟然需要这么多的模板。但你不能责怪它，因为它要工作在令它不寒而栗的Objective-C平台上！

我们创建了一个`FRPGalleryViewModel`的mock版本，告诉它期望`importPhotoSignal`被调用。然后才进行对象的初始化。这里使用了一点点技巧，因为我们在mockObject上调用了init方法，但它(init)实际上是一个NSProxy的子类。然后，对OCMock来讲，它足够聪明，它了解这一切，有能力做出正确的选择。只是看起来有点诡异罢了。我们使用`[mockObject init]`给`mockObject`赋值，也是为了屏蔽编译警告。最后我们验证了所有预期可能被调用的方法。

这个例子中表现出来的测试很困难的情况也说明了另一个问题，你应该避免视图模型的初始化方法产生"副作用"(参见前面章节提到的“函数的副作用”)，应该使用`didBecomeActiveSignal`来代理。

下面我们来测试`FRPPhotoViewModel`.再次突出引起函数副作用和使用`didBecomeActiveSignal`的区别。

快速浏览下实现：

```Objective-C

@implementation FRPPhotoViewModel

- (intancetype)initWithModel:(FRPPhotoModel *)photoModel {
	self = [super initWithModel:photoModel];
	if(!self) return nil;
	
	@weakify(self);
	[self.didBecomeActiveSignal subscribeNext:^ (id x) {
		@strongify(self);
		self.loading = YES;
		[[FRPPhotoImporter fetchPhotoDetails:self.model] 
			subscribeError: ^ (NSError *error) {
				NSLog(@"Could not fetch photo details: %@",error);
			} 
			completed: ^ {
				self.loading = NO;
				NSLog(@"Fetched photo details");
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
首先我们来测试`photoName`方法：

```Objective-C
#import <Specta/Specta.h>
#define EXP_SHORTHAND
#import <Expecta/Expecta.h>
#import <OCMock/OCMock.h>

#import "FRPPhotoViewModel.h"
#import "FRPPhotoModel.h"

SpecBegin(FRPPhotoViewModel)

describe (@"FRPPhotoViewModel", ^{
	it(@"should return the photo's name property when photoName is invoked", ^{
		NSString *name = @"Ash";
		
		id mockPhotoModel = [OCMockObject mockForClass:[FRPPhotoModel class]];
		[[[mockPhotoModel stub] andReturn:name] photoName];
		
		FRPPhotoViewModel *viewModel = [[FRPPhotoViewModel alloc] initWithModel:nil];
		id mockViewModel = [OCMockObject partialMockForObject:viewModel];
		[[[mockViewModel stub] andReturn:mockPhotoModel] model];
		
		id returnName = [mockViewModel photoName];
		
		expect(returnedName).to.equal(name);
		[mockPhotoModel stopMocking];
	});
});

```
我们为mock的视图模型的model属性添加了一个mockPhotoModel，它会mocks所有的途径。

现在来看这个复杂的初始化方法，这东西看起来真巨大！近20行纯粹的未经测试的代码。哎呀！让我们来一点点简化这个事情，并逐步加上我们的测试代码。

```Objective-C

```



