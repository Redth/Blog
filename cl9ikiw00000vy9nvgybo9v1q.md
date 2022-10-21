# Shared Images for Xamarin with Resizetizer NT

Images in Xamarin, and mobile apps in general have always been a pain point for me. For most apps, this means for every image you want to use, you'll need to resize it 5 times for each android display density, 3 times for iOS, and 3 times for UWP. That's 11 different images for every source image!

Years ago I created a silly little thing called [Resizetizer](https://github.com/redth/resizetizer). This provided a way to create the multitude of resolutions required for each platform automatically, based on a source image, and a YAML configuration file. I implemented this as an MSBuild Task that would run in a project which you specified a YAML file for. After the images were output in various sizes, you'd still have to go into your individual Android, iOS, and UWP projects and include the generated images. This was ok, and a reasonable time saver, but it was far from perfect...

Meet the New Technology
-----------------------

A few weeks ago I decided it was time to resurrect the project and fulfill my ultimate vision for Resizetizer. Since it's such a departure from how the original one worked, and given the original one has at least a couple other users, I thought it best to create a brand new package.

[Resizetizer NT](https://github.com/Redth/ResizetizerNT) (New Technology 🤭) allows you to add your source image assets into your shared code layer project, and have them automatically resized and included in each platform's app head project.

For those who like boring videos, this one will not disappoint as it undelightfully displays how to use Resizetizer in your apps:

<iframe width="560" height="315" src="https://www.youtube.com/embed/O7a7HKiFypw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

Or if you prefer the text version, keep reading!

### Including Images in your Shared Project

First of all, you'll need to install the `Resizetizer.NT` [NuGet package](https://www.nuget.org/packages/Resizetizer.NT/0.1.0-preview7) into the project with your images, and all the app head (so your Xamarin.iOS, Xamarin.Android, and UWP) projects in your solution.

Next, you add images. It's really easy. Just drag your image into your project, and set the item's build action to be `SharedImage`.

Finally, open your .csproj file and add a `BaseSize` attribute to the `SharedImage` item:

    <ItemGroup>
    	<SharedImage Include="xamagon.svg" BaseSize="60,60" />
    </ItemGroup>
    

### Base-what now?!

Ok, so the thing is, every platform has a concept of a normal, nominal, original, base, 1x, whatever-you-want-to-call-it resolution. On Android this is `drawable-mdpi\xamagon.png`, on iOS it's `xamagon@1x.png` (without the `@1x` part), and on UWP it's `xamagon.scale-100.png`.

In other words, this is the display density independent size of the image, or the width and height you will likely specify in your code. If you were using Xamarin.Forms and your `BaseSize="60,60"`, your XAML code would look like this:

    <Image Source="xamarin.png" WidthRequest="60" HeightRequest="60" />
    

Since vectors are by definition infinitely scalable, they don't really have a 'size'. Setting the `BaseSize` attribute on your `SharedImage` item allows Resizetizer to infer the size you intend to use it in code, and appropriately resize the image for all of the other resolutions of each platform.

### What about non-vector sources?

Yeah you can use non-vector sources (or at least PNG's) as well, but you still need to specify the `BaseSize`, otherwise Resizetizer will assume the actual width/height is the base size and you'll get some pretty ugly pixelated messes for higher density display versions of your image.

How does it work?
-----------------

The wizard behind the curtain may be rather disappointing, but I'll show them to you anyway.

The magic is all in MSBuild targets. When you install the nuget package into your app projects, it hooks itself into the build process specifically for each platform.

First of all, a target is invoked on all projects which the app references, if it exists. This target gets a list of `SharedImage` items from the referenced projects so it knows what images to process. Once it has the list it begins resizing the source image to all the correct resolutions for the platform the app project is targeting. This list of resolutions and their output naming conventions is opinionated and predefined for each platform. The lovely [Skia.Svg](https://github.com/wieslawsoltes/Svg.Skia) and [SkiaSharp](https://github.com/mono/SkiaSharp) are used to load up the vectors and/or bitmaps, resize them, and render them out to the appropriate location in the app's intermediate output directory.

Finally, for each platform, the resized output images are included as the appropriate Item type so that the app project's build sees them just like any other resource you'd manually reference and includes them in the resulting app.

Voila! Not all unicorns and fairy dust after all!

Great, what else?
-----------------

This is a good start, and makes dealing with images much more productive, but there's more juice to squeeze out of these oranges! Eventually I'd like to add some more features:

*   Allow for a `TintColor` attribute on `SharedImage` items which can set the fill color on SVG paths
*   Create a special type of `SharedImage` for app icons and output the appropriate asset catalog / mipmap resources / sized UWP icons based on one source
*   Add support for generating launch/splash screens which would generate a Storyboard / Activity Theme / Launch settings for native apps
*   Sometimes vectors are preferable still, in which case we could have a way to convert .svg to Android Drawable Vectors and maybe PDF's for iOS and ??? for UWP

There's probably even more opportunity here for features.

Anyway, what do you think?

Check out the [NuGet Package](https://www.nuget.org/packages/Resizetizer.NT/0.1.0-preview7) and the [Source Code](https://github.com/Redth/ResizetizerNT).

Also, thanks to [@jonathanpeppers](https://twitter.com/JonathanPeppers), Dean Ellis, and [@mattleibow](https://twitter.com/mattleibow) for letting them pick their brains on things, and [@wieslawsoltes](https://twitter.com/wieslawsoltes) for creating [Svg.Skia](https://github.com/wieslawsoltes/Svg.Skia)!