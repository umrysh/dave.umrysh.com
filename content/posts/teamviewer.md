---
title: "Removing Teamviewer with Group Policy"
date: 2019-10-25T12:45:11-06:00
draft: false
---

I recently was tasked with removing [Teamviewer](https://www.teamviewer.com/en-us/) from all of the computers in a client's network. Rather than go from workstation to workstation I thought there must be a way to do this by utilizing [Group Policy](https://en.wikipedia.org/wiki/Group_Policy) in [Active Directory](https://en.wikipedia.org/wiki/Active_Directory). After some time spent searching the web I found a way. I am posting my steps here for my own reference and in case it may assist someone else in the future.

The main piece of the puzzle comes in the form of a simple [batch](https://en.wikipedia.org/wiki/Batch_file) file. Create the file "remove_teamviewer.bat" on the desktop of your [Domain Controller](https://en.wikipedia.org/wiki/Domain_controller) and paste the below text into the file.

```
@echo off
if exist "C:\Program Files (x86)\TeamViewer\uninstall.exe" "C:\Program Files (x86)\TeamViewer\uninstall.exe" /S
```

The benefit that I had discovered while scouring the web was that Teamviewer actually has a built in silent uninstaller. You just need to invoke the uninstaller with the `/S` flag to remove Teamviewer without any user interaction. This make things a whole lot simpler. The `if` statement is just used to check if Teamviewer is installed before calling the uninstaller.

Now that you have your batch script created open your Group Policy Management console, right click on the name of your domain and select "Create a GPO in this domain, and Link it here"

![Screenshot.png](/img/tv_6.png)

Give it a name such as "Remove Teamviewer" and then navigate to the "Scripts" section under "Computer Configuration". You will see two options; Startup and Shutdown. I added the batch script to both by double clicking on them one at a time.

![Screenshot.png](/img/tv_3.png)

Let's start with the Shutdown script. Double click it and then click to "Add" a new script. Then click on "Browse". Now drag and drop your batch file into the Explorer window that opened, select it and click "Open". Then click "Ok" on the other windows to close them and start the process over again for the Startup script.

![Screenshot.png](/img/tv_5.png)

I also added the script to the "Logon" section under "User configuration". I don't think I really needed to add it here as well, but I figured why not.

![Screenshot.png](/img/tv_4.png)

Once finished with the configuration I was back on the Group Policy Management console window so I clicked on my new GPO and added a Security Filter so that the script will run only on the group of computers I want

![Screenshot.png](/img/tv_2.png)

Here is a quick preview showing the settings of my GPO

![Screenshot.png](/img/tv_1.png)

And finally, I right-clicked on the GPO and changed it's status to enforced

![Screenshot.png](/img/tv_7.png)

I logged into one of the workstations, ran `gpupdate /force` in a command prompt, and then restarted the computer. After it rebooted I logged in and Teamviewer was no longer installed :)
