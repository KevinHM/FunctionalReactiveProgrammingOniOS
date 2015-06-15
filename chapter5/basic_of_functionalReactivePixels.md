# FunctionalReactivePixels的基础知识
FunctionReactivePixels将会是一个简单的观看'500px'中最受欢迎的照片的应用。一旦我们完成这一节，应用的主界面将会像下面这样：

> 插入一个图片

当然我们也可以像下图一样观看全屏模式下的图片。

> 插入一个图片

这个App将使用Collection Views。如果你没有太多这方面的经验，也不需要太过担心---他们(CollectionView)就像TableView一样，使用起来非常简单。如果你对UICollectionView感兴趣，可以阅读我的[另一本书](http://www.amazon.com/iOS-UICollectionView-Complete-Edition-Programming-ebook/dp/B00IHZKDCU).

我们将使用CocoaPods来管理我们的依赖，现在创建一个新的工程。我喜欢使用空模版以便我可以完全控制viewController层级。

首先、我们将创建一个UICollectionViewController的子类FRPGalleryViewController.同时我们创建一个UICollectionViewFlowLayout的子类FRPGalleryFlowLayout.

```
#import the new flow layout's header in the view controller's implementation file and
#then override FRPGalleryViewController's init method

- (id)init{
    FRPGalleryFlowLayout *flowLayout = [[FRPGalleryFlowLayout alloc] init];
    self = [self initWithCollectionViewLayout:flowLayout];
    if(!self) return nil;
    return self;
}

```

这将初始化collection View的layout为我们自己的layout.这个flowlayout子类的实现非常简单，只需要设置一些属性就可以了。

```
@implementation FRPGalleryFlowLayout
- (instancetype)init{
    if (!(self = [super init])) return nil;

    self.itemSize = CGSizeMake(145,145);
    self.minimumInteritemSpacing = 10;
    self.minimumLineSpacing = 10;
    self.sectionInset = UIEdgeInsetsMake(10,10,10,10);

    return self;
}
@end
```

很棒!下一步，我们需要把Viewcontroller展现在屏幕上。为了实现这个，我们首先要在应用的application delegate的`application: didFinishLaunchingWithOptions:`方法。我们想要将collectionview Controller置于一个navigationController容器中：

```
- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.rootViewController = [[UINavigationController alloc] initWithRootViewController:[[FRPGalleryViewController alloc] init]];

    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    return YES;
}
```
很好!如果我们现在运行，我们将看到一个空视图。

我们来填充一些内容。创建一个Podfile文件,并填写如下内容:
```
platform :ios, "7.0"
target "FRP" do
    pod 'ReactiveCocoa', '~> 2.1.4'
    pod 'libextobjc', '~> 0.3'
    pod '500-iOS-api', '~> 1.0.4'
    pod 'SVProgressHUD', '~> 0.9'
end

target "FRPTests" do

end
```
下一章，我们将添加一些测试。现在运行`pod install`,然后打开Xcode通用的`workspace`文件。打开与编译头文件`FRP-Prefix.pch`(Xcode6之后，新建工程默认不加载pch文件，需要自己添加，Apple的最佳实践中已经不推荐使用全局的预编译pch文件)，然后添加下面的内容。这些语义会自动加载到项目的所有文件中。

```
//Pods
#import <ReactiveCocoa/ReactiveCocoa.h>
#import <500px-iOS-api/PXAPI.h>
#import <libextobjc/EXTScope.h>

//App Delegate
#import "FRPAppDelegate.h"
#define AppDelegate ((FRPAppDelegate *)[[UIApplication sharedApplication] delegate])

```

对于这样使用AppDelegate单例的用法，Saul Mora说：“每次看到你这么做，我家的狗都想死”。
但是这不是一本关于设计模式的书---这是一本关于ReactiveCocoa的书，所以我们可能要害死一些狗狗。。。

创建一个AppDelegate的属性来hold住500px API客户端


```
@property (nonatomic, readonly) PXAPIHelper * apiHelper;

```

在`application:didFinishLaunchingWithOptions:`方法中实例化这个变量。

```
self.apiHelper = [[PXAPIHelper alloc]
                    initWithHost:nil
                    consumerKey:@"DC2To2BS0ic1ChKDK15d44M42YHf9gbUJgdFoF0m"
                    consumerSecret:@"i8WL4chWoZ4kw9fh3jzHK7XzTer1y5tUNvsTFNnB"];
```

我提供了一对一次性消费的密钥---请不要疯到你也使用这对密钥，你可以[申请](https://500px.com/)自己的。

好了，我们差不多也该建立数据的加载了。我们需要一个数据模型来hold住我们的信息。我创建了下面的`FRPPhotoModel`。

```
@interface FRPPhotoModel : NSObject
@property (nonatomic, strong) NSString *photoName;
@property (nonatomic, Strong) NSNumber *identifier;
@property (nonatomic, strong) NSString *photographerName;
@property (nonatomic, strong) NSNumber *rating;
@property (nonatomic, strong) NSString *thumbnailURL;
@property (nonatomic, strong) NSData *thumbnailData;
@property (nonatomic, strong) NSString *fullsizedURL;
@property (nonatomic, strong) NSData * fullsizedData;


@end

@implementation FRPPhotoModel

@end
```

非常好，到这里，我们将不直接在ViewController中加载内容，相反，这部分逻辑将被抽象到另一个类中。创建一个名为`FRPPhotoImporter`的类。

到现在为止没有一处代码是关于函数式的。别担心，我们就要这么做了！这个`FRPPhotoImporter`将不会真正返回一个`FRPPhotoModel`对象，相反他会返回一些随身携带API最新的请求结果的信号。

```
@interface FRPPhotoImporter : NSObject
+ (RACSignal *)importPhotos;

@end
```
`FRPPhotoImporter`的`importPhotos`方法返回一个从API发送最新结果的RACSignal。这个RACSignal实际上是一个RACReplaySubject.但是由于ReactiveCocoa编程指南中不建议使用RACSubjects，我们申明的公共接口的返回类型为RACSignal而非RACSubject.现在让我们继续往下看:

```
+ (RACSignal *)importPhotos{
    RACReplaySubject * subject = [RACReplaySubject subject];
    NSURLRequest * request = [self popularURLRequest];
    [NSURLConnection sendAsynchronousRequest:request 
    								queue:[NSOperationQueue mainQueue] 
    					completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError){
    						if (data) {
    							id results = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
    							
    							[subject sendNext:[[[results[@"photos"] rac_sequence] map:^id(NSDictionary *photoDictionary){
    								FRPPhotoModel * model = [FRPPhotoModel new];
    							
    								[self configurePhotoModel:model withDictionary:photoDictionary];
    								[self downloadThumbnailForPhotoModel:model];
    							
    								return model;
    							}] array];
    							
    							[subject sendCompleted];
    						}
    						else{
    							[subject sendError:connectionError];
    						}
    }];
    
    return subject;
    
}
```

这里面包含的内容太多，我们慢慢来整理一下：

- 首先我们创建了一个新的`RACReplaySubject`实例(这将是我们要返回的对象)。
- 其次我们创建了一个`NSURLRequest`来获取500px上热门的`FRPPhotoModel`数据。
- 随后我们发送一个网络的异步请求，并立即返回RACSubject对象。

这个直接返回的结果值得我们关注。

这个RACSubject对象被异步网络请求的回调block捕获，当API接口返回数据时回调block就会被调用，然后RACSubject对象会将结果传送出来，这些值将被我们的订阅了RACSubject信号的接收者所接受。

这是你看到的异步操作中，一个非常普通的模式。

1. 创建一个RACSubject. 
2. 从异步调用的完成block中向RACSubject传送结果值。
3. 立即返回这个RACSubject对象

重要的是，要注意一个普通的RASSubject及其子类RACReplaySubject之间的区别。RACReplaySubject可以确保他背后的Subject只会被订阅一次，避免执行重复的操作(就像上面这种网络活动的情况)，RACReplaySubject将会缓存这个订阅的值，并将其转发给新的订阅者们--- 对我们的需求来说这非常完美。就像ReactiveCocoa的开发者Justin Spahr-Summers所指出的，这也能够避免可能的竞争状况。

我们发送了一个完整的数据集而不是单个随时间变化的流。如果我们连环地发送一个个单独的`FRPPhotoModel`流，这将'更加Reactive',也有助于实现分页的需求，但是我们不打算采用这种方式，因为他有点点‘高级’了。你可以下载[octokit](https://github.com/octokit/octokit.objc)：一个类似这种方式的例子。

URL请求的构造方法看起来应该是这样的:

```
+ (NSURLRequest *)popularURLRequest {
	return [AppDelegate.apiHelper urlRequestForPhotoFeature:PXAPIHelperPhotoFeaturePopular 
				resultsPerPage:100 page:0 
				photoSize:PXPhotoModelSizeThumbnail 
				sortOrder:PXAPIHelperSortOrderRating 
				except:PXPhotoModelCategoryNude];
}
```
subject发送什么，完全看不到好吗？呃。这取决于回调block.

```
if(data){
	id results = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
	[subject sendNext:[[[results[@"photos"] rac_sequence] map:^id (NSDictionary *photoDictionary){
		FRPPhotoModel *model = [FRPPhotoModel new];
		[self donwloadThumbnailForPhotoModel:model];
		
		return model;
	}] array]];
	
	[subject sendCompleted];
}
else{
	[subject sendError:connectionError];
}
```
测试是否有数据返回时，可以说这不是一个很好的错误条件检测的方法，但这是一个教学的例子。如果数据为`nil`,我们会发送一个`errorValue`,否则我们会反序列化`JSON`数据并处理它。这不太容易很快就看清楚是怎么做到的，让我们来仔细看看。

```
[subject sendNext:[[[results[@"photos"] rac_sequence] map:^id (NSDictionary *photoDictionary){
	FRPPhotoModel * model = [FRPPhotoModel new];
	[self configurePhotoModel:model withDictionary:photoDictionary];
	[self downloadThumbnailForPhotoModel:model];
	return model;
}] array]];

[subject sendCompleted];

```
发送一个值，随着subject撸过去，第一个表达式结构相当简洁(但是场景很典型)。这个值是`photos`的值，然后转化为一个序列(sequence),然后做映射，最后转化为一个数组。这是上一章介绍的非常简单的`map`技术。

这个`map`(映射)非常有意思。序列中的每一个元素，都会创建一个新的`FRPPhotoModel`对象、设置它然后返回它。为每一个`results[ @"photos" ]`的数组元素创建了一个`FRPPhotoModel`数组。这个数组就是随着subject发送过来的值。最后我们发送一个完成值`completedValue`好让订阅者们知道任务完成了。

|原始值| 映射 | 映射值 |
| :---: | :---: | :---:|
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |
| {values} | map | FRPPhotoModel |


注意在信号上手动附送值的能力是非典型的，这是RACSubject实例的专属能力。

`configurePhotoModel:withDictionary:`方法，看起来应该像下面这样：

```
+ (void)configurePhotoModel:(FRPPhotoModel *)photomodel withDictionary:(NSDictionary *)dictionary{
	//Basic details fetched with the first, basic request
	photomodel.photoname = dictionary[@"name"];
	photomodel.identifier = dictionary[@"id"];
	photomodel.photographerName = dictionary[@"user"][@"username"];
	photomodel.rating = dictionary[@"rating"];
	
	photomodel.thumbnailURL = [self urlForImageSize:3 inArray:dictionary[@"images"]];
	
	//Extended attributes fetched with subsequent request
	if (dictionary[@"comments_count"]){
		photomodel.fullsizedURL = [self urlForImageSize:4 inArray:dictionary[@"images"]];
	}
}
```
除了URL的属性设置，都是最基本的东西。依靠其他的方法来从500px的API中返回的图片列表中提取正确的url信息。500px API返回的数据结构是下面这样的格式：

```
(
	{
		size = size;
		url = ...;
	}
)
```

这是一个字典数组，每一个字典中包含一个`size`字段和一个`url`字段。我们读取这样字段的方法如下：

```
+ (NSString *)urlForImageSize:(NSInteger)size inDictionary:(NSArray *)array{
	return [[[[[array rac_sequence] filter:^ BOOL (NSDictionary * value){
		return [value[@"size"] integerValue] == size;
	}] map:^id (id value){
		return value[@"url"];
	}] array] firstObject];
}
```
这里有一些隐含的错误处理，如果序列为空，`NSArray`的`firstObject`方法默认返回nil.

 - 第一步，我们过滤掉那些`size`字段不匹配要求的字典。
 - 然后，将这些符合要求的字典做一次映射来提取字典中`url`字段的内容。
 - 最后，我们获得一个NSString 对象的序列，把它转化为数组，然后返回`firstObject`.
 
> 这里插图一个

在ReactiveCocoa中类似上面的链式调用非常常见。值从`rac_sequence`推送到`filter:`方法中，最后推送到`map:`方法里。最后调用序列`rac_sequence`的`array`方法，将序列的结果转化为`array`.

最后，我们的`downloadThumbnailForPhotoModel:`方法，看起来应该是下面这样：

```
+ (void)downloadThumbnailForPhotoModel:(FRPPhotoModel *)photoModel{
	NSAssert(photoModel.thumbnailURL, @"Thumbnail URL must not be nil");
	
	NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:photoModel.ThumbnailURL]];
	[NSURLConnection sendAsynchronousRequest:request 
		queue:[NSOperationQueue mainQueue] 
		completionHandler:^(NSURLResponse *response, NSData *data, NSError * connectionError){
			photoModel.thumbnailData = data;
	}];
}
```

这个方法里面没有任何的关于`Reactive`的部分---仅仅是下载thumbnail的url，然后在完成块中适当地设置相关属性。

我们几乎做完了这个画廊所需要的所有基础的事情，接下来，我们看看`viewController`.在实现文件里定义下面的的私有属性。

```
@interface FRPGalleryViewController ()
@property (nonatomic , strong) NSArray *photoArray;

@end
```
来看下viewDidLoad中的实现。

```
static NSString * CellIdentifier = @"Cell";

- (void)viewDidLoad{
	[super ViewDidLoad];
	
	//Configure self
	self.title = @"Popular on 500px";
	
	//Configure View
	[self.collectionView registerClass:[FRPCell class] forCellWithReuseIdentifier:CellIdentifier];
	
	//Reactive Stuff
	@weakify(self);
	[RACObserver(self, photosArray) subscribeNext:^(id x){
		@strongify(self);
		[self.collectionView reloadData];
	}];
	
	//Load data
	[self loadPopularPhotos];
}
```

我们为viewController设置了一个title并且为collectionView注册了一个类，collectionView将会在他的cells中复用这个类的实例。这里我引用了一个不存在的UICollectionViewCell的子类，我们很快会创建她。

在'Reactive Stuff'注释之下，你会发现一些奇怪的语法。

```
@weakify(self);
[RACObserver(self, photosArray) subscribeNext:^(id x){
	@strongify(self);
	[self.collectionView reloadData];
}];
```
`RACObserver`是一个C的宏定义，带两个参数：对象及对象某个属性的`keyPath`（关键路径）。他会返回一个带属性值的信号,无论这个属性的值怎么变都会及时地通过该信号反馈出来。在这里当self结束分配的时候会发送一个`completion Value`的值。订阅这个信号的目的是无论我们的photosArray中的元素属性怎么变，我们都能够在collectionView重新加载的时候实时获取反馈。

在Objective-C的ARC条件下@weakify/@strongify这个双人舞是非常常见的。@weakify创建一个新的self的弱引用weakself，@strongify创建这个weakself的强引用，并在@strongify的作用域中起作用。strongify的这种做法，一般称为“影子变量”，那是因为这个新的强引用的变量就叫`self`,替代了原本强引用的self.








