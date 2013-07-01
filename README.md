KIF iOS Integration Testing Framework
=====================================

KIF, which stands for Keep It Functional, is an iOS integration test framework. It allows for easy automation of iOS apps by leveraging the accessibility attributes that the OS makes available for those with visual disabilities.

KIF-next is a branch of KIF which builds and performs the tests using a standard `ocunit` testing target.  Testing is conducted synchronously (running the run loop to force the passage of time) rather than by constructing an array of steps allowing for more complex logic and composition.  This also allows KIF-next to take advantage of the Xcode 5 Test Navigator, command line build tools, and Bot test reports.  [Find out more about Xcode 5 features.](https://developer.apple.com/technologies/tools/whats-new.html)

**KIF uses undocumented Apple APIs.** This is true of most iOS testing frameworks, and is safe for testing purposes, but it's important that KIF does not make it into production code, as it will get your app submission denied by Apple. Follow the instructions below to ensure that KIF is configured correctly for your project.

There's [a port underway to use KIF with Mac OS X apps](https://github.com/joshaber/KIF), as well.

Features
--------

#### Minimizes Indirection
All of the tests for KIF are written in Objective C. This allows for maximum integration with your code while minimizing the number of layers you have to build.

#### Easy Configuration
KIF integrates directly into your iOS app, so there's no need to run an additional web server or install any additional packages.

#### Test Like a User
KIF attempts to imitate actual user input. Automation is done using tap events wherever possible.

#### Automatic Integration with Xcode 5 Testing Tools
Xcode 5 introduces [new testing an continuous integration tools](https://developer.apple.com/technologies/tools/whats-new.html) built on the same testing platform as KIF-next.  You can easily run a single KIF test with the Test Navigator or kick off nightly acceptance tests with Bots.

See KIF in Action
-----------------

KIF-next uses techniques described below to validate its internal functionality.  You can see a test suite that exercises its entire functionality by simply building and testing the KIF scheme with ⌘U.  Look at the tests in the "KIF Tests" group for ideas on how to build your own tests.

Installation
------------

To install KIF, you'll need to link the libKIF static library directly into your application. Download the source from the [bnickel/KIF/tree/kif-next](https://github.com/bnickel/KIF/tree/kif-next) and follow the instructions below.

*NOTE* These instruction assume you are using Xcode 4 or later. For Xcode 3 you won't be able to take advantage of Workspaces, so the instructions will differ slightly.

### Add KIF to your project files
The first step is to add the KIF project into the ./Frameworks/KIF subdirectory of your existing app. If your project uses Git for version control, you can use submodules to make updating in the future easier:

	cd /path/to/MyApplicationSource
	mkdir Frameworks
	git submodule add -b KIF-next https://github.com/square/KIF.git Frameworks/KIF

If you're not using Git, simply download the source and copy it into the ./Frameworks/KIF directory.

### Add KIF to Your Workspace
Let your project know about KIF by adding the KIF project into a workspace along with your main project. Find the KIF.xcodeproj file in Finder and drag it into the Project Navigator (⌘1). If you don't already have a workspace, Xcode will ask if you want to create a new one. Click "Save" when it does.

![Create workspace screen shot](https://github.com/square/KIF/raw/master/Documentation/Images/Create Workspace.png)

### Create a Testing Target
You'll need to create a test target for your app.  You may already have one named *MyApplication*_Tests if you selected to automatically create unit tests.  If you did, you can keep using it if you aren't using it for unit tests.  Otherwise, follow these directions to create a new one.

Select your project in Xcode and click on "Add Target" in the bottom left corner of the editor.  Select iOS -> Other -> Cocoa Touch Unit Testing Bundle.  Give it a product name like "Acceptance Tests", "UI Tests", or something that indicates the intent of your testing process.  You can select "Use Automatic Reference Counting" even if the remainder of your app doesn't, just to make your life easier. 

### Configure the Testing Target
Now that you have a target for your tests, add the tests to that target. With the project settings still selected in the Project Navigator, and the new integration tests target selected in the project settings, select the "Build Phases" tab. Under the "Link Binary With Libraries" section, hit the "+" button. In the sheet that appears, select "libKIF.a" and click "Add".  Repeat the process for CoreGraphics.framework.

![Add libKIF library screen shot](https://github.com/square/KIF/raw/master/Documentation/Images/Add Library.png)

![Add libKIF library screen shot](https://github.com/square/KIF/raw/master/Documentation/Images/Add Library Sheet.png)

KIF takes advantage of Objective C's ability to add categories on an object, but this isn't enabled for static libraries by default. To enable this, add the `-ObjC` flag to the "Other Linker Flags" build setting as shown below.

![Add category linker flags screen shot](https://github.com/square/KIF/raw/master/Documentation/Images/Add Category Linker Flags.png)

You need your tests to run hosted in your application.  To do this, first add your add your application by first selecting "Build Phases", expanding the "Target Dependencies" section, clicking on the "+" button, and in the new sheet that appears selecting your application target and clicking "Add".

Next, configure your bundle loader.  In "Build Settings", expand "Linking" and edit "Bundle Loader" to be `$(BUILT_PRODUCTS_DIR)/*My App*.app/*My App*` where *My App* is the name of your app.  Expand the "Unit Testing" section and edit "Test Host" to be `$(BUNDLE_LOADER)`.

The last step is to configure your unit tests to run when you trigger a test (⌘U).  Click on your scheme name and select "Edit Scheme…".  Click on "Test" in the sidebar followed by the "+" in the bottom left corner.  Select your testing target and click "OK".

Example
-------
With your project configured to use KIF, it's time to start writing tests. There are two main classes used in KIF testing: the the test case (`KIFTestCase`, subclass of `SenTestCase`) and the UI tester (`KIFTester`).  The the ocunit test runner loads the test case classes and executes their test.  Inside these tests, the tester performs the UI operations which generally imitate a user interaction. Three of the most common tester actions are "tap this view," "enter text into this view," and "wait for this view." These steps are included as factory methods on `KIFTester` in the base KIF implementation.

KIF relies on the built-in accessibility of iOS to perform its test steps. As such, it's important that your app is fully accessible. This is also a great way to ensure that your app is usable by the sight impaired. Making your application accessible is usually as easy as giving your views reasonable labels. More details are available in [Apple's Documentation](http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/iPhoneAccessibility/Making_Application_Accessible/Making_Application_Accessible.html#//apple_ref/doc/uid/TP40008785-CH102-SW5).

The first step is to create a test class to test some functionality.  In our case, we will create a login test (`LoginTests`). Create a new class that inherits from KIFTestCase.  You may have to update the import to point to `<KIF/KIF.h>`. The test method name provides a unique identifier. Your `KIFTestCase` subclass should look something like this:

*LoginTestCase.h*

	#import <KIF/KIF.h>
	
	@interface LoginTests : KIFTestCase
	@end

*LoginTestCase.m*

	#import "LoginTests.h"
	#import "KIFTester+EXAdditions.h"

	@implementation LoginTests

	- (void)beforeEach
	{
	    [tester navigateToLoginPage];
	}
	
	- (void)afterEach
	{
	    [tester returnToLoggedOutHomeScreen];
	}
	
	- (void)testSuccessfulLogin
	{
	    [tester enterText:@"user@example.com" intoViewWithAccessibilityLabel:@"Login User Name"];
	    [tester enterText:@"thisismypassword" intoViewWithAccessibilityLabel:@"Login Password"];
	    [tester tapViewWithAccessibilityLabel:@"Log In"];

	    // Verify that the login succeeded
	    [tester waitForTappableViewWithAccessibilityLabel:@"Welcome"];
	}
	
	@end

Most of the tester actions in the test are already defined by the KIF framework, but `-navigateToLoginPage` and `-returnToLoggedOutHomeScreen` are not. These are examples of custom actions which are specific to your application. Adding such steps is easy, and is done using a factory method in a category of `KIFTester`, similar to how we added the scenario.

*KIFTester+EXAdditions.h*

	#import <KIF/KIF.h>

	@interface KIFTester (EXAdditions)
	
	- (void)navigateToLoginPage;
	- (void)returnToLoggedOutHomeScreen;

	@end

* KIFTester+EXAdditions.m*

	#import "KIFTester+EXAdditions.h"

	@implementation KIFTester (EXAdditions)

	- (void)navigateToLoginPage
	{
	    [self tapViewWithAccessibilityLabel:@"Login/Sign Up"];
	    [self tapViewWithAccessibilityLabel:@"Skip this ad"];
	}

	- (void)returnToLoggedOutHomeScreen
	{
	    [self tapViewWithAccessibilityLabel:@"Logout"];
	    [self tapViewWithAccessibilityLabel:@"Logout"]; // Dismiss alert.
	}

	@end

Everything should now be configured. When you run the integration tests using the test button, ⌘U, or the Xcode 5 Test Navigator (⌘5).

Troubleshooting
---------------

### Step fails because a view cannot be found

If KIF is failing to find a view, the most likely cause is that the view doesn't have its accessibility label set. If the view is defined in a xib, then the label can be set using the inspector. If it's created programmatically, simply set the accessibilityLabel attribute to the desired label.

If the label is definitely set correctly, take a closer look at the error given by KIF. This error should tell you more specifically why the view was not accessible. If you are using `-waitForTappableViewWithAccessibilityLabel:`, then make sure the view is actually tappable. For items such as labels which cannot become the first responder, you may need to use `-waitForViewWithAccessibilityLabel:` instead.

### Unrecognized selector when first trying to run

If the first time you try to run KIF you get the following error:

	2011-06-13 13:54:53.295 Testable (Integration Tests)[12385:207] -[NSFileManager createUserDirectory:]: unrecognized selector sent to instance 0x4e02830
	2011-06-13 13:54:53.298 Testable (Integration Tests)[12385:207] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[NSFileManager createUserDirectory:]: unrecognized selector sent to instance 0x4e02830'

or if you get another "unrecognized selector" error inside the KIF code, make sure that you've properly set the -ObjC flag as described above. Without this flag your app can't access the category methods that are necessary for KIF to work properly.

Continuous Integration
----------------------

A continuous integration (CI) process is highly recommended and is extremely useful in ensuring that your application stays functional. The easiest way to do this will be with Xcode 5, either using Bots, or Jenkins or another tool that uses xcodebuild.  For tools using xcodebuild, review the manpage for instructions on using test destinations.

Contributing
------------

We're glad you're interested in KIF, and we'd love to see where you take it.

Any contributors to the master KIF repository must sign the [Individual Contributor License Agreement (CLA)](https://spreadsheets.google.com/spreadsheet/viewform?formkey=dDViT2xzUHAwRkI3X3k5Z0lQM091OGc6MQ&ndplr=1). It's a short form that covers our bases and makes sure you're eligible to contribute.

When you have a change you'd like to see in the master repository, [send a pull request](https://github.com/square/KIF/pulls). Before we merge your request, we'll make sure you're in the list of people who have signed a CLA.

Thanks, and happy testing!
