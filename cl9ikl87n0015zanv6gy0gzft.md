# iOS7: Fun times with the new Full Screen Layout!

In a mad rush, I had to prepare an app for iOS7 _after_ its release. Yes, I know, shame on me for not listening to everyone telling me to start getting my apps ready early. How bad could it be anyway?

For the most part, not too bad. The worst part for me came in the shape of dealing with this new idea that every `UIViewController` now expands the bounds of the entire screen, including where the status bar and navigation bars could be. In any case, I had to do a bit of learning really quickly. If you don’t already know what I’m talking about, here’s a picture of one unpleasant conversion experience:

![http://i.imgur.com/wHK3KSO.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361457178/orvGWMsYI.png)

At this point, many thoughts went through my head. I could just change the table view’s Frame right? Well then I would have to check if it’s iOS7 or not, and adjust according to iOS version, and and and… that just doesn’t seem right. Why would you do this to me, Apple?! At this point I took a deep breath, relaxed, and did what any competent developer would do: I Googled it.

This of course led me to StackOverflow, and several people asking the same thing. [Why is my navigation bar and status bar appearing over my view?](http://stackoverflow.com/questions/17074365/status-bar-and-navigation-bar-appear-over-my-views-bounds-in-ios-7). There’s some pretty good answers there no doubt.

Everything new is old again!
----------------------------

Of course, the easiest way is just: make it work like it used to! This is entirely possible! All it requires really, is setting your view controller’s `EdgesForExtendedLayout` property to `UIRectEdge.None`. This was a quick fix, and ended up getting me up and running again until I could figure things out (you can of course choose some, all, or none of the edges to be used for extended layout with this property – for example, you might want the top to blur under the navigation bar, but the bottom to not blur underneath the tabs).

This was a workable solution, but I still wasn’t really happy with it. After all, the cool kids were all using the fancy blurred view scrolling behind the navigation and status bars like in the image below, and, who doesn’t want shiny and new?

![blurred](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361458292/gB1S-UTjM.png)

Newfangledness $#&@\*
---------------------

So, what Apple really intends is to get a nice blurring effect of the contents of a scrollable view that happen to be _under_ navigation bars, status bars, tab bars, etc. It’s really quite nice, if you haven’t seen it in action for yourself yet.

To make this happen, we essentially need to project our view to the bounds of the entire screen. The problem with this of course, is that we don’t want the top parts of our scrollable view to be initially hidden behind the navigation and status bars.

Content Insets to the rescue! (grab a coffee first)
---------------------------------------------------

If our view fills up the entire bounds of the screen, yet there’s still a status bar and navigation bar to show over top of it, we need a way to make sure the contents of our view aren’t initially hidden by the navigation and status bars.

For this purpose, we can take advantage of the `ContentInset` property that exists on any kind of scrollable view (eg: `UITableView`, `UIWebView`, `UIScrollView`, etc). This little gem of a property allows us to specify where our content should initially start, offset from the edges of the view that is holding it.

It’s important to note that Apple tries to help us out here. By default the `UIViewController` property `AutomaticallyAdjustsScrollViewInsets` is set to `true`, which means that in many cases, this will just work, and you won’t have to think about it. However, in practice you might run into some edge cases (haha, get it?) where the inset cannot automatically be inferred correctly, like I did.

In one case, my `UITableView` was not automatically adjusting the inset correctly. Long story short, I had to account for a status bar (**20pt**), and a navigation bar (**44pt**) for a total of (**64pt**) inset from the top edge of the screen. That meant I needed to set my `UITableView.ContentInset = new UIEdgeInsets (64, 0, 0, 0);`. The result was my UITableView still taking up the entire screen, but not making the top row initially hidden behind the navigation bar, while still taking advantage of the nice blurred scrolling underneath of it.

More dynamic?
-------------

You may have scoffed at my example of hard-coding the inset. All I can say is that sometimes you just need it to work immediately. If you’d like to do this more properly, you should consider the other new properties Apple has included on the `UIViewController` which are: `TopLayoutGuide` and `BottomLayoutGuide`. These serve as a reference to us to let us know what the _top_ (and _bottom_) of our content should be within the bounds of the screen (by using the `.Length` property of Top/Bottom layout guides).

Now, there is one catch. As of this article being published, the `.Length` property was missing from the Xamarin API. There’s already a bug in for a fix, and in the meantime there’s a work around:

    var length = MonoTouch.ObjCRuntime.Messaging.float_objc_msgSend (TopLayoutGuide.Handle, (new Selector("length")).Handle);
    

Oops, there’s another catch. This property doesn’t seem to return > 0 until the views have all been laid out. So, you may consider doing something like this:

    public override void ViewDidLayoutSubviews () {
      base.ViewDidLayoutSubviews ();
      var length = MonoTouch.ObjCRuntime.Messaging.float_objc_msgSend ( 
        TopLayoutGuide.Handle, (new Selector("length")).Handle); 
      TableView.ContentInset = new UIEdgeInsets (length, 0, 0, 0);
    }
    

Also, don’t get caught with your pants down!
--------------------------------------------

If you haven’t already guessed it, some of these properties are iOS7 only. So if you’re targeting iOS6 still, you’ll have to do some sort of version checking to make sure you only use these API’s when the app is running under iOS7!

Anyway, I hope this finds its way to someone else who is going through the same learning pain as I did, to get their app ready for iOS7!