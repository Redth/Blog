# Get your MonoTouch apps ready for iPhone 5 / iOS 6 today!

So the new iPhone 5 has arrived, as well as the Gold Master iOS6 SDK! Hurray!

Unless you’ve been under a rock, you know that the iPhone 5 has a taller screen, which means a new screen resolution to deal with for us developers! Apple is lacking bit in their own documentation, but a few developers have figured out how to best handle the new resolution.

First of all, the new resolution is _640×1136_, that’s 640 pixels wide, by 1136 high. Now, just like the Retina display, even though this is the resolution, many parts of the iOS API report back the sizes in points rather than pixels, which means the new size in points is really _320×568_. I’d like to share with you how you can get your MonoTouch apps up and running to be compatible with the new resolution today!

First, Install the X-Code 4.5 GM iOS6 SDK from the Apple Developer site
-----------------------------------------------------------------------

It’s listed as the iOS6 beta at the time of writing this post, but it’s really GM once you get to the download section. You must be part of the paid Apple Developer program to be able to access this. Download the .dmg, mount it, and drag XCode into your Applications folder. You may want to rename the old one so you can revert to it if need be. I renamed the old one _XCode 4.4.1.app_.

Create a Default.png of the correct size
----------------------------------------

The way that iOS6 determines if your app is Tall compatible or not is by looking for a specifically sized and named image file for your splash/default screen. The name of this file is:

> Default-568h@2x.png

This file must actually be _640×1136_ pixels in size.  
Add this file to your MonoTouch Application Project, and make sure the Build mode is set to Content.!

Change the Simulator to iPhone (Retina 4-inch)
----------------------------------------------

After your MonoTouch Application starts in debug mode, you may need to change the Simulator to use the new hardware type. Go to the Hardware -> Device menu, and select “_iPhone (Retina 4-inch)_”. You may need to re-launch your app after doing this.

Presto, your app now runs in ‘Tall’ mode!

Create new sizes of other Image assets
--------------------------------------

Unfortunately, the image naming convention of -568h@2x.png only seems to extend to the Default image, but not other images in your application. This means if you’re using a custom background image for your view (eg: UITableView background), you will likely need to create a new background image of the correct resolution, and determine in your application when to use each image.

For example, in my non-4inch compatible app, I have two images:

1.  Images/TableViewBackground.png – _320×358_
2.  Images/TableViewBackground@2x.png – _640×716_

With the new resolution, I’ll need to create a third image (I’ve decided to use the -568h@2x.png naming convention even though it isn’t observed by Apple):

1.  Images/TableViewBackground-568h@2x.png

Write some code to detect iPhone 5 and use the new Image Assets
---------------------------------------------------------------

Now as I mentioned, even though I’ve named the image just like the way Apple detects the new default image, iOS6 (at least currently) doesn’t know to use it automatically, so we have to write some code to detect when the app is running on an iPhone 5, and then to use the correct image when that’s the case.

Here’s some sample code to detect whether or not we have an iPhone 5. I’ve elected to call it ‘Tall’ detection, as the iPhone 6 will likely use this tall resolution too. **NOTE:** Thanks to [@Clancey](http://twitter.com/jtclancey "@jtclancey") for some additions to the IsTall detection!

    public static bool IsTall {
        get {
            return UIDevice.CurrentDevice.UserInterfaceIdiom == UIUserInterfaceIdiom.Phone 
                && UIScreen.MainScreen.Bounds.Height * UIScreen.MainScreen.Scale >= 1136; 
        }
    }
    

Finally, the approach I take in my applications is to use a common property for the image, and make that property getter decide on the height of the device, and which image to return:

    static UIImage tableViewBackgroundImage = null;
    
    public static UIImage TableViewBackgroundImage {
        get {
            if (tableViewBackgroundImage == null)
                tableViewBackgroundImage = IsTall ? UIImage.FromFile("Images/TableViewBackground-568h@2x.png") : UIImage.FromFile("Images/TableViewBackground.png");
            return tableViewBackgroundImage;
        }
    }
    

In this example, I keep a single static instance of my background image around, and lazy load it based on whether or not it’s a tall device. This way the same image instance is shared for multiple viewcontrollers that use it! In the case where it’s not a tall device, Apple still obeys the @2x.png naming convention to load retina graphics automatically, so you still don’t have to do that detection, it’s done for you.

Other considerations
--------------------

If you have any other code that does anything with height calculations, or maybe creating a view to be a certain height, you may also need to update this code. The general rule is you should now be calculating heights and y-coordinates based on the UIScreen.MainScreen.ApplicationFrame or UIScreen.MainScreen.Bounds. This is probably a good practice anyway, and hopefully you don’t need to do too much work since you’ve already been following that good practice, right?

Hopefully that helps get your MonoTouch apps prepared for iPhone 5, before it’s even out!

UPDATE:
=======

### Select the right Image!

[@martinbowling](http://twitter.com/martinbowling "@martinbowling") contributed another nice method to add to your ‘helper’ class shortly after this post was originally published. It will help you select the right Image in code, easily! It expects that you name your ‘Tall’ image files with the _\-568h@2x_ convention. This is another, even better technique than I’ve illustrated above. Perhaps Xamarin will adopt this method in the future…

    private static string tallMagic = "-568h@2x";
    
    public static UIImage FromBundle16x9(string path) 
    {
        //adopt the -568h@2x naming convention
        if(IsTall()) { 
            var imagePath = Path.GetDirectoryName(path.ToString());
            var imageFile = Path.GetFileNameWithoutExtension(path.ToString()); 
            var imageExt = Path.GetExtension(path.ToString());
            imageFile = imageFile + tallMagic + imageExt;
            return UIImage.FromFile(Path.Combine(imagePath,imageFile));
        } else {
            return UIImage.FromBundle(path.ToString());
        }
    }