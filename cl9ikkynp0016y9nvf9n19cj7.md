# Such Android API levels, much confuse. Wow.

![Such API Levels, Much Android, So confuse](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361441936/BAtW7iix5.png)  
I have been bitten more than once by the confusion of Android’s many API levels when building Xamarin.Android apps. It gets even more complex when you start referencing other libraries that target different API levels. Just the other day I had an [issue come up with AndHUD](https://github.com/Redth/AndHUD/issues/9) where the resolution for a runtime error had nothing to do with the code itself. I decided it was time to finally write this post.

As if it weren’t confusing enough, Google generally gives three names to every API level: an integer API Level number, a version number, and a delicious sounding dessert name.

For example, Gingerbread was version 2.3 or API Level 9 (also Gingerbread had 2.3.7 and API level 10 as an update). Kit Kat is version 4.4, API level 19. Luckily [Google has published](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html) the various versions in a table so you can figure out what is what.

When it comes to your Xamarin.Android apps, it’s important to be aware of all of the spots you can set API levels in your project settings. In particular, there are three places of interest:

1.  Project Settings -> General -> Target Framework
2.  Project Settings -> Android Application -> Minimum Android Version
3.  Project Settings -> Android Application -> Target Android Version

![Android-API-Levels-Project-Settings](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361443443/9OHsIZ8O_.png)

1\. Target Framework
--------------------

This setting is arguably the most important one. The Target Framework basically tells the compiler what API’s are available to compile your app with, at _compile time_. This setting really has no bearing on what API’s are actually available to your app to use on a device at runtime, but rather which API’s are _potentially_ available.

In AndHUD, I wanted to be able to use `Android.Animation.ValueAnimator` (which is only available in API Level 11 and higher). If I had written code using that class and set my Target Framework to anything below API Level 11, I would have seen a compiler error saying the class `ValueAnimator` could not be found. Since I set the Target Framework to API Level 11, my code compiled with no issues.

Generally, you will want to set your Target Framework to be at least as high as the highest API Level required by any code in your app.

2\. Minimum Android Version
---------------------------

This one’s pretty simple, it’s the lowest API level that your app will allow itself to be installed and run on.

Why would you want this to be different then the Target Framework? Well, in AndHUD, I wanted to make the progress circle animate when it changed. Since the `ValueAnimator` class was only available in API Level 11 or higher, I had to make a decision: Did I want to only support devices running API Level 11 or higher, or did I want to downgrade the user experience for older devices that didn’t support this animation?

I chose to support devices running older API levels. Since my Target Framework was set to API Level 11, I needed to set the _Minimum Android Version_ to _API Level 9 (2.3 Gingerbread)_, so the app would be allowed to still run on API Levels 9 and 10.

What this means, is that you can compile your app to use any API’s available up until the Target Framework, but your app will still run on devices running with API levels as low as your Minimum Android Version.

This doesn’t magically allow you to use unsupported API’s at runtime on devices that do not support them. This simply allows you to still run your app on a device that _might_ not support some of the API’s your compiled app _may_ be using. At this point it’s up to you to ensure at runtime that you don’t use any API calls which aren’t supported on the version of Android the device is currently running.

Downgrading Experience at runtime
---------------------------------

If your _Minimum Android Version_ is lower than your _Target Framework_, there are probably some devices that your app will run on which do not support all the API calls you’ve used in your app. In this situation you need to make decisions about how to downgrade your user experience when API’s are not supported on a device your app is running on. This may be cosmetic, or it could mean making some features or functionality available only to devices with newer versions of Android.

Since I decided to allow AndHUD to work on devices with API levels lower than 11, yet I used `ValueAnimator` which isn’t available on those lower API Levels, I needed to make sure that at runtime, I didn’t try to use the `ValueAnimator` class when it was not available.

To detect your API Level at runtime, you can access the `Android.OS.Build.Version.SdkInt` property. There are a number of constants to check this property against such as `Android.OS.BuildVersionCodes.Honeycomb`. Here’s how I dealt with downgrading the user experience in AndHUD:

    // Get the version of Android we're currently running under
    var version = Android.OS.Build.VERSION.SdkInt;
    
    // Check to see if we're >= API Level 11 (HoneyComb)
    if (version >= Android.OS.BuildVersionCodes.Honeycomb) { 
      // It's safe to use ValueAnimator
      var va = (Android.Animation.ValueAnimator)Android.Animation.ValueAnimator.OfInt (progress, newProgress)
        .SetDuration (250);
      va.Update += (sender, e) => {
        var interimValue = (int)e.Animation.AnimatedValue;
        progress = interimValue;
        Invalidate ()
      };
      va.Start ();
    } else {
      // Not safe to use ValueAnimator so let's just change the progress without an animation
      progress = newProgress; Invalidate ();
    }
    

You can see the difference in the animation below. To the left, we’re running **<** API Level 11 without any animation to the progress circle as it advances. To the right, we’re running on **\>=** API Level 11 with smooth animating of the progress circle.

![AndHUD-Animation](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361444897/gtZ6aDwaN.gif)

How you downgrade user experience on older API Levels at runtime is going to be different depending on what API’s you’re actually using. There is no simple answer for this, you just need to use your best judgement.

3\. Target Android Version
--------------------------

I left this one until the end for a reason. It’s probably not something you need to change unless you know exactly why you need to change it. My recommendation here is to leave this one set to _Automatic – use target framework version_ which will automatically target the same version you picked for **#1 Target Framework**.

There are some edge cases you might need to set this property different, but I haven’t run across any myself yet.

**UPDATE:** There is one fairly popular edge case that has come to light since I originally wrote this post. You might want to use a higher Target Android Version than your Target Framework to satisfy **aapt.exe**’s compilation.

For example, with the AppCompat v7 bindings that Xamarin provides, Google has used some resource attributes that only exist in Lollipop (v5.0, API Level 21). You could set your Target Framework to v5.0 and everything will compile, but you might not want to have to worry about not using that many new API’s in your app. It is possible to set your Target Framework to v4.0.3 but set your Target Android Version to v5.0 to satisfy **aapt**.

But wait, there’s more (complexity)
-----------------------------------

Now that you understand the difference between Target and minimum Android version settings there’s one more curve ball to throw at you.

You may have noticed already that in Android Library projects, there’s only one place where you can change the Android version, and that’s **#1 Target Framework**. This makes sense, since a library cannot run on its own, and should have no opinion of which minimum Android version an app that uses the library should target (that would be rude).

In the case of AndHUD, the actual AndHUD library is set to Target Framework API Level 11. It’s generally good practice to make your Android library projects Target the minimum API level it needs to compile.

For library projects, this doesn’t mean you’re off the hook for checking Android versions at runtime, in fact, in a library that others are consuming, you should be extra careful about which API calls you use and whether or not they are supported at runtime. Try to be a considerate library developer: _**don’t crash other peoples’ apps!**_

It’s also _**very important**_ to note that if you’re consuming an Android library in your Android application, your application’s Target framework must be set to as high, or higher than the Target Framework in the library project, or you will no doubt, run into errors just like in the AndHUD issue I referred to at the beginning of this post.

Hopefully that sheds a bit more light into how Android versions (API Levels) relate to your Xamarin.Android libraries and applications.

Good luck, and stay on the level!