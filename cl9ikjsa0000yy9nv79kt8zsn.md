# PushSharp 3.0 - The Push Awakens!

![PushSharp 3.0](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361389396/gS_qZu_wE.png)  
Some of you may be familiar with a little library called [PushSharp](https://github.com/Redth/PushSharp).  If you’re not, it’s a C# library that helps developers send push notifications to various platforms.  I started this effort a few years ago, and somehow during those years, seemingly despite my best effort, this thing became [fairly popular](https://github.com/search?l=C%23&p=2&q=stars%3A%3E1&s=stars&type=Repositories).

Now, more recently (ok-ok, for the last year at least), PushSharp has been sorely neglected. It still works for the most part, especially if you fork it and merge in some of the open pull requests to improve it like the _[many](https://github.com/Redth/PushSharp/network)_ others that have already done so, but it’s old, it’s out-dated, and it needs some love.

While I can make a number of excuses for the project’s inactivity (like a new addition to my family, a new job, and a new house), It’s clear that developers still want a C# library to send push notifications, and my suggestion to use Amazon EC2 and Azure (both great economical alternatives) just doesn’t cut it.

![evolve-2014](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361390459/8W1vOrIqg.png)  
At the last Xamarin Evolve conference (I now work at Xamarin, by the way) I was humbled by the number of people who flagged me down and asked me about PushSharp. In fact, I felt ashamed for pretty much abandoning the project at that point, and vowed to myself to make things better. I decided on the flight home to start rewriting the library, this time with the wonderful `async/await`, `Tasks`, and `HttpClient` at its core.

If you’re keeping score, Evolve was awhile a long time ago now. I lost focus again since I started that rewrite, with one of the big reasons being that I had become extremely frustrated with Apple’s APNS protocol ([again](http://redth.codes/the-problem-with-apples-push-notification-ser/)), or more-so its lack of defined behaviour, unorthodox design, and lack of resources available to test this thing at scale. This is and always has been the achilles heel of PushSharp.

Enter WWDC15. Although I didn’t notice it until very recently, Apple announced they are going to add a HTTP/2 based protocol for sending APNS notifications, this time with a success/fail response for each message, and based around a documented standard. Hearing this news made me _**extremely**_ excited, and motivated to get back to work on PushSharp.

At last.
--------

Enter PushSharp 3.0.  I’ve invested more time and energy into [PushSharp v3.0](https://github.com/Redth/PushSharp/tree/3.0-dev#pushsharp-v30---the-push-awakens), and this time, by virtue of public announcement, am dedicating myself to completing it!

The API has been compeltely overhauled to make extensive use of `Tasks`, `async/await`, and `HttpClient` as I mentioned before.  The library already scales better, and is more reliable, and I do believe this should be a worthy successor to the neglected bits currently rotting away.

With this release, the focus is on simplicity and reliability.  This also means API has changed quite a bit.  For instance you’ll be responsible for constructing your own message payloads on iOS and Windows now (the old fluent helper methods became obsoleted too quickly and introduced too many bugs).  Don’t let that fool you though, it’s still pretty simple to use:

    // Configuration
    var config = new ApnsConfiguration ("push-cert.pfx", "push-cert-pwd");
    
    // Create a new broker
    var broker = new ApnsServiceBroker (config); 
    
    // Wire up events 
    broker.OnNotificationFailed += (notification, exception) => { 
      Console.WriteLine ("Notification Failed!");
    };
    broker.OnNotificationSucceeded += (notification) => { 
      Console.WriteLine ("Notification Sent!");
    };
    
    // Start the broker
    broker.Start ();
    
    // Queue a notification to send
    broker.QueueNotification (new ApnsNotification {
      DeviceToken = "device-token-from-device", 
      Payload = JObject.Parse ("{\"aps":{\"badge\":7}}") 
    }); 
    
    // Stop the broker, wait for it to finish
    broker.Stop ();
    

So, give the [README](https://github.com/Redth/PushSharp/tree/3.0-dev#pushsharp-v30---the-push-awakens) a look, and jump on the [Gitter.im](https://gitter.im/Redth/PushSharp) channel if you have questions or are interested in helping me test at scale.

Currently all of the 3.0 code is available in the **_[3.0-dev](https://github.com/Redth/PushSharp/tree/3.0-dev)_ [branch](https://github.com/Redth/PushSharp/tree/3.0-dev)**.  You can find _**\-PreRelease**_ NuGet packages on [NuGet.org](https://www.nuget.org/packages/PushSharp/) too.  The API is quite a bit different than before.

Thanks to everyone who has used PushSharp, who has said nice things about it (deserved or otherwise), and hopefully this library still has a bright future ahead!