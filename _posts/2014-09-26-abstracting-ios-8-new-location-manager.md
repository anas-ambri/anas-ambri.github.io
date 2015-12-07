---
layout: post
title: "Abstracting iOS 8 new Location Manager"
categories: tech
tags: iOS 8
 -
---

In the new iOS release, [CLLocationManager](https://developer.apple.com/library/prerelease/iOS/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html), the class responsible for retrieving location, gets a major overhaul, by introducing specific permission methods. While you can get some details on these changes by looking at the [documentation](https://developer.apple.com/library/prerelease/iOS/documentation/CoreLocation/Reference/CLLocationManager_Class/index.html), I wanted to introduce a wrapper class we are using at Guestful for wrapping all the Location-retrieval logic, especially the backward-compatibility code. This solution is based off Michael Babiy's [code](http://www.michaelbabiy.com/cllocationmanager-singleton/), which exposes a method for enabling a Singleton CLLocationManager class. The method exposed here takes this even further, by allowing any number of "observers" on this class.

##The Code
We start off by exposing a protocol for our observers to implement, in our `GFLocationManager.h` file:

```objc
#import <Foundation/Foundation.h>
#import <CoreLocation/CoreLocation.h>

@protocol GFLocationManagerDelegate

- (void)locationManagerDidUpdateLocation:(CLLocation *)location;

@end

@interface GFLocationManager : NSObject<CLLocationManagerDelegate>

+ (GFLocationManager *)sharedInstance;
- (void) addLocationManagerDelegate:(id<GFLocationManagerDelegate>) delegate;
- (void) removeLocationManagerDelegate:(id<GFLocationManagerDelegate>) delegate;

@end
```

The interface for our GFLocationManager is pretty straightforward: we provide with a `sharedInstance` method (singleton, duh!) and two methods to add/remove our observer. Don't forget to implement the protocol `CLLocationManagerDelegate`.

On the `GFLocationManager.m` side, we initiate our `manager` by following the new iOS 8 requirements, and simply broadcast any new location updates to the observers:

```objc
#import "GFLocationManager.h"

@interface GFLocationManager()

@property (strong, nonatomic) CLLocationManager* manager;
@property (strong, nonatomic) NSMutableArray *observers;

@end

@implementation GFLocationManager
static int errorCount = 0;
#define MAX_LOCATION_ERROR 3

//This is a singleton the objective-c way
+ (GFLocationManager*) sharedInstance {
    static GFLocationManager *sharedInstance = nil;
    static dispatch_once_t token;
    dispatch_once(&token, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}

- (instancetype) init {
    self = [super init];
    if(self) {
        
	//Must check authorizationStatus before initiating a CLLocationManager
        CLAuthorizationStatus status = [CLLocationManager authorizationStatus];
        if (status == kCLAuthorizationStatusRestricted && status == kCLAuthorizationStatusDenied) {
        } else {
            _manager = [[CLLocationManager alloc] init];
            _manager.delegate = self;
            _manager.desiredAccuracy = kCLLocationAccuracyBest;
        }
        if (status == kCLAuthorizationStatusNotDetermined) {
            //Must check if selector exists before messaging it
            if ([_manager respondsToSelector:@selector(requestWhenInUseAuthorization)]) {
                [_manager requestWhenInUseAuthorization];
            }
        }
        
        _observers = [[NSMutableArray alloc] init];
    }
    return self;
}

- (void) addLocationManagerDelegate:(id<GFLocationManagerDelegate>)delegate {
    if (![self.observers containsObject:delegate]) {
        [self.observers addObject:delegate];
    }
    [self.manager startUpdatingLocation];
}

- (void) removeLocationManagerDelegate:(id<GFLocationManagerDelegate>)delegate {
    if ([self.observers containsObject:delegate]) {
        [self.observers removeObject:delegate];
    }
}


#pragma mark - Location Manager Delegate

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
{
    [self.manager stopUpdatingLocation];
    for(id<GFLocationManagerDelegate> observer in self.observers) {
        if (observer) {
            [observer locationManagerDidUpdateLocation:[locations lastObject]];
        }
    }
}

-(void) locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error
{
    errorCount += 1;
    if(errorCount >= MAX_LOCATION_ERROR) {
        [self.manager stopUpdatingLocation];
	errorCount = 0;
    }
}

@end
```

Note a few things:
- The location manager will stop trying to update location if it fails three consecutive times. This is just a heuristic value; I would be interested to know if there are cases where this won't be enough.
- We kick off a retrieval of location every time an observer requests it. This works because `CLLocationManager` takes care of caching the value 

##How will you use this?
- In your viewController, start by implementing the `GFLocationManagerDelegate` protocol:

```objc
- (void) locationManagerDidUpdateLocation:(CLLocation *)location {
    self.currentLocation = location;
}
```
- Then, you can add/remove `self` in the `viewWillAppear` and `viewWillDisappear` methods of your viewController:

```objc
-(void) viewWillAppear:(BOOL)animated
{
    [[GFLocationManager sharedInstance] addLocationManagerDelegate:self];
}

- (void) viewWillDisappear:(BOOL)animated {
    [[GFLocationManager sharedInstance] removeLocationManagerDelegate:self];
}
```

Now, everytime the location is retrieved, the `locationManagerDidUpdateLocation` will be called.


###Improvements
Introduce time-based refreshing of the location: if `addLocationManagerDelegate` has not been called in 5 min, schedule it again.
