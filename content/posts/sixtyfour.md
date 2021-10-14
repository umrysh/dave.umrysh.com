---
title: "Sixtyfour"
date: 2021-03-15T11:34:35-06:00
draft: false
---

If you say it can't be done, I will try my best to find a way :)

One of my clients utilizes the [8x8](https://www.8x8.com) service to handle their office phone system and in turn it is their defacto office messaging platform as well. It is a great system and does what it does well.... except there is no Linux client.

This means I am always stuck writing messages through the Android app and manually typing out urls and error messages.

I have my own [Matrix server](https://www.8x8.com) that I host for my friends. I really enjoy how well it works and I have built a few different bots to allow for automated messaging. Such as typing `!$ tsla` and getting the current stock price as a reply.

So my goal was to mirror my 8x8 account into respective Matrix channels so I could read and respond to messages from within Matrix.

8x8 offers a product called [SameRoom](https://sameroom.io) which at first glance seemed like it could work except that it doesn't. It feels like it was abandoned years ago. And if I did get it work it would only work for public rooms on 8x8, DMs are not able to be synchronized. Which is not ideal.

My next thought was that I could reverse engineer the Android app but that could lead to a break of the terms of service and cause issues for my client's account should 8x8 get angry about it.

Then I came up with a neat hack. I noticed all the 8x8 notifications that appeared on my Android phone had the Android quick reply options. So I thought if I could write an Android app that read all incoming notifications, check if it was coming from 8x8, if so send a copy to Matrix, and then save the notification intent.

Then I would have a Matrix bot that would listen for `!m` and sent whatever followed to my Android app (via Google Cloud Messaging). Then the Android app would post the text as a quick reply to the most recently saved Notification.

It was crazy enough that it could work. I also have an Android phone that is mounted to the back of my monitor that is always on which would be perfect for running this (I use it as my webcam via the awesome [Droidcam](https://www.dev47apps.com) software)

I decided to use [Cordova](https://cordova.apache.org) for the app as it is what I am most comfortable with. I found a great plugin called [NotificationListener](https://github.com/coconauts/NotificationListener-cordova) that I based most of the "incoming message" part of the app off of and then added in [my own code](https://git.umycode.com/dave/cordova-plugin-reply-to-notification) to handle the saving of the intents and the sending of the outgoing messages.

So far it has worked well. It has also allowed me the ability to build bots to send automated messages to the different 8x8 rooms I belong to. Such as deploy messages from my deploy bot.

However not everything works as great as I want:

* Photos: as the notifications coming from 8x8 only contain text that means any photos that co-workers post get mirrored to Matrix as `[image.png]` which can be a pain when troubleshooting issues. I have gotten around this on my end by sending URLs to images which co-workers can click on to view my screenshots.

* The latest version of the 8x8 client appears to neuter urls that co-workers post.

* In a group 8x8 chat room I am able to reply to the same Notification many times however I am only able to reply once to a direct message notification. This means I need to choose my replies carefully. Even if my coworker sends me five messages in a row in our DM I only get one reply until they send the next message.


However even with those caveats this was a fun little project.

If you are interested in seeing the code I have posted a version of the app here: [https://git.umycode.com/dave/sixtyfourapp](https://git.umycode.com/dave/sixtyfourapp)


