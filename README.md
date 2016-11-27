# Objective-C Style Guide at OneByte

This style guide outlines the coding norms of iOS development at OneByte. 

## Introduction

Here are some of the documents from Apple that informed the style guide. If something isn’t mentioned here, it’s probably covered in great detail in one of these:

* [The Objective-C Programming Language](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App Programming Guide](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)

## Table of Contents

* [Notation Syntax](#notation-syntax)
* [Spacing](#spacing)
* [Conditionals](#conditionals)
  * [Ternary Operator](#ternary-operator)
* [Error handling](#error-handling)
* [Methods](#methods)
* [Variables](#variables)
* [Naming](#naming)
* [Categories](#categories)
* [Comments](#comments)
* [Init & Dealloc](#init-and-dealloc)
* [Literals](#literals)
* [Strings](#strings)
* [Navigation Bar](#navigation-bar)
* [CGRect Functions](#cgrect-functions)
* [Constants](#constants)
* [Enumerated Types](#enumerated-types)
* [Bitmasks](#bitmasks)
* [Properties](#properties)
* [Image Naming](#image-naming)
* [Booleans](#booleans)
* [Singletons](#singletons)
* [Imports](#imports)
* [Protocols](#protocols)
* [Multi-device support](#multi-device-support)
* [Auto-Generated Code](#auto-generated-code)
* [Super Class](#super-class)
* [Xcode Project](#xcode-project)

## Notation Syntax 

**Dot** notation should **always** be used for accessing and mutating properties.
Bracket notation is preferred in all other instances. It is harmful to switch between the notations in the similar instances because that would bring inconsistency in the code.

**For Example:**
```objc
contentView.backgroundColor = [UIColor clearColor];
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[contentView setBackgroundColor:[UIColor clearColor]];
UIApplication.sharedApplication.delegate;
```

## Spacing

* There should be exactly one blank line between methods to aid in visual clarity and organization.
* Whitespace within methods should be used to separate functionality (though often this can indicate an opportunity to split the method into several, smaller methods). In methods with long or verbose names, a single line of whitespace may be used to provide visual separation before the method’s body.
* `@synthesize` and `@dynamic` should each be declared on new lines in the implementation.

## Conditionals

Conditional bodies should always use braces even when a conditional body could be written without braces (e.g., it is one line only). These errors include adding a second line and expecting it to be part of the if-statement. Another, [even more dangerous defect](http://programmers.stackexchange.com/a/16530) may happen where the line “inside” the if-statement is commented out, and the next line unwittingly becomes part of the if-statement. In addition, this style is more consistent with all other conditionals, and therefore more easily scannable.

**For example:**
```objc
if (!error) {
    return success;
}
```

**Not:**
```objc
if (!error)
    return success;
```

or

```objc
if (!error) return success;
```

### Ternary Operator

The ternary operator, `?` , should only be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an if statement, or refactored into named variables.

**For example:**
```objc
result = a > b ? x : y;
```

**Not:**
```objc
result = a > b ? x = c > d ? c : d : y;
```

## Error Handling

When methods return an error parameter by reference, switch on the returned value, not the error variable.

**For example:**
```objc
NSError *error;
if (![self trySomethingWithError:&error]){
    // Handle Error
}
```

**Not:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

## Methods

In method signatures, there should be a space after the scope (`-` or `+` symbol). Methods should be divided into self explanatory, logical segments and there should be space between them.

**For example:**
```objc
- (void)setUserWithName:(NSString *)name andWithProfilePicture:(UIImage *)profilePicture;
```

**Not:**
```objc
-(void)setUserWithNameAndProfilePicture:(NSString *)name image:(UIImage *)profilePicture;
```

Avoid extra re-factoring i.e. methods with less implementation should be placed in actual code flow instead of putting them in the methods.

**For example:**
```objc
 if (![self.emailTextField isEqualToString:@""]) {
   [self showAlertViewWithTitle:@"Signup Failed" andWithMessage:@"Email field is missing"];
		 return false;
 }
```

**Not:**
```objc
 if (![self emptyFieldValidation]){
   [self showAlertViewWithTitle:@"Signup Failed" andWithMessage:@"Email field is missing"];
		 return false;
 }
 
 - (BOOL)emptyFieldValidation:(NSString *)text{
    if([text isEqualToString:@""])
    {
       return true;
    }
    return false;
 }
```

## Variables

Variables should be self explanatory, with the variable’s name clearly communicating what the variable _is_ and how it can be used.

**For example:**

* `NSString *title`: It is reasonable to assume a “title” is a string.
* `NSString *titleHTML`: This indicates a title that may contain HTML which needs parsing for display. _“HTML” is needed for a programmer to use this variable effectively._
* `NSAttributedString *titleAttributedString`: A title, already formatted for display. _`AttributedString` hints that this value is not just a vanilla title, and adding it could be a reasonable choice depending on context._
* `NSDate *now`: _No further clarification is needed._
* `NSDate *lastModifiedDate`: Simply `lastModified` can be ambiguous; depending on context, one could reasonably assume it is one of a few different types.
* `NSURL *URL` vs. `NSString *URLString`: In situations when a value can reasonably be represented by different classes, it is often useful to disambiguate in the variable’s name.
* `NSString *releaseDateString`: Another example where a value could be represented by another class, and the name can help disambiguate.

Single letter variable names should be avoided except as simple counter variables in loops.

Asterisks indicating a type is a pointer should be “attached to” the variable name. 

**For example,** `NSString *text` **not** `NSString* text` or `NSString * text`, except in the case of constants (`NSString * const NYTConstantString`).

Property definitions should be used in place of naked instance variables whenever possible.
Direct instance variable access should be avoided except in initializer methods (`init`, `initWithCoder:`, etc…), `dealloc` methods and within custom setters and getters. For more information, see [Apple’s docs on using accessor methods in initializer methods and `dealloc`](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6).

**For example:**

```objc
@interface Tool: NSObject

@property (nonatomic) NSString *toolName;

@end
```

**Not:**

```objc
@interface Section : NSObject {
    NSString *toolName;
}
```
## Comments

When they are needed, comments should be used to explain **why** a particular piece of code does something e.g. complex algorithm steps. And there is no need to put comments in front of self explanatory variables and methods.
Any comments that are used must be kept up-to-date or deleted.

Block comments should generally be avoided, as code should be as self-documenting as possible, with only the need for intermittent, few-line explanations. **Note:** This does not apply to those comments used to generate documentation.

## init and dealloc

`init` methods should be placed immidiately after the @implemenmtation keyword. The dealloc method should come directly afterwards.

`init` methods should be structured like this:

```objc
- (id)init
{
    self = [super init];
    
    if(self)
    {
        //Code
    }
    
    return self;
}
```

## Literals

`NSString`, `NSDictionary`, `NSArray`, and `NSNumber` literals should be used whenever creating immutable instances of those objects. Pay special care that `nil` values not be passed into `NSArray` and `NSDictionary` literals, as this will cause a crash.

**For example:**

```objc
NSArray *departments = @[@"WMS", @"GI", @"Sales", @"Management"];
NSDictionary *onebyteDevelopers = @{@"iOS" : @"Hammy", @"Android" : @"Faraz", @"Web" : @"Faheem"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *lahoreZipCode = @54570;
```

**Not:**

```objc
NSArray *departments = [NSArray arrayWithObjects:@"WMS", @"Sales", @"GI", @"Management", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *lahoreZipCode = [NSNumber numberWithInteger:54570];
```
## Strings

`NSString` should be initialize/update using simpler approach.

**For example:**

```objc
firstNameLabel.text = self.firstName;
```
**Not:**

```objc
firstNameLabel.text = [NSString stringWithFormat:@"%@", self.firstName];
```
## Navigation Bar

Default iOS Navigation Bar should be use instead of creating custom Navigation Bar view class. This simply give default iOS navigational experience and makes the code simple and less.

**For example:**

```objc
- (void)viewWillAppear:(BOOL)animated{
    [super viewWillAppear:animated];

    self.navigationItem.title = @"Fitness";
    
    self.navigationController.navigationBar.barTintColor = [UIColor blueColor];
}
```

**Not:**

```objc
-(void)initializeNavgationBar
{
 self.navigationController.navigationBarHidden = YES;
 
 self.navigationBarViewController = [[NavigationBarViewController alloc] initWithNibName:@"NavigationBarViewController" bundle:nil];

 self.navigationBarViewController.navigationBarTitleText = [NSString 
stringWithFormat:@"Suggestions For You"];

 self.navigationBarViewController.navigationBarBackgroundHexValue = [NSString 
stringWithFormat:@"F9C21C"];

 self.navigationBarViewController.statusBarBackgroundHexValue = [NSString 
stringWithFormat:@"BD9217"];

 self.navigationBarViewController.backButtonShow = YES;

 self.navigationBarViewController.view.frame = CGRectMake(0, 0, self.view.frame.size.width, 

64);

 [self.view addSubview:self.navigationBarViewController.view];
}
```

## `CGRect` Functions

When accessing the `x`, `y`, `width`, or `height` of a `CGRect`, always use the [`CGGeometry` functions](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html) instead of direct struct member access. From Apple's `CGGeometry` reference:

> All functions described in this reference that take CGRect data structures as inputs implicitly standardize those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**For example:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**Not:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

## Constants

Constants are preferred over in-line string literals or numbers, as they allow for easy reproduction of commonly used variables and can be quickly changed without the need for find and replace. Constants should be declared as `static` constants and not `#define`s unless explicitly being used as a macro.

There should be a a separate class for Constant named `Constants.h` and `Constants.m` so there would be effect at a single place instead of multiple places wherever constants are used.

**For example:**

Constants.h

```objc
extern NSString * const 1BProductManagerName;

extern int const 1BCurrentEmployeeCount;
```

Constants.m

```objc
NSString * const 1BProductManagerName = @"Muneeb Dilshad";

int constant 1BCurrentEmployeeCount = 50;
```

**Not:**

```objc
#define productManagerName @"Muneeb Dilshad"

#define currentEmployeeCount 50
```

If it is intended to use within the class, it should be declared like so,
```objc
static NSString * const ORSProductManagerName = @"Muneeb Dilshad";";
```

## Enumerated Types

When using `enum`s, use the new fixed underlying type specification, which provides stronger type checking and code completion. The SDK includes a macro to facilitate and encourage use of fixed underlying types: `NS_ENUM()`.

**Example:**

```objc
typedef NS_ENUM(NSInteger, ToolDataRange)
{
    ToolDataRangeWeekly = 0,
    ToolDataRangeDaily,
    ToolDataRangeMonthly,
    ToolDataRangeCount,
};
```

## Bitmasks

When working with bitmasks, use the `NS_OPTIONS` macro.

**Example:**

```objc
typedef NS_OPTIONS(NSUInteger, NYTAdCategory) {
    ORSDepartmentCGI      = 1 << 0,
    ORSDepartmentWMS      = 1 << 1,
    ORSDepartmentGI       = 1 << 2,
    ORSDepartmentBD       = 1 << 3
};
```

## Properties

Public properties should be declated in header file while private properties should be declared in the implementation file under ```@interface ViewController ()``` of a class. `nonatomic, copy` should be use for creating property of any `NSMutableArray`, `NSArray`, `NSMutableDictionary`, `NSDictionary` while `nonatomic, assign` should be use for `int`, `float`, `bool`, etc and `nonatomic, retain` for rest of the most cases.

**For example:**

```objc
@interface ToolViewController ()

@property (nonatomic, copy) NSArray *tools;

@property (nonatomic, strong) UIButton *settingsButton;
@property (nonatomic, strong) UIButton *infoButton;

@property (nonatomic, assign) int toolIndex;

@property (nonatomic, assign) bool isToolSelected;

@end
```
Properties should be grouped together on the basis of their data types.

Public ```@properties``` should not be ```@synthesized``` because this is an old convention which is no longer necessary and they can be accessed through ```self```.


## Image Naming

Image names should be named consistently to preserve organization and developer sanity. They should be named as one camel case string with a description of their purpose, followed by the un-prefixed name of the class or property they are customizing (if there is one), followed by a further description of color and/or placement, and finally their state.

**For example:**

* `RefreshBarButtonItem` / `RefreshBarButtonItem@2x` and `RefreshBarButtonItemSelected` / `RefreshBarButtonItemSelected@2x`
* `ArticleNavigationBarWhite` / `ArticleNavigationBarWhite@2x` and `ArticleNavigationBarBlackSelected` / `ArticleNavigationBarBlackSelected@2x`.

Asset Catalogue should be used to organise and manage any image assets in you application.

## Booleans

Never compare something directly to `YES`, because `YES` is defined as `1`, and a `BOOL` in Objective-C is a `CHAR` type that is 8 bits long (so a value of `11111110` will return `NO` if compared to `YES`).

**For an object pointer:**

```objc
if (!someObject) {
}

if (someObject == nil) {
}
```

**For a `BOOL` value:**

```objc
if (isAwesome)
if (!someNumber.boolValue)
if (someNumber.boolValue == NO)
```

**Not:**

```objc
if (isAwesome == YES) // Never do this.
```

If the name of a `BOOL` property is expressed as an adjective, the property’s name can omit the `is` prefix but should specify the conventional name for the getter.

**For example:**

```objc
@property (assign, getter=isEditable) BOOL editable;
```

_Text and example taken from the [Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)._

## Singletons

Singleton objects should use a thread-safe pattern for creating their shared instance.
```objc
+ (instancetype)sharedInstance {
    static id sharedInstance = nil;

    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[[self class] alloc] init];
    });

    return sharedInstance;
}
```
This will prevent [possible and sometimes frequent crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

## Imports

If there is more than one import statement, group the statements [together](http://ashfurrow.com/blog/structuring-modern-objective-c). Commenting each group is optional.

Note: For modules use the [@import](http://clang.llvm.org/docs/Modules.html#using-modules) syntax.

```objc
// Frameworks
@import QuartzCore;

// Models
#import "ORSUser.h"

// Views
#import "CustomButton.h"
#import "CustomUserView.h"
```

## Protocols

In a [delegate or data source protocol](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html), the first parameter to each method should be the object sending the message.

This helps disambiguate in cases when an object is the delegate for multiple similarly-typed objects, and it helps clarify intent to readers of a class implementing these delegate methods.

**For example:**

```objc
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
```

**Not:**

```objc
- (void)didSelectTableRowAtIndexPath:(NSIndexPath *)indexPath;
```

There should always be implementation in the protocols methods, in the case if it is left for future use, they should be implemented with ```NSLog(@"Function Name");```

## Auto-Generated Code

Any auto-generated code or overridden methods that do not extend the super method in any way should be
deleted as they are redundant.This helps declutter the class.

## Multi-Device Support

**Auto-layout** should be applied in all interface files to support multiple iOS Device resolutions. It is great to give this support before moving to next view and it is recommended to apply it through Interface Builder.

## Super Class

Same natured views with different data used in numerous views should be added through super class view and customised accordingly in order to enhance re-usability and decrease development time. 

## Xcode project

The physical files should be kept in sync with the Xcode project files in order to avoid file sprawl. Any Xcode groups created should be reflected by folders in the filesystem. Code should be grouped not only by type, but also by feature for greater clarity.

When possible, always turn on “Treat Warnings as Errors” in the target’s Build Settings and enable as many [additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings) as possible. If you need to ignore a specific warning, use [Clang’s pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas).

# Other Objective-C Style Guides

If ours doesn’t fit your tastes, have a look at some other style guides:

* [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/2011/06/28/my-objective-c-style-guide/)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)
* [Wikimedia](https://www.mediawiki.org/wiki/Wikimedia_Apps/Team/iOS/ObjectiveCStyleGuide)
