---
layout: post
title: "Accessing Camera and Photos in iOS"
comments: true
published: true
categories: ios
disqus: y
tags:
 - iOS
 - Camera
 - Photos
---

## Accessing camera

### Requesting access for media types

One can request access to the camera/microphone using `AVCaptureDevice::requestAccessForMediaType:completionHandler:`. Note however that `completionHandler` will be called in a background thread. If you want to do some UIKit operation, you need to [dispatch to the main queue](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVCaptureDevice_Class/#//apple_ref/occ/clm/AVCaptureDevice/requestAccessForMediaType:completionHandler:).
One good approach is the following:

```objectivec
    void (^cb)(BOOL) = ^void(BOOL granted) {
        if (!granted) {
            dispatch_async(dispatch_get_main_queue(), ^{
                //show alert
            });
        }
    };
    
    
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    if (status == ALAuthorizationStatusNotDetermined) {
        [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:cb];
    } else {
        cb(status == AVAuthorizationStatusAuthorized);
    }
```

## Accessing photos

### Requesting access
Same process as with AV, we must check the `Photos` framework.

```objectivec
void (^cb)(PHAuthorizationStatus) = ^void(PHAuthorizationStatus status) {
        if (status == PHAuthorizationStatusDenied || status == PHAuthorizationStatusRestricted) {
   	   //handle denied access                     
        }
    };
    PHAuthorizationStatus status = [PHPhotoLibrary authorizationStatus];
    if (status == PHAuthorizationStatusNotDetermined) {
        [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus newStatus) {
            cb(newStatus);
        }];
    } else {
        cb(status);
    }
```
In this case, requestAuthorization's callback is called in the same thread. Note that, until the user has seen the permission prompt, calls to fetch images from the library will return empty objects, and attempting to write files into the library will fail.
