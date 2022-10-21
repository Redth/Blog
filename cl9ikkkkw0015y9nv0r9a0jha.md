# Finding your Android app's MD5 or SHA1 Signature from your keystore

![Grump Cat does not like keystores](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361425517/T4yucdY0S.jpeg)  
If you’ve ever used Google Maps or Amazon App Services in your Android app, you’ve probably gone through the pain of having to find an MD5 or SHA1 signature to give to the service so they could generate an API Key for you.

If you haven’t been through this experience, let me assure you: It’s painful, tedious, and annoying.

First of all, you need to find out which .keystore file was used to sign your app. This is probably different depending on if you’re running a debug build versus a signed build for the app store.

If you’re a [Xamarin.Android](http://xamarin.com/android) user, you may not have even been aware that a `debug.keystore` file was created for you at some point and is used to sign all of your debug builds, so you can remain blissfully unaware that it exists, or where it even lives.

There’s a great article on the Xamarin Docs Site: Finding your Keystore’s MD5 or SHA1 Signature that shows you how to generate these values step by step. Put aside 15 minutes to go through it though (and hey, I wrote the initial draft of the doc, it’s as short as Google allowed me to make it, I swear!).

Enter, Android Signature Tool.
------------------------------

If you’ve been to my blog, you probably know that I don’t like to complain about problems that I can’t offer a solution for, so today is your lucky day!

[![Android Signature Tool](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361427124/IUTANg7wp.png)](http://res.cloudinary.com/redth-codes/image/upload/v1508287575/android-signature-tool-screenshot_uvj8ex.png)

I’ve created a little tool called _**[Android Signature Tool](https://github.com/Redth/Android.Signature.Tool/)**_ (yes, the creative juices were flowing with that one). It’s a simply GUI app (with a GUI for both Mac and Windows) that aims to minimize the tears spent on getting MD5 and SHA1 signatures.

It’s very simple. It will try and first find java’s _**keytool**_ executable on your machine (which is needed to generate the signatures). The default option is to try and find your Xamarin generated _**debug.keystore**_ file for you, and spit out the MD5 and SHA1 signatures with the click of a button! You can also specify your own .keystore file (for which you need to know the alias, storepass, and keypass you used when you created it).

Hopefully this tool makes your life just a little bit more simple. Enjoy!