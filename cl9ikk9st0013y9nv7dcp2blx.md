# Smarter Xamarin Android Support v4 / v13 Packages

If you’ve ever built an Android app, you’ve probably used the Android Support libraries. Google created these as a way to enable developers to use new features on older Android versions. The most common Support library is arguably `Support-v4`, however as older Android devices retire and newer versions of Android increase in market share, `Support-v13` is becoming more commonplace.

_**Warning:**_ This is going to be a bit geeky of a post. If you don’t want to know the details, skip down to the **TL;DR** section for the need-to-know bits 🙂

> `Support-v4` can be used all the way back to Android API Level 4 (Donut – 1.6), but `Support-v13` is only compatible back to API Level 13 (Honeycomb – 3.2).

The challenge with these two support libraries is that they are mutually exclusive. That is, you can’t have `Support-v4` and `Support-v13` referenced in the same app.

There’s a good reason for this though: `Support-v13` actually contains all the code and classes that you’d find in `Support-v4`, it just adds a bit more on top of them. So, the answer is, just use `Support-v13` if you need to instead, right? _If only it were that simple._

Trouble in Paradise
-------------------

Since `Support-v4` is so prevalent in apps and provides so much to developers, a lot of third party libraries themselves reference it. _Facebook_, and even _Google Play Services_ both depend on `Support-v4`, for example.

> Usually 3rd party libraries choose to depend on `Support-v4` and not `Support-v13` so that they can be used in apps with older API levels.

In the Xamarin world, these libraries (or bindings) are compiled to expect a reference to `Xamarin.Support.v4.dll`, and if you try and add `Xamarin.Support.v13` to your project, things blow up (since `Support-v13` already contains `Support-v4`). This has proven to be painful for developers wanting to use the new `Support-v13` features, but also any 3rd party libraries which reference `Support-v4`.

TypeForwardedTo the rescue!
---------------------------

Luckily, fellow Components Team member [Matthew](https://twitter.com/mattleibow) thought up a crafty solution to this issue. There’s a somewhat little known `Assembly` level attribute called [TypeForwardedTo](http://goo.gl/Whj7Cm). This nifty attribute allows you to forward an expected type declaration in a given assembly to an actual implementation of it which exists in another assembly.

So, what we’ve done is created a `Xamarin.Android.Support.v4.dll` assembly which contains nothing but `[assembly: TypeForwardedTo (..)]` declarations to forward all of the expected `Support-v4` types to `Xamarin.Android.Support.v13.dll` instead.

This means if you’re using a library which requires a reference to `Xamarin.Android.Support.v4.dll` (Let’s say Google Play Services for instance), you can swap in this ‘type-forwarded’ assembly, and then add a reference to `Xamarin.Android.Support.v13.dll` as well. Since _**v13**_ contains all the implementation which _**v4**_ has just forwarded to it (and then some), you can use the new _**v13**_ fanciness in your app, while still satisfying the _**v4**_ reference that 3rd party libraries are depending on!

Sprinkle in some magic NuGet dust…
----------------------------------

This wouldn’t be nearly as fun if you had to worry about which `Xamarin.Android.Support.v4.dll` you needed to pick (the actual implementation, or the type-forwarded one). Luckily, in addition to the `TypeForwardedTo` magic, we’re adding some NuGet pixie dust to the mix!

If you’re interested in the mechanics, this is essentially the new .nuspec file we’re using (You can skip ahead to the TL;DR section below otherwise):

    <?xml version="1.0"?>
    <package>
      <metadata>
        <id>Xamarin.Android.Support.v4</id>
        <title>Xamarin Support Library v4</title> 
        <version>20.0.0.4</version>
        <!-- Ommitted boring parts -->
        <dependencies>
            <!-- Depend on v13 support nuget if targeting equal or higher than Android 3.2 --> 
          <group targetFramework="MonoAndroid32">
            <dependency id="Xamarin.Android.Support.v13" version="20.0.0.4" />
          </group>
        </dependencies>
      </metadata>
      <files>
        <!-- Use Support v4 lib for anything up to Android 3.2 -->     <file src="V4Xamarin.Android.Support.v4.dll" target="libMonoAndroid10" />
        <!-- Use Support v4 typeforwarded (to v13 types) lib for anything equal or higher than Android 3.2 -->
        <file src="V4TypeForwardedXamarin.Android.Support.v4.dll" target="libMonoAndroid32" />
      </files>
    </package>
    

TL;DR – The need to know bits
-----------------------------

We’ve made some minor, but important changes to the `Xamarin.Android.Support.v4` NuGet package:

1.  If your app is targeting Android API Level 13 or higher (Honeycomb 3.2), you will automatically get the `Xamarin.Android.Support.v13` NuGet package as a dependency.
2.  If your app is targeting Android API Level 13 or higher, you will also get the _**Type-Forwarded**_ assembly version of `Xamarin.Android.Support.v4.dll` installed by the NuGet package. This will work in conjunction with and forward all of its types to the `Xamarin.Android.Support.v13.dll` assembly that gets installed by the _**v13**_ NuGet dependency.
3.  If your app is targeting anything _**below**_ API Level 13 (eg: Froyo, Gingerbread), you will get the good old `Xamarin.Android.Support.v4.dll` assembly that you’ve come to know and love!

To be clear, yes, we have decided that if you are targeting API Level 13 or higher, and choose to use the Android Support library _**v4**_, you will actually be using _**v13**_ instead. We decided the very small size increase was well worth the much improved compatibility between your apps and 3rd party libraries.

You can use these new packages today! We will be rolling out updates to our Components to achieve the same results in the near future!

Enjoy!