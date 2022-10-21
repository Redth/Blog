# Would you like some C# in your Cake?

![Cake C# Build DSL](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361396723/NmKlRjnbD.png)  
I will not blog about food. I will not blog about food. I will not blog about food… but I really like cake!

Ok, all cheese (also delicious) aside, Cake is a “cross platform build automation system with a C# DSL to do things like compiling code, copy files/folders, running unit tests, compress files and build NuGet packages.”. Although I think by now it’s plenty more than that as well.

If you’ve never heard of Cake, go to the website now: [http://cakebuild.net/](http://cakebuild.net/)

I’ve been using/contributing/evangelizing cake for a little while now, and I’m going to share some of my reasoning, my experiences, and some things to keep in mind for practicing safe cake consumption.

    var target = Argument ("target", "Default");
    
    Task ("Default").Does (() => {
      Information ("Mmm {0}", "cake!");
    });
    
    RunTarget (target);
    

Why yet another DSL for build tasks?
------------------------------------

There’s make, there’s rake, fake, nake, and probably more by now. In truth, I have nothing against any of these options, they just aren’t quite the right fit for me.

You see, it was getting to the point where Makefiles were getting out of control for some of the projects I was working on. I’m not a make guru by any means, which surely did not help, but I was also lacking a build solution that was more cross-platform friendly.

I started looking at alternative solutions. Rake is a common one but I’m not that comfortable with ruby. Fake was an obvious choice for a .NET developer, and a good way to learn F#, but I’m not sure I (or the other C# developers on my team) wanted another learning curve just to get our actual work done.

I noticed a little effort called Cake. Heavily inspired by Fake, running on true C# code, geared towards the types of projects we work on. It immediately captivated me.

Cross Platform
--------------

I mentioned that cross-platform was important to me. When I first discovered cake, it unfortunately did not run on Mono. While eventually the same Roslyn back-end that Cake uses with the MSCLR should work just fine on mono as well, it meant needing to contribute a bit of code to help cake run under the Mono script host in the meantime.

This is done, it works, and while it may not be perfect, it’s pretty darn close – certainly good enough to be very usable!

There are some other minor issues here and there, but mostly unrelated to cake itself, and more-so other tools not being very portable (I’m looking at you, nuget!).

All things considered, I use cake primarily on OSX running on Mono, and as a great side-effect, it runs on Windows too!

Tasks, Dependencies, etc.
-------------------------

I mentioned cake was heavily inspired by F#, and at its core uses the same concept of tasks and dependency chaining.

In your script, you can define many tasks. You can make them depend on one another so that when you run a specific task it will automatically walk through the dependency chain and run those as well. Here’s a pattern we commonly use:

    var target = Argument ("target", "libs");
    
    Task ("externals")
      .WithCriteria (!FileExists ("./externals/file.zip")
      .Does (() =>
    {
      DownloadFile ("http://site.com/file.zip", "./externals/file.zip");
      Unzip ("./externals/file.zip", "./externals/");
    });
    
    Task ("libs")
      .IsDependentOn ("externals")
      .Does (() =>
    {
      NuGetRestore ("./source/Code.sln");
      DotNetBuild ("./source/Code.sln", c => c.Configuration = "Release");
      CopyFiles ("./**/*.dll", "./output/");
    });
    
    Task ("nuget")
      .IsDependentOn ("libs")
      .Does (() => 
    {
      NuGetPack ("./Code.nuspec");
    });
    
    RunTarget (target);
    

Aliases, helpers, addins
------------------------

The power of cake comes from the wealth of [Aliases](http://cakebuild.net/dsl) that are available to use in your build scripts. There’s simple aliases for MSBuild, NuGet, NUnit, File/Directory operations, Compression, and even HTTP. Things like building a .NET project come down to a few simple commands:

It’s also extremely simple to create your own addins to use with cake. I’ve made several already myself:

*   [Cake.Xamarin](https://github.com/Redth/Cake.Xamarin) – Build Android/iOS projects, Xamarin-Component
*   [Cake.XCode](https://github.com/Redth/Cake.XCode) – XCodeBuild and CocoaPods
*   [Cake.FileHelpers](https://github.com/Redth/Cake.FileHelpers) – Advanced file/regex manipulation
*   [Cake.Yaml](https://github.com/Redth/Cake.Yaml) – YAML Serialization
*   [Cake.Json](https://github.com/Redth/Cake.Json) – JSON Serialization

Addins are included in your script with a simple preprocessor directive:

    #addin "Cake.Xamarin"
    

Addins specified like this will automatically be downloaded from NuGet when the script is pre-processed.

Everyone can have Cake
----------------------

Cake is building momentum. It’s currently available to install via [homebrew](http://brew.sh/) on mac, coming soon to [Chocolatey](https://chocolatey.org/) on windows, already supported with syntax highlighting in [Atom](https://atom.io/), with [omnisharp](https://github.com/OmniSharp/) support coming soon as well.

It’s just C#
------------

Did I mention cake is simply c#? That’s right, nothing really special here, the only non-standard part about it is a couple of the preprocessor directives.

This means that for the most part, you can do anything with it you could do in standard C#.

We’ve leveraged this to make some of our build scripts more object-oriented so that a lot of the script code can be reused within many different projects. In terms of lines of code, we’ve reduced the number of build script lines to probably less than one-third what they were in makefiles!

The other big benefit to our team is that everyone on the team knows C#, so they can all actually understand how our build scripts work.

Learn from my mistakes
----------------------

There are a couple of things I’ve learned about cake that I learned the hard way, so you don’t have to.

1.  **FilePath and DirectoryPath are your friends, do not fight them.**  
    When I first started using Cake I worried too much about file and directory paths and treated them like strings with great paranoia.  What I’ve learned is you need to use FilePath and DirectoryPath as much as you can.  They will keep your code cross platform.  They also have implicit conversions to strings, so you don’t have to explicitly instantiate them to use them, just use the string value.  Trust me on this one, these guys are your friends.
2.  **More files isn’t necessarily better.**  
    We had a tendency to maintain a LOT of makefiles that referenced other makefiles that called targets in yet more makefiles.  Initially I tried reproducing this pattern in cake, and it was not ideal.  Consider adding parameters to invoke certain actions with certain values in a single cake script instead of adding many script files.
3.  **Contribute, and make Addins!**  
    The project leaders of Cake are awesome!  They happily accept quality and sensible contributions and are just generally nice people!

It’s also silly easy to make your own addins, and best of all, with some xml comments and a PR to Cake, you can have your own addin api documentation generated and displayed for you on the [official cake documentation site](http://cakebuild.net/dsl)!

Have your cake, and eat it too!
-------------------------------

Hopefully I’ve enticed you to try some delicious cake! Of course cake isn’t for everyone, some people favour other desserts or foods entirely. But if you’re a C# developer, looking for a familiar build system dsl, I encourage you to try some cake today!