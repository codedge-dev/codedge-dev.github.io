---
layout: post
title:  "Presenting a Login View Controller on Launch"
date:   2015-04-06 00:16:59
author: henry
categories: ios
---
I've struggled with login view controllers for a long time, or really any initial user flow that can vary depending on what is set. Setting a user login/registration view controller as the default root view controller seems wrong-- it should instead be a modal view controller that is only used sometimes. Taking this other approach results in a brief second where the modal is being presented. While it works, it isn't clean. A login view should simply be there when it is needed.

The Solution: Changing the Root View Controller
-----
It turns out the root view controller must be changed to work as desired, but Xcode storyboards seem to lack the proper functionality to make this happen. Instead we must do it programmatically.

Change your `didFinishLaunchingWithOptions` like so, where `verifyLoginCredentials()` is whatever method you use to verify whether a user has already logged in.
{% highlight swift %}
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        let userAuth = verifyLoginCredentials()
        let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
        if userAuth {
            self.window?.rootViewController = storyboard.instantiateInitialViewController() as? UIViewController
        }
        else {
            var rootController = storyboard.instantiateViewControllerWithIdentifier("UserAuthViewController") as UIViewController!
            var navigation = UINavigationController(rootViewController: rootController!)
            self.window?.rootViewController = navigation
        }
        return true
    }
{% endhighlight %}

Now add the following lines your `signUp()` or `logIn()` method. This will switch to your root view controller back to your initial one set by the storyboard.
{% highlight swift %}
func signUp() {
    let appDelegateTemp = UIApplication.sharedApplication().delegate
    appDelegateTemp?.window??.rootViewController = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle()).instantiateInitialViewController() as? UIViewController
}
{% endhighlight %}

Is the user logging out? Add these lines to your logOut function to switch your root view controller back to the login/signup view controller.
{% highlight swift %}
func logOut() {
    let appDelegateTemp = UIApplication.sharedApplication().delegate
    let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
    var rootController = storyboard.instantiateViewControllerWithIdentifier("UserAuthViewController") as UIViewController!
    var navigation = UINavigationController(rootViewController: rootController!)
    appDelegateTemp?.window??.rootViewController = navigation
}
{% endhighlight %}

And thats it! No more modal presentations of signup views. It simply works!