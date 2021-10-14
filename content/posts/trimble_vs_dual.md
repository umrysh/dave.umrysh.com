---
title: "Trimble vs Dual"
date: 2019-07-23T18:00:08-06:00
draft: false
---

![Trimble R1 and a Dual XGPS150A](/img/trimble_vs_dual_sm.jpg)

So I ran into an interesting issue today when attempting to get [Golden Retriever](http://goldenretrieverapp.com) (GR) to connect to a [Trimble R1](https://www.trimble.com/r1) device. I would pair the R1 to the phone and then attempt to connect to it from within GR using the "Bluetooth GPS" button, but nothing would happen. No points would be pulled. I checked the logs coming from the phone and I was getting a "`bt socket closed, read return: -1`" error each time GR tried to use the Bluetooth serial connection to read from the R1.

I have not ran into this issue before as we usually recommend clients use the [Dual GPS](https://gps.dualav.com) badges as they have always worked great. You just pair them to the phone and then open the GR menu and click on "Bluetooth GPS". The GR app will create a socket connection to the GPS badge and listen for any location changes. Easy peasy.

So after much searching I did find this post from over a year ago: [https://community.trimble.com/thread/3780-bluetooth-communication-with-trimble-r2](https://community.trimble.com/thread/3780-bluetooth-communication-with-trimble-r2)

In the post someone is asking Trimble for help reading from a R2 device via a Bluetooth serial connection. Alas, from the response the Trimble rep left it sounds like Trimble has decided to do their own thing and not make the Bluetooth GPS device directly accessible via Bluetooth serial.

In the post he does name drop their SDK and the "GNSS Direct" app so I started to look into those options. I really didn't want to incorporate Trimble specific code into GR if I could avoid it so I decided to focus my attention on the "GNSS Direct" app.

And I am glad I did.

After installing the "GNSS Watch" app the first thing it does is request that I install the "GNSS Direct" app as this is where the actual magic happens. The "GNSS Direct" app can be enabled as a mock location app in Android. Which means all we have to do is connect the Trimble R1 to the Android device and it will begin feeding it's coordinates into Android in place of the phone's built in internal GPS radio.

Well, in theory that was all I had to do but I of course ran into another issue in the process. I had been using the [cordova-plugin-bluetooth-geolocation](https://github.com/heigeo/cordova-plugin-bluetooth-geolocation) library and apparently it does not like mock location providers. Luckily I'm not the first person to encounter this and a solution was already posted on [Stack Overflow](https://stackoverflow.com/questions/35899557/accessing-mock-bluetooth-gps-coordinates-in-cordova-geolocation). It involves utilizing a different location library when on Android, which is no biggie. I adjusted the GR code to make use of this other library when on Android and then everything worked!

>One last thing I will mention is when you are finished using the Trimble R1 open the "GNSS Watch" app, go to the "Source" page, change the "Position Source" to "Location Services" and click "select". From my limited testing this seems like the only way to turn off the Trimble app




### Here are the complete steps to connect a Trimble R1 to GR:

Enable Bluetooth on your Android device and pair the Trimble device to your phone

![Screenshot_20190723-161718.png](/img/Screenshot_20190723-161718.png)

Open the Play store and install the GNSS Watch app

![Screenshot_20190723-161810.png](/img/Screenshot_20190723-161810.png)

When you open the app it will send you to the Play store again to install the GNSS Direct package

![Screenshot_20190723-162137.png](/img/Screenshot_20190723-162137.png)

Open the GNSS Watch app and select the "Source" option from the menu

![Screenshot_20190723-161842.png](/img/Screenshot_20190723-161842.png)

You will then be asked to enable mock locations. Click to open Settings

![Screenshot_20190723-161855.png](/img/Screenshot_20190723-161855.png)

Scroll down and find the option called "Select mock location app" and click on it

![Screenshot_20190723-161913.png](/img/Screenshot_20190723-161913.png)

It will likely be a short list. Select the GNSS Status option from the list

![Screenshot_20190723-161923.png](/img/Screenshot_20190723-161923.png)

Now head back to the GNSS Watch app, go back into "Source" and click on your Trimble R1 and then click on "Select"

![Screenshot_20190723-162058.png](/img/Screenshot_20190723-162058.png)

While the GNSS Watch app is attempting to connect to your R1 you can re-open the menu and select the "Home" option. Eventually you will see something like this:

![Screenshot_20190723-162428.png](/img/Screenshot_20190723-162428.png)

Now minimize the GNSS Watch app and open GR. You *do not* need to click on the "Bluetooth GPS" menu option in GR. Simply open your next collection record and you will notice the accuracy of your internal GPS has become much higher. This is because the Trimble R1 is using it's own application to feed GPS coordinates directly into Android, GR just needs to ask Android for the best coordinate it has, which is the default action.

![Screenshot_20190723-162607.png](/img/Screenshot_20190723-162607.png)


### And for completeness, here are the steps to connect a Dual XGPS150A to GR:

Enable Bluetooth on your Android device and pair the Dual device to your phone

![Screenshot_20190723-161439.png](/img/Screenshot_20190723-161439.png)

Open GR and from the menu select "Bluetooth GPS"

![Screenshot_20190723-161452.png](/img/Screenshot_20190723-161452.png)

After a few seconds GR will let you know if it has connected to the external GPS device

![Screenshot_20190723-161530.png](/img/Screenshot_20190723-161530.png)

When you open your next collection record you will notice it's name in the location pop-up

![Screenshot_20190723-161536.png](/img/Screenshot_20190723-161536.png)