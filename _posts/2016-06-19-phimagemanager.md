---
layout: post
title: "PHImageManager"
comments: true
categories: ios
published: true
disqus: y
tags:
 - ios
---

`ALAsset` framework has been deprecated. `PHImageManager` is a new API that acts as a centralized coordinator for [image and video assets](http://nshipster.com/phimagemanager/). From NSHispter:

> Previously, each app was responsible for creating and caching their own image thumbnails. In addition to requiring extra work on the part of developers, redundant image caches could potentially add up to gigabytes of data across the system. But with PHImageManager, apps don’t have to worry about resizing or caching logistics, and can instead focus on building out features.

It provides synchronous opportunistic and asynchronous loading, as well as caching mechanisms. But first, we'll cover how to retrieve the assets.

## Fetching assets
Here is an example for pre-fetching 5 images, sorted by descending creation date order:

```objectivec
@import Photos;

@property (nonatomic, strong) NSArray<PHAsset *> *assets;

PHFetchOptions *options = [[PHFetchOptions alloc] init];
options.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:NO]];
            
PHFetchResult *results = [PHAsset fetchAssetsWithMediaType:PHAssetMediaTypeImage options:options];
long numberImages = 5;

[results enumerateObjectsUsingBlock:^(id  _Nonnull object, NSUInteger idx, BOOL * _Nonnull stop) {
    if ([object isKindOfClass:[PHAsset class]]) {
        [self.assets addObject:object];
    }
    if (idx == numberImages - 1) {
        *stop = YES;
    }
}];
```
If you are looking to reverse enumerate, use `enumerateObjectsWithOptions:usingBlock:` passing `NSEnumerationReverse` for [the options](http://stackoverflow.com/questions/13301600/objective-c-block-enumeration-for-array-in-reverse-order).

## Asynchronous loading

The basic usage example is as follows (taken from nshipster):

```objectivec
@property (nonatomic, strong) PHImageManager* manager;

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Cell" forIndexPath:indexPath];

    if (cell.tag) {
        [self.manager cancelImageRequest:(PHImageRequestID)cell.tag];
    }

    PHAsset *asset = self.assets[indexPath.row];

    cell.tag = [manager requestImageForAsset:asset
                                  targetSize:CGSizeMake(100.0, 100.0)
                                 contentMode:PHImageContentModeAspectFill
                                     options:nil
                               resultHandler:^(UIImage * _Nullable result, NSDictionary * _Nullable info) {
                                   cell.imageView.image = result;
                               }];
    return cell;
}
```

By default, the call will be asynchronous, which means `resultHandler` will be called after a result is available. This means that the `imageView`s might display a small delay before displaying the asset (See Opportunistic synchronous loading to avoid this problem).

## Pre-caching

If you are certain all your assets will need to be displayed, or have a limit of images to load, you can cache them using a `PHCachingImageManager`. Here is an example:

```objectivec
PHCachingImageManager* cachingImageManager = [PHCachingImageManager new];
cachingImageManager.allowsCachingHighQualityImages = YES; //Set to NO if fast scrolling
[cachingImageManager startCachingImagesForAssets:self.assets
                                                  targetSize:CGSizeMake(100.0, 100.0)
                                                 contentMode:PHImageContentModeAspectFill
                                                     options:nil];
```
This way, the image will be ready when requested. It is also encouraged to call `stopCachingImagesForAssets:targetSize:contentMode:options` or `stopCachingImagesForAllAssets`. This is important when making multiple `startCachingImagesForAssets` requests with different parameters.

## Opportunistic synchronous loading
To avoid the problem of asynchronous loading of `requestImageForAsset:`, it is possible to ask for a low-quality photo to put as placeholder. To do that:

```objectivec
PHImageRequestOptions* options = [PHImageRequestOptions new];
options.synchronous = YES;
options.deliveryMode = PHImageRequestOptionsDeliveryModeOpportunistic;
[[PHImageManager defaultManager] requestImageDataForAsset:asset options:options resultHandler:^(NSData * _Nullable imageData, NSString * _Nullable dataUTI, UIImageOrientation orientation, NSDictionary * _Nullable info) {
		 //Handle image
            }];
```

In this case, `resultHandler` will be called twice:

- A first time, with the low-quality image, synchronously, called on the **calling thread**. This means that it is necessary to `dispatch` to main queue to interact [with `UIKit`](http://stackoverflow.com/questions/27670325/photos-framework-requestimageforasset-returning-two-results-cant-set-image-vi).
- A second time, with a higher quality image, called on the main thread.
`synchronous` has to be set to `true` for this to work.
