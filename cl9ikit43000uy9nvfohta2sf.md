# Using settings.json and local.settings.json in your Xamarin apps!

Sometimes it's really useful to be able to change things like endpoint urls, API keys and other settings depending if you're building in debug or release mode, or based on some other condition all together.

Some .NET platforms have had the concept of `settings.json` files for some time now, but it's not something that's ever been supported in Xamarin.

There are a few different approaches to solving this problem, but I decided I finally wanted something a bit more simple and elegant. This post explores how to easily incorporate such a solution into your Xamarin app.

## Defining a settings.json file

Now, in this case `settings.json` is not quite the same as in ASP.NET apps, and the same API you use there for loading configurations will not work here. That wasn't really my goal, as I mostly just need a simple key/value settings file approach.

My `settings.json` file looks something like this:

```
{
	"apiUrlBase": "https://myserver.com/api/",
	"apiKey": "123456"
}
```  

For local development I'd like to test against the locally running web API instance so I define a `local.settings.json` file:

```
{
  	"apiUrlBase": "https://localhost:5001/api/",
  	"apiKey": "654321"
}
```
    

## Include the settings files in your project

Now the magic of having a `local.settings.json` for debug and a `settings.json` for release is that the build should pick the correct file for the configuration you are building.

This can be done very easily by using some MSBuild condition logic in your .csproj file:

```
<ItemGroup>
	<EmbeddedResource
		Include="settings.json"
		Condition="'$(Configuration)' != 'Debug' or !Exists('local.settings.json')" />
	<EmbeddedResource
		Include="local.settings.json"
		Link="settings.json"
		Condition="'$(Configuration)' == 'Debug' and Exists('local.settings.json')" />
</ItemGroup>
```
    

> Notice I'm including both files as `EmbeddedResource` items. More on this in the next section.

For each file that I've included, I've added conditions for the item. The `settings.json` file will be included if my configuration is not set to `Debug` _or_ if no `local.settings.json` file exists. The `local.settings.json` file will be included if the configuration _is_ `Debug` _and_ the file actually exists.

The other thing to note is that `local.settings.json` has `Link="settings.json"` specified which means it will actually be embedded with the filename of `settings.json`. This means no matter which file is used at build, the resource will be named the same in the output assembly, so we don't have to guess which filename to load at runtime.

You can make these conditions whatever you want. If you have a white label app, you could set a custom MSBuild property to specify the path to the settings file to use:

```
<ItemGroup>
	<EmbeddedResource
		Include="$(CustomerId).settings.json"
		Link="settings.json"
		Condition="'$(Configuration)' != 'Debug' and Exists('$(CustomerId).settings.json')" />
</ItemGroup>
```
    

You could then build with something like `-p:CustomerId=customer1` which would cause the build to use `cusomer1.settings.json`.

## Accessing the settings from your app's shared code

This part is rather easy. Since we used `EmbeddedResource` as the item group name in our .csproj, the json file will be embedded into the output assembly as a resource. We can access it with a short bit of code at runtime:

```
// Get the assembly this code is executing in
var assembly = Assembly.GetExecutingAssembly();

// Look up the resource names and find the one that ends with settings.json
// Your resource names will generally be prefixed with the assembly's default namespace
// so you can short circuit this with the known full name if you wish
var resName = assembly.GetManifestResourceNames()
	?.FirstOrDefault(r => r.EndsWith("settings.json", StringComparison.OrdinalIgnoreCase));

// Load the resource file
using var file = assembly.GetManifestResourceStream(resName);

// Stream reader to read the whole file
using var sr = new StreamReader(file);

// Read the json from the file
var json = sr.ReadToEnd();

// Parse out the JSON
var j = JObject.Parse(json);

var apiUrlBase = j.Value<string>("apiUrlBase");
var apiKey = j.Value<string>("apiKey");
```
    

This simply parses the JSON and manually fetches the key/value pairs. You could of course create a C# class to use for deserialization to support more complex configuration hierarchies.

* * *

There you have it! A rather simple, yet elegant way to pivot your Xamarin app's configuration settings using a simple JSON file and some MSBuild conditions.