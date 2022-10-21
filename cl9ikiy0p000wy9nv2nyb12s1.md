# How to Fix the dreaded `Version conflict;` NuGet error in your Xamarin.Android projects

It's a sunny day, your coffee is still hot, and you're ready to take on the world. You go to install a NuGet package into your Xamarin project, and that's when your day starts to fall apart:

```
Package restore failed.
```
    

This is not what you wanted to see.

```
NU1107: Version conflict detected for Xamarin.Android.Support.Compat. Reference the package directly from the project to resolve this issue. 
     MyApp -> Xamarin.Essentials 0.7.0.17-preview -> Xamarin.Android.Support.CustomTabs 27.0.2 -> Xamarin.Android.Support.Compat (= 27.0.2) 
     MyApp -> Xamarin.Forms 3.1.0.583944 -> Xamarin.Android.Support.v4 25.4.0.2 -> Xamarin.Android.Support.Compat (= 25.4.0.2).
```
    

All you wanted was to install a new package, or update an existing package, and your day just got more difficult.

## Wait, we can fix this!

This is something that a lot of users encounter, especially when dealing with Xamarin.Android projects as they tend to all require Android Support libraries, which necessarily have a very complex dependency chain.

While this can be a daunting error to run into, if we take another sip of coffee and truly look at the error message, it becomes a bit more clear the action we need to take.

Let's pick this error apart. What it's telling us is:

*   `Xamarin.Essentials` requires some `Xamarin.Android.Support` bits `>= 27.0.2`
*   `Xamarin.Forms` requires some `Xamarin.Android.Support` bits `>= 25.4.0.2`

It's important to remember that a requirement of Android Support libraries is that they must _all_ have the same version number in your project.

What NuGet is trying to tell us is that it can't figure out which version of `Xamarin.Android.Support.Compat` you want it to install, since your dependencies can't agree on what that version should be.

The solution is to help NuGet out a bit, by explicitly declaring the version it should use, in this case we can more easily deduce that since Xamarin.Forms is fine with Android Support _greater than_ `25.4.0.2` and since Xamarin.Essentials requires _at least_ `27.0.2`, we should pick `27.0.2` to install:

```
<PackageReference Include="Xamarin.Android.Support.Compat" Version="27.0.2" />
```
    

Now, you may run into this same type of issue multiple times in your project. In that case, just keep adding explicitly versioned dependency declarations to your project as NuGet asks you to help it decide.


I've also made a video to illustrate the process, which you can view below:

<iframe frameborder="0" scrolling="no" marginheight="0" marginwidth="0" width="788.54" height="443" type="text/html" src="https://www.youtube.com/embed/1GUqg2DNAF0?autoplay=0&amp;fs=0&amp;iv_load_policy=3&amp;showinfo=0&amp;rel=0&amp;cc_load_policy=0&amp;start=0&amp;end=0"></iframe>