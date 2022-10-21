# The problem with Apples Push Notification Service... Solutions and Workarounds...

Push Notifications have intrigued me since Apple first introduced them in iOS years ago. RIM had been doing this for a while, but as a platform it never excited me. As soon as the documentation for their APNs protocol was released I started busily implementing a solution to send push notifications in C#. The first version of the protocol was already terrible, but I’m not going to harp on that as we’ve got a newer version that’s only slightly better to tear apart.

I’m the author of PushSharp ([https://github.com/Redth/PushSharp](https://github.com/Redth/PushSharp)) which is a .NET library to assist developers in delivering push notifications on as many platforms as possible (iOS, Android, Windows, Windows Phone, and HTML5). This library was the culmination of my previous efforts in individual libraries (APNS-Sharp and C2DM-Sharp mostly), and represents a more abstracted, standardized, easier way to support push notifications on all the platforms you may target as a developer.

This post is a chance for me to vent, to explore my frustrations with Apple’s APNS protocol, and hope that they somehow listen and change it. You’ll notice how there’s no complaining about the way the other platforms implement their protocols. This is because they aren’t terrible (though they aren’t perfect either).

Apple’s Enhanced Format for Push Notifications
----------------------------------------------

I had great hopes that Apple finally fixed its protocol when they introduced the Enhanced format (basically v2 of their protocol). Both the original and the enhanced format are binary protocols. You can quickly see the differences between the two in the diagrams below:

**Original Protocol:**

> ![Original-Binary-Protocol](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361506494/TBHgNQAzZ2.jpeg)

**Enhanced Protocol:**

> ![Enhanced-Binary-Protocol](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361507648/tFz-AWWkO.jpeg)

You’ll notice that in the enhanced protocol, there’s two additional bits of information (besides the first byte being 1, indicating that it’s the new protocol as opposed to 0 in the original). These are two very good additions to the protocol:

1.  _Identifier_ – Your own 4 byte data to uniquely identify the notification – this is important as it’s returned to you in the error response packet if there’s a problem.
2.  _Expiry_ – A point in time after which the message is no longer valid if it has not already been delivered and can be discarded from Apple’s servers

In the original protocol, whenever you send Apple a notification that has a problem with it (maybe it’s too big, or it has an invalid device token or malformed payload, etc.), it simply closes the TCP connection without any warning. You are left to assume something is corrupt in your notification.

With the enhanced protocol also handles bad notifications it receives a bit differently. It still closes the connection to you when there’s an error, but before it does so, it sends back an error response packet. You can see the format in the diagram below, along with a list of status codes and their meanings:

**Error Response Packet:**

> ![Error-Response-Packet](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361508769/P2xwTp63k.jpeg)

**Status Codes & Meanings**

*   0 - No errors encountered
*   1 - Processing error
*   2 - Missing device token
*   3 - Missing topic
*   4 - Missing payload
*   5 - Invalid token size
*   6 - Invalid topic size
*   7 - Invalid payload size
*   8 - Invalid token 255 - None (unknown)

This packet is quite simple, with the first byte presumably indicating the protocol version, the second byte being the status or error code (Apple provides us with a list of status codes and what they mean – interestingly enough one of the status codes is ‘0 – No errors encountered’ – just keep this in mind for later). The final piece of info is the Identifier. This identifier will correspond to the identifier of the notification you sent which caused the error condition.

What’s the problem here?
------------------------

So far so good, right? Well, not so much. In theory this all sounds very good. Finally, we get an error response from the service, and some additional functionality. But there are still two major issues with the protocol that you would discover very quickly if you decided to try and implement a client for it yourself:

1.  Apple never sends a response for messages that succeeded.
2.  An error response may not immediately be returned before you have a chance to send more notifications to the stream, but before Apple closes the connection. These notifications that might still ‘make it through’ are left in limbo. They are never acknowledged or delivered, and never have error responses returned for them.

### How could Apple fix this?

Remember how I told you to keep in mind the error response status code of _'0 – No errors encountered’_? This is the silver bullet. If Apple simply always returned an error response, for every notification, even if the notification succeeded, we could simply build a library that wrote a notification to the stream, waited for a response, and then moved onto the next notification, over and over. There’d be no business of waiting around for an error response that might never come, and greatly simplify the pains of implementing this protocol. Apple might argue that this would consume more bandwidth, and while they may be right, in this day and age it would only amount to another ~6 MB per 1,000,000 notifications delivered. Considering that Google and Windows Phone both use HTTP protocols and generate significantly more bandwidth based on the underlying protocol alone, and are able to keep their infrastructure running, an extra 6 bytes per notification should be pocket change to Apple in the cost of maybe a few additional servers and bandwidth allocation.

It’s such a simple answer. It’s even in Apple’s own documentation. Yet, it’s not our reality.

So what is the workaround?
--------------------------

I’ve looked at many libraries, written in many different languages, to see how they worked around this problem. In about 99% of the cases I’ve observed, they all use the same, sadly inefficient approach: Waiting.

So again, we have a connection to Apple’s APNS server, and we want to send notifications over that connection repeatedly, and as fast as possible. Apple never sends us a response if a notification was sent successfully, but if one failed, they will send us an error response and close our connection.

The problem is, if we keep sending notifications over and over again, we might send a second, or third notification before Apple ever sends us an error response for the first one that failed. If this happens, the second and third notifications are never delivered, and are lost forever.

The easiest way to solve this is to asynchronously read from the connection stream, waiting for an error response. In doing so, however, this means you must also wait a little while after you write each notification to the stream to see if your asynchronous read ever receives anything. You can’t just do a synchronous blocking read on the stream since you’d be waiting indefinitely if the notification succeeded (since Apple sends no response in this case). To make matters worse, Apple doesn’t guarantee how quickly an error response will be sent to us. I’ve seen libraries wait for an error response from anywhere between 100 to 500 milliseconds.

It should be painfully obvious to you by now why this approach is flawed. If you have to wait even 100 milliseconds after sending every notification, that would take you almost 28 hours to send 1,000,000 notifications over a single connection!

Most libraries employ the use of multiple connections to circumvent this new issue they’ve created for themselves by waiting between each notification. If you use 10 connections to apple’s servers, that cuts your time down to 2.8 hours. This is better, but why should it take 10 connections 2.8 hours to deliver a theoretical maximum of only 300MB of data (1,000,000 notifications \* 301 bytes maximum size per notification)? This is asinine!

A better workaround
-------------------

I just couldn’t stand the thought of wasting 100-500 milliseconds per notification sent. I figured there had to be a better way, and I think I’ve found it! PushSharp employs a technique that is fairly easy in theory, and was a bit more difficult to implement in code.

Each time a notification is written to the connection stream, it is then added to a _Sent_ queue. If an error response is received, the corresponding notification is located in the ‘Sent’ queue (by its identifier). Anything before the error-causing notification in the queue is removed and assumed to be successfully sent. Anything after the error-causing notification is assumed to be lost, and re-queued to the ‘To Send’ queue to be tried again. There is also a cleanup thread running that constantly checks the oldest notification in the ‘Sent’ queue to see if it’s older than a few seconds and if so, it is assumed to have been successfully sent and is removed from the ‘Sent’ queue. This effectively moves the waiting period for an error response outside of the scope of the connection to the APNS servers. The diagram below illustrates how the Sent queue works:

**PushSharp Sent Queue**

> ![PushSharp-Sent-Queue](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361509854/or8SKALyx.png)

Conclusion
----------

So that’s it, that’s my big speech on why Apple’s Push Notifications are so troublesome, how they could make my life a lot easier with a couple small changes, and how I’ve learned to cope with the situation for now. I hope you enjoyed my ramble, and do please check out [PushSharp](http://github.com/redth/PushSharp)!