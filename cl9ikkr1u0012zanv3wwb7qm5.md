# What are App Links?

![What are AppLinks?](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361433504/lyy27PG7K.png)  
So there’s this new thing called App Links. It’s backed by Facebook and some other big names, so you know it’s going to gain at least a bit of traction. But the problem is, everyone seems to have a hard time figuring out what they actually are and do. The video on AppLinks.org sort of helps, but it still left me confused, even after watching it a few times.

Here’s my attempt to explain what App Links are, why you would need or want them, and how they work. A word of caution: this post is aimed at mobile app developers, so it may be a bit confusing if you’re not familiar with some of the concepts of mobile app development.

Let’s find a problem to solve!
------------------------------

App Links solves a problem that maybe not everyone has caught up to yet, or even knows is a problem, or will be a problem for them. Right now, if you were building an app and wanted to link to a Book on Amazon from within your app, how would you do it? You’d probably find the link to the book (let’s say `http://amzn.com/B007Q4OVHQ`) and simply open that URL in the web browser on the device.

![Why-Link-To-Websites-Instead-Of-Apps](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361434689/ZeHPZZlMO.jpeg)

Sending your user into the mobile browser to this page is fine, it works, and Amazon even displays it nicely in a Mobile friendly format for you. But is this really the best you can do? What if your user already has the Amazon app installed on their mobile device? They’re already logged in to the app, so the friction of actually buying the book you linked to at this point is much lower than if you had sent them to the website where they may still have to login. Besides, the native app is usually, generally a much better experience. Can’t our users have nice things?

Why are we linking back out to the web? The pieces are all there for a better experience, we just need to put them together!

Technical Hurdles
-----------------

Ok, so I’ve convinced you, instead of opening the mobile web page for the web link (`http://amzn.com/B007Q4OVHQ`) it would be much better to open that content right in the native app on the device. Let’s assume the Amazon iOS app has registered itself to handle the URL scheme `amazon://`. Now, let’s also assume that it’s been designed to accept a URL in the format of something like this:  
`amazon://content/item?id=B007Q4OVHQ`

So for the techies out there, when another app asks iOS to open this URL, since Amazon’s app is registered to handle `amazon://` type URLs, iOS will pass the entire URL into the Amazon app when it is launched, instead of trying to open this URL in the browser. At this point, it’s up to Amazon’s app logic to figure out how this **deep link** is opened inside of the app. Ultimately the app will be programmed to see that the user wants to open the details of an item with an _ID_ of _B007Q4OVHQ_

This all works fine today as is, and you can certainly do it (at least in theory, I have no idea if Amazon’s app actually supports deep linking like this). But, how are you supposed to know all the details of the URL scheme Amazon uses within their own app? They may not publish this information anywhere, and even if they do, once you implement it for Amazon, what happens when you want to do the same for ebay, and the next site, and the site after that?

You can see where I’m going with this. The process would result in an exhaustive amount of code to handle each site on a case by case basis. It would be a nightmare to implement, let alone maintain.

App Links to the rescue!
------------------------

Here’s where the App Links standard shines. The idea simply put is that a web link to content should be able to define how that content may be viewed within native apps, on mobile devices.

So, if Amazon were to add a bit of special HTML to the book’s web page (for [http://amzn.com/B007Q4OVHQ](http://amzn.com/B007Q4OVHQ)), instead of trying to figure out how we should construct an `amazon://` link on iOS for that content, we would simply scan the content of the book’s page to see if Amazon has told us about any other way to link to this same page’s content in mobile apps on other devices.

Dropping down to a more technical level, this content is defined by using special `<meta .. />` tags in the web page’s HTML. For the iOS example we’ve been looking at, Amazon could add something like this to their HTML:

    <meta property="al:ios:url" content="amazon://content/item?id=B007Q4OVHQ" /> 
    

This would describe the exact URL to use for deep linking to the content from this web page in the Amazon iOS app.

One more piece to the puzzle
----------------------------

This is all great, except you may be wondering, how does a mobile device know about this special metadata in the web page’s HTML? Well, it doesn’t, at least not by default.

The other big piece of the puzzle is getting apps to look for this metadata when they are going to navigate to a URL. It’s not hard to do in practice, but this means **every** time you navigate to a URL in your app, you will need to check to see if that URL contains any App Link metadata. I’ll leave the debate about how this is good or bad for another time.

On iOS, you would basically replace:

    UIApplication.SharedApplication.OpenURL("https://amzn.com/B007Q4OVHQ");
    

with this:

    Rivets.AppLinks.Navigator.Navigate(..)
    

What this does is instead of directly navigating to the URL, it loads the URL first, checks for App Link metadata, and if it finds it, navigates to the appropriate app URL instead of the web URL.

App Links and C#
----------------

See what I did there? Obviously `Rivets.AppLinks.Navigator.Navigate(..)` is not part of the iOS SDK.

I’ve been working on a C# implementation for App Links called [Rivets](https://github.com/xamarin/Rivets). If you have read anything about app Links so far, you’ll probably have heard of [Bolts](https://github.com/BoltsFramework/) which is an implementation of App Links for iOS (obj-c) and Android (java) created by Facebook. So, _Rivets_ is the Xamarin Android, Xamarin iOS, Windows Phone (and hopefully Windows Store soon) equivalent of Bolts!

Other Platforms _ahem, Android and Windows Phone_
-------------------------------------------------

The example we looked at was specifically about iOS, but make no mistake, Android and Windows Phone are along for the ride. I won’t go over the entire specification here, you can [visit the documentation](http://applinks.org/documentation/) for that, but basically each platform has its own specific metadata property tags. So on Android you’d have something like:

    <metadata property="al:android:url" content="amazon://content/item?id=B007Q4OVHQ" />
    <metadata property="al:android:package" content="com.amazon" />
    

Windows Phone is included in the standard right now, but sadly Windows Store and Windows Universal are not. I’ve already petitioned to have them supported, though there’s really nothing stopping us from adopting them as part of the standard as long as we are consistent on the keys used (eg: `al:windows_store:url` and `al:windows_universal:url`).

Finally, iOS actually has two other more specific targets: `iphone` and `ipad` (eg: `al:iphone:url`). The idea being that you might have different URL schemes for iPad and iPhone apps.

Fallback Plan
-------------

You might be wondering what happens when you try to navigate to a page which you presume might have App Link metadata, but it does not, or it does not have it for the platform you’re currently on (Windows Phone apps are still not as common for instance).

The standard for App Links has this taken care of. In the App Link metadata, a _**Fallback Url**_ can be specified. This way, if the device can’t find any links pertaining to the given device, it may still be able to find a Fallback URL specified by the App Link metadata. In a worst case scenario, if no Fallback URL is found, you can simply load the original web link as you would have before App Links existed. Simple.

In a nutshell
-------------

To recap (with an illustration below to help), App Links are simply a standard way for describing how the content in a web page can be viewed, or deep linked to within a mobile app. It’s this standard way of describing such information that allows any app to discover how to deep link to content in any other app by knowing only the web URL for the content you’re interested in linking to.

![App-Links-Example-Flow](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361436087/E0Lm84ZOA.png)

App Links are open source, accessible to everyone, and work across devices. Will they change the world? I don’t know. But I think they’re a pretty good way to solve a problem that we may not yet be aware of.

I’d encourage you to start adding App Links navigation to your apps today!