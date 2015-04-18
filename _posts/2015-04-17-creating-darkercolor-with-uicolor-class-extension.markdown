---
layout: post
title:  "Creating darkerColor() UIColor Class Extension in Swift"
date:   2015-04-07 00:18:55
author: henry
categories: ios swift
---

Swift's strict typing is great, but sometimes it can be a major source of frustration for developers used to the conventions of Objective-C. Take the case of the following code, a class extension of UIColor written in ObjC that returns a 20% darker version of the color that calls it.

{% highlight objective-c %}
@implementation UIColor (UIColorAdditions)

- (UIColor *)darkerColor {
    CGFloat amount = 0.8;
    CGFloat r, g, b, a;
    [self getRed:&r green:&g blue:&b alpha:&a];
                
    return [UIColor colorWithRed:amount*r green:amount*g blue:amount*b alpha:a];
}

@end
{% endhighlight %}

All this code is doing is passing in the pointers of `r`,`g`,`b`, and `a`, which get set to the respective attributes of the color. To do the exact same thing in Swift requires only a few more lines, but a slightly different way of thinking.

{% highlight swift %}
extension UIColor {
    func darkerColor() -> UIColor {
        var amount: CGFloat = 0.9
        var rgba = UnsafeMutablePointer<CGFloat>.alloc(4)
        
        self.getRed(&rgba[0], green: &rgba[1], blue: &rgba[2], alpha: &rgba[3])
        var darkerColor = UIColor(red: amount*rgba[0], green: amount*rgba[1], blue: amount*rgba[2], alpha: rgba[3])
        
        rgba.destroy()
        rgba.dealloc(4)
        return darkerColor
    }
}
{% endhighlight %}

*So what's going on here?*

Instead of passing a simple pointer, Swift has a strictly-typed `UnsafeMutablePointer` that requires you to manually allocate and deallocate memory (ick). There isn't a lot of documentation on the matter, but the biggest thing to note is the use of `.alloc(4)` to instantiate it, where `4` is the number of words in memory, one for each of the four variables you need. Once instantiated, it can be passed into `getRed(,green:,blue:)` with the appropriate offset index to set it to the correct value, much like how the pointer was used in the objc example. 

As in the days before Automatic Reference Counting, this pointer must be deallocated when you’re done with it or else memory will leak. Also note, it’s important to call `destroy()` before `dealloc(4)`. `destroy()` returns it to a null pointer, while `dealloc(4)` frees up your memory.

Most of the time you will not have to use `UnsafeMutablePointer` for anything. It exists primarily to ensure compatibility with some of the older Cocoa methods that expect pointers to be passed in. As the name implies, they’re designed to be everything Swift is not: unsafe.
