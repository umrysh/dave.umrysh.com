---
title: "Setting up a QuickBooks linux database server"
date: 2019-07-10T14:56:33-06:00
draft: false
---
I have a client that has QuickBooks database manager installed on a Windows server to allow multi-user access to the company data file. Everything was working great up until recently so as a test I decided to set up a Fedora server that would run the Linux version of the QuickBooks database manager and test hosting their company data files from it instead. That way we can rule out if the issues they were encountering are being caused by the Windows server. QuickBooks has a really great [tutorial](https://quickbooks.intuit.com/community/Help-Articles/Install-Linux-Database-Server-Manager/td-p/201632) on their website on how to get the database manager installed but there were a few things that it didn't mention so I thought I would make a post detailing the steps in case I ever have to do this again.

On a side-note I hope they will update the Linux database manager soon as the most recent version of Fedora it will run on is 27 and at the time of writing the current version of Fedora is 30. At first I tried installing the database manager on Fedora 30 and it did not like that at all. So I had to track down version 27 from the archives:

`https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/27/Server/x86_64/iso/`

Once Fedora was installed I completed all the normal things you do on a new Linux install: disable root login via SSH, install fail2ban, set a static IP, gave my user account sudo access, etc

Then I installed some dependencies

`sudo yum install samba gamin.i686 libgcc.i686 glibc.i686 libstdc++.i686`

Adjusted the firewall and such

`sudo firewall-cmd --permanent --add-port=10172/tcp`

`sudo firewall-cmd --permanent --add-port=8019/tcp`

`sudo firewall-cmd --permanent --add-port=55368-55387/tcp`

`sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1`

`sudo firewall-cmd --permanent --add-service=samba`

`sudo firewall-cmd --reload`

Then I grabbed the database manager

`wget https://http-download.intuit.com/http.intuit/Downloads/2019/nrnt7lbwhwUS_R1/qbdbm-29.0-1.i386.rpm`

and installed it

`sudo rpm -ivh qbdbm-29.0-1.i386.rpm`

I made a directory to store the QuickBook files

`sudo mkdir /mnt/quickbooks`

opened up the monitor config file and set it to the folder I just created

`sudo nano /opt/qb/util/qbmonitord.conf`


I then created a new group for my Samba share

`groupadd -r qbusers`

Created a new QuickBooks user for Samba (everyone is just going to use the same Samba user)

`sudo useradd -m qbuser`

`sudo usermod -a -G qbusers qbuser`

`sudo passwd qbuser`

`sudo smbpasswd -a qbuser`

Changed the folder permissions on the Samba folder

`sudo chmod -R 775 /mnt/quickbooks`

`sudo chgrp -R qbusers /mnt/quickbooks`

I then added the following to the bottom of `/etc/samba/smb.conf`

```
[quickbooks]
    path = /mnt/quickbooks
    comment = samba share for company files
    valid users = qbuser
    public = no
    writable = yes
    printable = no
    create mask = 0765
```

Then I restarted Samba

`sudo systemctl enable smb`

`sudo systemctl start smb`

At this point I could now map the folder from my Windows laptop and add and remove files (yay!)

Next I just restarted the QuickBook init scripts

`sudo /etc/init.d/qbdbfilemon restart`

`sudo /etc/init.d/qbdbmgrn_29 restart`

The last step was to test if the QuickBooks server actually works. Since I do not personally use QuickBooks I downloaded the [30 day trial](https://quickbooks.intuit.com/community/Help-Articles/Download-a-trial-of-QuickBooks-Desktop/m-p/185974) version and created a sample company. I copied the sample company files to the newly mapped drive and tried to open the files....

It worked!

Now to take the server over to the client's office and run some tests :)
