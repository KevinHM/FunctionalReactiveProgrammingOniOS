# 网络层回访

还有一个机会来进一步接受我们函数反应型编程的理念，那就是我们的网络层 `FRPPhotoImporter`,我们先来看看下载图片的方法：

```
+ (void)downloadThumbnailForPhotoModel:(FRPPhotoModel *)photoModel {
	[self download:photoModel.thumbnailURL withCompletion:^(NSData *data) {
		photoModel.thumbnailData = data;
	}];
}

+ (void)downloadFullsizedImageForPhotoModel:(FRPPhotoModel *)photoModel {
	[self download:photoModel.fullsizedURL withCompletion:^(NSData *data){
		photoModel.fullsizedData = data;
	}];
}

+ (void)download:(NSString *)urlString withCompletion:(void (^)(NSData *data))completion {
	NSAssert(urlString, @"URL must not be nil");
	
	NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:urlString]];
	[NSURLConnection sendAsynchronousRequest:request 
								queue:[NSOperationQueue mainQueue] 
								completionHandler:
									 ^(NSURLResponse *response, NSData *data, NSError *connectionError) {
											if(completion) {
												completion(data);
											}  
									 }];
}

```
Completion blocks?这是另外一个使用Signals的机会。更深入一点来说，我们可以使用`NSURLConnection`的ReactiveCocoa的扩展。下面我们来重写上面的方法：

```
+ (void)downloadThumbnailForPhotoModel:(FRPPhotoModel *)photoModel {
	RAC(photoModel, thumbnailData) = [self download:photoModel.thumbnailURL];
}

+ (void)downloadFullsizedImageForPhotoModel:(FRPPhotoModel *)photoModel {
	RAC(photoModel,fullsizedData) = [self download:photoModel.fullsizedURL];
}

+ (RACSignal *)download:(NSString *)urlString {
	NSAssert(urlString , @"URL must not be nil");
	
	NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString: urlString]];
	
	return [[[NSURLConnection rac_sendAsynchronousRequest:request] 
				map:^id (RACTuple *value) {
					return [value second];
				}] deliverOn:[RACScheduler mainThreadScheduler]];
}

```
这里有两个大的不同：

  1. asdf
  2. asdf
