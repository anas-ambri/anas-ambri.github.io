--
layout: post
title: "The case against Parse's `@dynamic`/`@nsmanaged`"
comments: true
categories: ios
published: true
disqus: y
tags: ios
 - Parse
---

`@dynamic`/`@nsmanaged` just tells the compiler that the getter and setter methods are implemented not by the class itself but somewhere else (like the superclass or will be provided at runtime). This is used in the following cases:

- When using Parse, when subclassing `PFObject<PFSubclassing>`. 
- When using CoreData, when subclassing `NSManagedObject`

By default, a property is `@synthetize`, so you have to explicitly set it.

In both cases, it is the parent class that takes care of implementing the getter and setter.

## The problem with this approach

Say, you have a flag on your user object, used to determine if the user can buy alcohol (minors can't). This would look like this:

```objectivec
@interface User : PFUser <PFSubclassing>
@property (retain) NSNumber *canBuyAlcohol;
@end

@implementation User
@dynamic canBuyAlcohol;
@end

```

Ultimately, in your code, you will be using this to determine if you can allow a user to buy alcohol.

```objectivec
User* user = [User currentUser];
if([user.canBuyAlcohol boolValue]) {
    self.buyAlcoholButton.hidden = YES;
}
```

Now, what happens when the definition of `canBuyAlcohol` changes? Say, for example, that the following corner cases are added:

- A user is allowed to buy alcohol if they are doing it for their job
- A user is allowed to buy alcohol if they are carrying a prescription from a doctor
etc.

Because we don't have the ability to override the getter/setter of the property, changing the logic *requires* applying the new logic everywhere the flag is used. Basically, copy-pasting code like the following everywhere:

```objectivec
User* user = [User currentUser];
if([user.canBuyAlcohol boolValue]
    ||
    ((user.prescription != nil && [user.prescription contains:@"Alcohol"])
    ||
    (user.job != nil && [user.job.qualifications contains:@"Alcohol"]))
    ) {
    self.buyAlcoholButton.hidden = YES;
}
```

And I mean, **everywhere**!

Of course, all of this could've been avoided if we can override the getter of `canBuyAlcohol`, to avoid copy-pasting.

And, it turns out, that is possible!

## Overriding getter/setter for CoreData property

Custom getter

```objectivec
- (NSObject*) getFoo {
    [self willAccessValueForKey:@"foo"];
    NSString *foo = [self primitiveValueForKey:@"foo"];
    [self didAccessValueForKey:@"foo"];
    return foo;
}
```

Custom Setter

```objectivec
- (void) setFoo:(NSObject *)inFoo {
  [self willChangeValueForKey:@"foo"];
  [self setPrimitiveValue:inFoo forKey:@"foo"];
  [self didChangeValueForKey:@"foo"];
}
```


## Can you do the same thing with Parse?

Technically, you can't. Parse doesn't have a *public* interface of what it does inside [its accessors](https://parse.com/questions/subclassing-pfobject-while-overriding-dynamically-added-accessors). However, looking at their open-source libraries, we can probably guess. The problem is that they could always change it, and break whatever functionality was built on top (and yes, those bugs would be impossible to track).


## So, what's the solution?

Always define your own getters/setters for anything that can involve some kind of validation. For our `canBuyAlcohol` example, consider this:

```objectivec
@interface User : PFUser <PFSubclassing>
@property (retain) NSNumber *canBuyAlcohol;
- (BOOL) canBuyAlcoholCheck;
@end

@implementation User
@dynamic canBuyAlcohol;
- (BOOL) canBuyAlcoholCheck {
  //Do check
}
@end

```

You need to make sure that the method doesn't have the same name as the property, otherwise you will run into an infinite loop.



### Further reading

- [What are the differences between synthesize and dynamic?](http://stackoverflow.com/questions/1160498/synthesize-vs-dynamic-what-are-the-differences)
- [Custom setter for CoreData property](http://stackoverflow.com/questions/2971806/custom-setter-methods-in-core-data)
- [Custom getter for CoreData property](http://stackoverflow.com/questions/15853696/with-coredata-if-i-have-an-dynamic-property-can-i-override-its-getter-just-li)
- [Why getters and setters are important](http://stackoverflow.com/a/1568230/2426818)
