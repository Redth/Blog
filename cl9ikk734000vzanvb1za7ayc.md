# New and Improved Xamarin Studio Launcher

**UPDATE – Nov 29, 2016:** By popular demand, I’ve created an update which is compatible with both Xamarin Studio and Visual Studio for Mac.  It’s now called MS Solution Launcher and can be found on GitHub (source code and binary release).

Go Check out [MS Solution Launcher on GitHub](https://github.com/Redth/MSSolutionLauncher)!

**UPDATE – Sep 27, 2016:**  There’s a new version that I had forgotten to share from some time back which works with the newer versions of Xamarin Studio.  Enjoy the updated download link at the bottom of the post 🙂

**UPDATE – Jan 12, 2015:** Thanks to the wonderful [@Vaclav](http://twitter.com/vancura), Xamarin Studio Launcher now has a proper icon! Please download the much prettier v4 at the end of this post!

Awhile back I made a quick little AppleScript based app called [Xamarin Studio Launcher](http://redth.info/Xamarin-Studio-Launcher/) to help launch multiple instances of Xamarin Studio (it had a pretty little icon that you could keep in your dock to open new instances – which I recently updated to the newer Xamarin look).

[![Xamarin Studio Launcher](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361408797/cGN19RcPp.png)](http://res.cloudinary.com/redth-codes/image/upload/v1508287323/Xamarin.Studio.Launcher.Icon__norfvy.png)

While it was cute, and relatively functional, as a recovering ex-Windows user, I found myself still constantly opening `.sln` files from finder, which would cause them to open in an existing instance of Xamarin Studio, closing whatever current solution happened to be open in that instance.

Now I know about the ability to open multiple solutions in the same Xamarin Studio instance, and generally I’m pretty good about forcing myself into learning the nuances of the platform I’m working on, however, having multiple instances of Visual Studio open was something I grew so accustomed to that I just couldn’t shake the habit of being on a Mac!

Not sure why it took me so long to make this, but here it is, finally!

Xamarin Studio Launcher v3
--------------------------

This new launcher app works first and foremost exactly like the [previous Xamarin Studio Launcher](https://redth.codes/Xamarin-Studio-Launcher/) I released. You can put it in your dock, and when you open it, it will launch a new, blank instance of Xamarin Studio.

_The new feature_ is that it can now handle opening `.sln` files. If you choose to open a `.sln` file with this app, it will open that `.sln` file in a _new_ instance of Xamarin Studio.

This means you can set Finder to open all `.sln` files with Xamarin Studio Launcher so any time you double click or otherwise open a `.sln` file from Finder, it will open in its own instance of Xamarin Studio!

**How to set this as the default app for .sln files**

![Finder Change All](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361409984/-cAgCry6Y.png)

1.  Find a `.sln` file in Finder
2.  Right click the `.sln` file and `Get Info` (or highlight the file and `cmd + i`
3.  Under the _Open With_ section, click the drop-down list and click _Choose_
4.  Navigate to and select _Xamarin Studio Launcher_
5.  Click _Change All_

**Download**

Here’s the .zip file containing _Xamarin Studio Launcher.app_:  
[Download Xamarin Studio Launcher](http://redth.info/assets/Xamarin.Studio.Launcher.v5.zip) _(Updated Sep 27, 2016)_