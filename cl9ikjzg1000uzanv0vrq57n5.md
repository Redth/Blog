# CyanogenMod for your Xamarin apps

CyanogenMod is an aftermarket Android firmware which keeps growing in popularity. With the first release of their platform SDK, it’s now possible to harness the power of the Quick Settings pulldown menu in your own apps running on CyanogenMod.

Quick Settings Custom Tiles are a great way to add an easy, global way for your users to do things like toggle a feature of your app on and off.

Adding a Quick Settings Custom Tile to your Xamarin.Android app is easy now with the new [CyanogenMod SDK Component](https://components.xamarin.com/view/cyanogen-platform-sdk) on the Xamarin Component Store.

After you’ve added the component to your project, you can use the `CustomTile.Builder`to build your tile:

    var customTile = new CustomTile.Builder (this) 
      .SetOnClickIntent (pendingIntent)
      .SetContentDescription ("Monkey Toggler")
      .SetLabel ("Monkeys " + States.Off)
      .SetIcon (Resource.Drawable.ic_launcher)
      .Build ();
    

Once you’ve created your custom tile, you can publish it with the `CMStatusBarManager`:

    CMStatusBarManager.GetInstance (this).PublishTile (CustomTileId, customTile);
    

The `CustomTileId`is a unique ID associated with your tile. When you publish a new tile with the same ID, it will update the existing tile under that identifier. This makes it possible to change the appearance based on the current state of your app.

Don’t worry if your app isn’t running on CyanogenMod, in that case the SDK gracefully fails. You won’t see any exceptions, but rather, no Custom Tiles will be published on platforms which do not support it. You can feel safe in adding this feature to your app!

This is the first and only feature that CyanogenMod has released in its Platform SDK. I’m excited to see what new features they add in the near future! Go [grab the component](https://components.xamarin.com/view/cyanogen-platform-sdk) today and give your CyanogenMod users a greater user experience!