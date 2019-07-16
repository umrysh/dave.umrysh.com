---
title: "PowerShell makes file backup easy"
date: 2019-07-16T12:00:40-06:00
draft: false
---

I personally do not write in [PowerShell](https://en.wikipedia.org/wiki/PowerShell) often but sometimes PowerShell is the best language for the job. For example, backing up your [Sage 50](http://www.sagesoftware.ca/ca/sage-50-accounting) folder from your Windows server to a folder on your Network Attached Storage(NAS) at regular intervals with [Windows Task Scheduler](https://en.wikipedia.org/wiki/Windows_Task_Scheduler). I also take full image backups regularly of the Windows server with [Iperius](https://www.iperiusbackup.com) in case the entire server goes down, but having regular backups of just the Sage 50 files allows us to quickly roll back any Sage database changes. Plus the NAS is in a RAID 6 and backs up regularly to [Amazon S3](https://en.wikipedia.org/wiki/Amazon_S3), where I have versioning enabled as well. So getting the Sage 50 database files onto the NAS adds a whole lot of redundant data security.

> Note: My company, [Umycode Technologies](https://umycode.com), is a approved reseller of Iperius products and therefore can provide the software at a 20% discount from retail. If you are in need of a great backup solution please [reach out](mailto:dave@umycode.com).

Here is the PowerShell script I use which is saved in the root of the `C:/` drive and named `Backup.ps1`

```
Add-Type -AssemblyName System.Web
net use "\\192.168.150.11\backup" /USER:domain\user "password"
Robocopy "C:\accounting" "\\192.168.150.11\backup\sage\accounting\" /PURGE /E /W:5 /r:2
if ($lastexitcode -lt 9)
{
    $dateTime= (Get-Date).ToString()
    Add-Content -Path "\\192.168.150.11\backup\sage\lastBackup.log" -Value "`r`n$dateTime"
    $encmsg = [System.Web.HttpUtility]::UrlEncode("Sage 50 accounting drive backup succeeded")
    Invoke-WebRequest -Uri "http://example.com/powershellpost.php?message=$encmsg"
}
```

The second line the script mounts the backup folder using the credentials given. Simply replace the network location of your backup folder as well as the values for **domain\user** and **password** with your own credentials.

>I know it is not recommended to store plain text credentials in a script but in this case the risk is low. The admin account is the only account that can access this server and the admin account already has access to the backup drive.

I then use Robocopy to mirror the Sage 50 folder (the `C:\accounting` folder) to the proper sub-folder in the backup location.

If all goes well then the execution will enter the `if` statement which writes out the current time to a log file and then performs a `GET` request to a [PHP](https://en.wikipedia.org/wiki/PHP) script I have located on my webserver which acts as a email relay for my numerous scripts. This PHP script simply sends me an email via [Mailgun](https://www.mailgun.com) with the text in the `$encmsg` variable. I could add some authentication to the PHP script but I have yet to have any bots hit the script and execute it, so it's not a priority for me at this stage. Here is a copy of the PHP script:

```
<?php
    if ( isset($_REQUEST['message']))
    {
        $to = "me@email.com";

        $contents = $_REQUEST['message'];
        $subject = $_REQUEST['message'];
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
        curl_setopt($ch, CURLOPT_USERPWD, 'api:key-XXXXXXXXXXXXXXXX');
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'POST');
        curl_setopt($ch, CURLOPT_URL,
            'https://api.mailgun.net/v3/email.com/messages');
        curl_setopt($ch, CURLOPT_POSTFIELDS,
            array('from' => 'do_not_reply <do_not_reply@email.com>',
                'to' => $to,
                'subject' => $subject,
                'html' => $contents));
        $result = curl_exec($ch);
        curl_close($ch);
    }
?>
```

Finally, the last piece of the puzzle is setting up the PowerShell script to run at regular intervals via Window Task Scheduler. There are plenty of tutorials online with how to add a basic task to Task Scheduler so I won't go into that here, but you will need to add an extra argument to the **Actions** tab to allow PowerShell the ability to execute the script.

You will want to set the **Action** to **`Start a program`** and under the settings section set **Program/script** to **`Powershell.exe`** and set **Add arguments** to **`-ExecutionPolicy Bypass C:\Backup.ps1`**.

And that's it!

A nice quick and easy way to regularly mirror a folder to a network location