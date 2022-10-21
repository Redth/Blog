# Improvements coming to Xamarin Google Play Services

> **Update (June 22, 2015)**  
> It’s been a month since I blogged about upcoming Google Play Services changes, and a lot has happened since then 🙂
> 
> Google released yet another version of Google Play Services (7.5) so we’ve accounted for this update.
> 
> We also spent a lot of time on QA this time around, running the updates against many existing applications and samples that we have, and we are moving to a more automated build system that will allow us to release more quickly going forward.
> 
> Finally, we re-evaluated some breaking changes we had planned to make and decided that although they would be nice to haves, they were simply not worth breaking so many code bases. We’ve reverted some of these naming changes at least for the near future, so incorporating the update in your app should be pretty simple.
> 
> Rest assured, all of the improvements we made that didn’t break existing applications have been kept, so the future still looks great for Google Play Services on Xamarin.Android!

Google Play Services is a rapidly evolving library that Google has been using to combat fragmentation on the platform, and ensure their developers have access to great new API’s without having to worry about what Android version their users are actually on.

The rate at which this library has evolved recently has been especially challenging for us to keep pace with in getting bindings out for Xamarin.Android, but, rest assured, we are committed to bringing out a great Google Play Services experience in your Xamarin apps. Here are some of the improvements we’ve been making that are now available as pre-release!

Latest and Greatest
-------------------

Google has rapidly iterated in the last few revisions. We went from 6.5 to 7.0 to 7.3 very quickly, and we’re happy to say we’re shipping the latest and greatest and expect to keep a better track record going forward on getting the latest out quickly!

**NOTE:** As of this post, the improvements are available on NuGet as _**24.0.0-rc1**_ versioned pacakges.

NuGet what you Want
-------------------

We’ve now split up Google Play Services into many NuGet packages to match the changes Google made to split them up into many .jar files. Using NuGet we’re able to manage the dependencies between all of the packages. For example, all packages depend on the _Google Play Services – Base_ package, while others such as _Location_ depend on _Maps_ as well.

You can now grab only the packages you need in your app without bringing in everything else. This should help keep your app size down as Google continues to expand Play Services adding more and more features and API’s.

Finally, the old _Google Play Services – All_ package has been kept around to help make the transition easier. It now depends on all of the individual NuGet packages.

Breaking Changes, Nicer Naming
------------------------------

In the process of binding libraries to make them more C# friendly, we often run into cases where casing is different between java and C#. For example, in java, we have `cast.Cast` which in C# translates into  
`Cast.Cast`. Since we cannot have a class named the same as its namespace in C#, this presents a problem. Traditionally we have renamed the class, for example to `Cast.CastClass`. This isn’t always very discoverable or nice to look at.

> **Update:** We will no longer be renaming these namespaces and classes. No breaking changes here anymore!

We’ve now decided to change the namespaces instead of renaming the conflicting classes. All namespaces now have _Sdk_ appended to them, so we now have `CastSdk.Cast` for instance. This should be much more consistent and discoverable going foward.

The other area where this is a problem is with fields in java. For instance, `Fitness.FITNESS_API` and `Fitness.FitnessApi` in java both transform into `Fitness.FitnessApi` in a C# binding by default. To avoid conflicts and bring consistency across the bindings, we’ve decided to keep the java naming conventions in the C# binding.

You’ll now use the upper-cased `Fitness.FITNESS_API` fields to specify an Api to add in your `IGoogleApiClient.AddApi (..)` calls, whereas you’ll use the lower-cased `Fitness.FitnessApi` fields to actually invoke methods of the given Api.

> **Update:** We’ve decided to keep, but deprecate the old `Api` properties to not break your existing code, but the new names will be included still!\*\*

Happier C# Developers
---------------------

Our binding tooling generally does a good job at making java libraries more C#-ified. Google Play Services has a number of cases that are less friendly to our tooling and often this has created a sub-par developer experience from a C# perspective.

We’re continuing to work at improving this story in Google Play Services, and there are a couple of changes that should make life better:

### IPendingResult.AsAsync

A very common pattern in Google Play Services is to make use of the interface `IPendingResult`. This is sort of like a `Task` – it’s a promise or a future. You can `.Await()` it in a blocking manner (though you must do so off the UI thread), or you can `.SetResultCallback (IResultCallback)` which means you need to pass in an implementation of the interface which usually means creating an implementor class with some Actions or Events to make the code a bit less ugly (since we don’t have anonymous classes in C#).

You now have more options with `IPendingIntent` instances through some extension methods:

    // 1. Turn it into a Task with the desired result type var result = await SomeMethodReturningPendingIntent (...)   
      .AsAsync<TypeOfStatus> () 
    
    // 2. Pass in an Action<T> to the callback SomeMethodReturningPendingIntent (...) 
      .SetResultCallback<TypeOfStatus> (result => { // ... });
    

### Helper Implementor Classes

There’s still a lot of areas that require you pass in some sort of class implementing a listener type of interface. `ILocationListener` is a good example. Instead of having to build your own class which implements this simple interface, we’ve added a class `LocationListener` which implements `ILocationListener` and exposes an event for you. Nothing earth-shattering here, but hopefully it saves you from writing the same boiler plate code over and over.

You can expect more implementation classes like this going forward.

### GoogleApiClientBuilder

The first place you’ll probably notice the heavy use of interfaces is in the `GoogleApiClientBuilder`. The  
`.AddConnectionCallbacks (..)` and `.AddOnConnectionFailedListener (..)` methods both require instances of interface implementations as parameters. You can still call these methods like normal, but now there’s extension methods allowing you to pass Action’s into them as appropriate. Again, this is code you could easily write, but shouldn’t have to.

### Less p0, p1, p2

This has always been a bit of an eye-sore, to see poorly named parameters. We’ve made some improvements in our tooling that are going to continue to help get better parameter names. We’ve started trying some of these techniques out already in Google Play Services!

Feedback?
---------

I don’t always get to explore all of the API’s inside Google Play Services on a day to day basis, so I’m not always aware of where the pain points are.

I’m very open to feedback if there’s other areas you’d like to see some C# improvements to like we’ve started.

Please don’t be shy, mention me on twitter with any suggestions you have!