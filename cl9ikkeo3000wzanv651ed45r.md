# Unifying Google Play Services

Once upon a time, you may have noticed there were a lot of choices when it came to adding Google Play Services to your Xamarin.Android apps.

[![Google Play Services Chaos](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361419341/-C-YAzklZ.png)](http://redth.codes/wp-content/uploads/2015/10/xamarin-google-play-services-components.png)

This was always a bit confusing. Which version should you use for your app? Should you use the version for the highest API level you’re targeting, or the lowest one? The answer wasn’t always clear.

I won’t go into great detail as to why we did this in the past, because the good news is, there have been improvements in more recent versions of Xamarin.Android which have allowed us to unify many of these packages into one!

Starting today, we have simplified Google Play Services bindings into the two following packages (both for NuGet and Component Store):

*   Google Play Services (Froyo)
*   Google Play Services

This mirrors exactly what Google releases to developers: a .jar file frozen in time for Froyo compatibility, and the new, frequently updated .jar file for Gingerbread and above.

Which one do I use?
-------------------

As a general rule, you should almost always use Google Play Services, unless you have a special need to target Froyo, however given that Froyo is now only 0.7% of devices, you are most likely safe to drop support for it.

_There is one caveat:_ If you are targeting Gingerbread, there will be some classes in Google Play Services that you aren’t able to use while running on Gingerbread (eg: [MapFragment](http://developer.android.com/reference/com/google/android/gms/maps/MapFragment.html)). You should take care to avoid using these if you are running on an API Level they aren’t compatible with ([Google has documented this](http://developer.android.com/reference/com/google/android/gms/maps/MapFragment.html)). Luckily Google also provides `Support*` versions of these classes (eg: [SupportMapFragment](http://developer.android.com/reference/com/google/android/gms/maps/SupportMapFragment.html)).

Updated & Disabled/Delisted Components/NuGets
---------------------------------------------

Finally, if you have referenced the _Google Play Services (ICS)_ Component or NuGet, you should update it (it will now appear as simply _Google Play Services_). This also means we have disabled and delisted the following Components and NuGet packages:

*   Google Play Services (Gingerbread)
*   Google Play Services (JellyBean)
*   Google Play Services (KitKat)

If you were using any of those packages, now is a good time to update them to use the new packages!