---
title: "My first TWA"
date: 2019-07-24T12:44:20-06:00
draft: false
---

When I decided to make the jump from coding just native Android apps to a cross-platform solution a few years back I choose [Cordova](https://cordova.apache.org). This meant that some of my simpler apps could very easily be converted in [Progressive Web Apps](https://en.wikipedia.org/wiki/Progressive_web_applications) (PWA) which is fantastic for the very important reason that your users can install your app without needing to go through the Google Play store or the Apple App store. Your user simply visits your app's website and they are then asked if they wish to install your PWA to their home screen. If they choose not to they can continue to use you app just like any other website.

The beauty of this is when I want to push an update I just need to update the files on the website and the next time the user loads my app they will be told there is an update. After dealing with the lengthy delays imposed by Apple's App store this is like a breath of fresh air. PWAs are not yet as fully supported on iOS but Apple is being slowly pulled kicking and screaming into the PWA world. So I am hopeful iOS users will be able to enjoy the full benefits of PWAs soon.

My first foray into PWAs was with the [Work Tyme app](https://app.worktyme.ca). I still maintain both an Android and iOS version of the Work Tyme app but have been suggesting users move to the PWA for the simple reason mentioned above, it receives updates the quickest.

>Please note that the Work Tyme PWA has not been properly coded to work well on desktop browsers so please be aware of that before you click on the link. I plan to get that fixed in the future.

Last week I randomly started reading up on [Trusted Web Activities](https://developers.google.com/web/updates/2019/02/using-twa) (TWA) which sounds fantastic! It means I could convert my Android app into a TWA, push it to the Play store, and then never have to worry about it again. The TWA would, in essence, function as an installable wrapper for my PWA. Going forward whenever I update my PWA, my TWA is automatically updated as well. This drops me from maintaining three versions of the Work Tyme app down to two. If someday iOS should support TWAs, I would be down to a single version with live updating! It may never happen, but a man can dream can't he?

To create my TWA I first attempted to use the [PWA Builder](https://www.pwabuilder.com) from Microsoft to generate a TWA. I had heard others had used it for that purpose but I was really disappointed with the results. So I instead opted to download a copy of the [svgomg-twa](https://github.com/GoogleChromeLabs/svgomg-twa) code and tweak it to work with my Work Tyme PWA.

The first thing that I needed to do was create a [Digital Asset Link](https://developers.google.com/digital-asset-links/v1/getting-started) and have it posted at [https://app.worktyme.ca/.well-known/assetlinks.json](https://app.worktyme.ca/.well-known/assetlinks.json) so that my TWA would be allowed to access my PWA. The file itself is pretty much just a copy and paste except I needed my unique certificate fingerprint. To find it I simply logged into my Android developer account, navigated to app signing and copied the SHA-256 certificate fingerprint (without the SHA256: at the beginning).

I then opened the "`/app/build.gradle`" file and adjusted the settings in the following section to be for my Work Tyme App instead

```
def twaManifest = [
    applicationId: 'org.chromium.twa.svgomg',
    hostName: 'svgomg.firebaseapp.com', // The domain being opened in the TWA.
    launchUrl: '/', // The start path for the TWA. Must be relative to the domain.
    name: 'SVGOMG TWA', // The name shown on the Android Launcher.
    themeColor: '#303F9F', // The color used for the status bar.
    backgroundColor: '#bababa', // The color used for the splash screen background.
    enableNotifications: false // Set to true to enable notification delegation
]
```

This is my current version:

```
def twaManifest = [
    applicationId: 'com.umrysh.worktyme',
    hostName: 'app.worktyme.ca', // The domain being opened in the TWA.
    launchUrl: '/', // The start path for the TWA. Must be relative to the domain.
    name: 'Work Tyme', // The name shown on the Android Launcher.
    themeColor: '#4383cd', // The color used for the status bar.
    backgroundColor: '#efeff4', // The color used for the splash screen background.
    enableNotifications: false // Set to true to enable notification delegation
]
```

I also made sure to change the `versionCode` and `versionName` values in this file to be the ones needed for my application. Lastly, I went through all the `drawable` and `mipmap` folders located in "`/app/src/main/res/`" and changed the icons to have the Work Tyme logo instead. Once that was all completed I opened the folder in [Android Studio](https://developer.android.com/studio/) and built the app. It really was just that easy.

The new TWA version of the Android app has been in the Play store for a few days now and I have not heard a single complaint.