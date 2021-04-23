---
title: "Weird Cordova Error on iPad"
date: 2021-04-23T14:09:24-06:00
draft: false
---

I ran into a sneaky bug in a recent Cordova app I was building. It would run fine on Android and iPhone but when I tried to run it on an iPad, it would crash. And it was a sneaky crash as well. Nothing out of the ordinary in the Apple Console app and the errors thrown in the Safari console were all about plugins not being defined.

The actually HTML and CSS ran fine, just all the plugins came back undefined.

After searching for anyone else with the same problem I found someone who recommended inserting the below code at the top of index.html as the first Javascript in the page:

```
<script type="text/javascript">
window.onerror = function (errorMsg, url, lineNumber) {
  alert('Error: ' + errorMsg + ' Script: ' + url + ' Line: ' + lineNumber);
}
</script>
```

Doing so gave me the super descriptive error alert of "Error: Script error Script: Line: 0"

My next step was to remove all Javascript from the app which then allowed for the app to run on the iPad without an error. I then begun slowly adding back Javascript until I encountered the same error message. It took me a long time but the issue seemed to be when I called `ons.ready(onDeviceReady);` (I am using the [Onsen UI](https://onsen.io) template for this particular app)

This got me thinking, maybe there is an issue with Onsen...

So I changed `ons.ready(onDeviceReady);` to `document.addEventListener('deviceready', onDeviceReady, false);`

This time when I ran the app there were no errors, but `onDeviceReady` was never called. That is when I had my eureka moment. When you add the Javascript import statements for Onsen you don't actually import cordova.js as it is supposed to handle that for you. However, the fact that the event `deviceready` never gets called and that plugins are undefined must mean that Cordova is not being added to the project when being run on an iPad...

So I added `<script src="cordova.js"></script>` into the `<head>` of my index.html and it worked!!

The only issue is for all the other devices where things had been working I now have an error about Cordova already being imported, but it doesn't actually cause any issues at this point so I am considering this a win.

