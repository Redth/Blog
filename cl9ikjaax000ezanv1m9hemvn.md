# Cake #load url:

One of the most underrated features in [Cake](https://redth.codes/cake-load-url/cakebuild.net/) is quite possibly the ability to chain multiple scripts together by using the `#load` preprocessor directive. I find this feature really helpful in setting up some commonly used Tasks, functions. I also make heavy use of `#load ../somefile.cake` to centralize and keep consistent my `#addin` and `#tool` declarations and the versions in use for each of those.

It's not uncommon for me to have a _common_ cake file like this:

    // Tools needed by cake addins
    #tool nuget:?package=XamarinComponent&version=1.1.0.49
    #tool nuget:?package=ILRepack&version=2.0.13
    #tool nuget:?package=Cake.MonoApiTools&version=1.0.10
    #tool nuget:?package=Microsoft.DotNet.BuildTools.GenAPI&version=1.0.0-beta-00081
    #tool nuget:?package=NUnit.Runners&version=2.6.4
    
    // Cake Addins
    #addin nuget:?package=Cake.FileHelpers&version=2.0.0
    #addin nuget:?package=Cake.Json&version=2.0.28
    #addin nuget:?package=Cake.Yaml&version=2.0.0
    #addin nuget:?package=Cake.Xamarin&version=2.0.1
    #addin nuget:?package=Cake.XCode&version=3.0.0
    #addin nuget:?package=Cake.Xamarin.Build&version=3.0.3
    #addin nuget:?package=Cake.Compression&version=0.1.4
    #addin nuget:?package=Cake.Android.SdkManager&version=2.0.1
    #addin nuget:?package=Cake.Android.Adb&version=2.0.4
    

You can see we use a lot of Cake addins in our scripts, and in order to keep our CI builds sane, we want to make sure we're pinning very specific versions of each. Without this common file, it's pretty easy for versions of things to get mismatched between cake files, and for things to generally get out of hand quickly, and for CI builds to suddenly stop working when a new version of something is released.

Loading from a URL
------------------

Being able to `#load` arbitrary cake files is great, but as the number of repositories we use cake in grows, the need for a more centralized approach between repositories is becoming increasingly apparent.

One approach is to create a nuget package with the common cake script in it and load the nuget package (and the cake script). This is definitely fine, but it feels a little bit less agile than just fetching a file from an arbitrary url.

Instead, what I really felt I wanted was the ability to do this:

    #load url:http://some.com/file.cake
    

BYO Cake Modules
----------------

It turns out, that Cake is architected really _really_ well, and it's actually pretty trivial to make a custom Cake _Module_ to introduce new functionality such as this.

In this specific case, there's an interface `ILoadDirectiveProvider` that I could implement in my own module. It has the following methods to implement:

*   `bool CanLoad(IScriptAnalyzerContext context, LoadReference reference)`
*   `void Load(IScriptAnalyzerContext context, LoadReference reference)`

The interface is pretty self explanatory. I just needed to check each `LoadReference` passed into `CanLoad` to decide if it was a url that could be loaded or not (this is why you'll see the `url:` scheme required - it makes it super easy to determine the intention of the `#load` directive and whether or not it is a url to be loaded).

My `Load` implementation ultimately just downloads the URL and saves it to the _Urls_ path (which is _./tools/Urls/_ by default), and finally tells the script analyzer context to `Analyze` the downloaded file.

After implementing this, there's some boiler plate code to register your module with Cake:

    [assembly: CakeModule(typeof(UrlLoadDirectiveModule))]
    
    namespace Cake.UrlLoadDirective.Module
    {
    	public sealed class UrlLoadDirectiveModule : ICakeModule
    	{
    		public void Register(ICakeContainerRegistrar registrar)
    		{
    			registrar.RegisterType<UrlLoadDirectiveProvider>().As<ILoadDirectiveProvider>().Singleton();
    		}
    	}
    }
    

Cake.UrlLoadDirective.Module
----------------------------

I must say, it was pretty painless to create this module. It's currently [available on Nuget](https://www.nuget.org/packages/Cake.UrlLoadDirective.Module/), and of course it's all open source - [check it out on GitHub](https://github.com/Redth/Cake.UrlLoadDirective.Module/).

### Installation

Installing a Cake Module is pretty simple. Just add a file like this:  
`./tools/Modules/packages.config`

With the content:

    <?xml version="1.0" encoding="utf-8"?>
    <packages>
    	<package id="Cake.UrlLoadDirective.Module" version="1.0.2" />
    </packages>
    

The module will be automatically downloaded and loaded when you run your script, and you're now all set to start using the `#load url:...` directive!

### Thanks!

Thanks to the Cake team for pointing me in the right direction on this one, and for the fantastic architecture of Cake itself to even make this possible.

Have fun `#load url:http://ing.your/urls.cake`!