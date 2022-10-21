# iTunes Media HotKey Disabler

This simple application will allow you to disable or enable iTunes from listening to when you press the media keys (Play / Pause) on your keyboard. This is great for applications such as gTunes which make use of the hot keys but cannot themselves prevent iTunes from also responding to those keyboard keys.

![itunes-media-hotkey-disabler](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361451501/Luwi8aB7q.png)

*   [Download **iTunes.Media.HotKey.Disabler.1.0.0.dmg**](https://googledrive.com/host/0B5rPKkNu1C2INC13ZEV5dGFmYk0/iTunes.Media.HotKey.Disabler.1.0.0.dmg)

Why create this app?
--------------------

While working on an upcoming Google Music app (gTunes) that’s currently waiting for approval on the app store, I wanted to make use of the media keys (Play/Pause, Next, Prev, etc.). This was easily enough done, however there was one major issue. Apple does not expose any API’s to prevent iTunes from opening when the Play/Pause keyboard button is pressed on a Mac. There are ways around this, but to my limited Mac development knowledge, none of them would be allowed on the App Store.

How does it work?
-----------------

The most consistent and easiest way I’ve found to stop iTunes from listening for the Play/Pause button key was to stop the Launch Agent _**com.apple.rcd**_

If you’re more comfortable with Terminal, you can do the exact same thing that this app does with the command:

    launchctl unload -w /System/Library/LaunchAgents/com.apple.rcd.plist
    

Similarly, the Launch agent can be re-enabled so that iTunes listens for the Play / Pause key again with the command:

    launchctl load -w /System/Library/LaunchAgents/com.apple.rcd.plist
    

More interesting things
-----------------------

It’s worth noting I don’t know a lot yet about Mac development, but I’ve been able to create this app (and the upcoming gTunes) using [Xamarin.Mac](http://xamarin.com/mac), creating a fully native app all in C#! You should check out [Xamarin](http://xamarin.com/)!