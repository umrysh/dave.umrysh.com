---
title: "Data Warehouse"
date: 2021-03-19T12:37:40-06:00
draft: false
---

So this was a first for me. One of my clients has a MySQL database cluster in Amazon's RDS service and they then store any media related to the data in Amazon S3. During the course of normal operations data gets deleted from both the database and S3 (No sense paying Amazon for hosting data you aren't using). The issue is they now want to maintain that data for potential machine learning purposes in the future.

So I was tasked with coming up with a way to get a "live" mirror of the database and media offsite and have it never delete data. Updating is fine but we want to retain all data forever.

I also wanted to do this as cheaply as possible :)

So here is what I have set up. I purchased a QNAP TS-451D2 along with five 8TB Western Digital Red drives (I always buy a spare) and set up the QNAP as a RAID 6.

After creating the volumes, setting up the network, etc. I opened the Control Panel and under the section called "Applications" I turned on the SQL server.

Next I opened the QNAP app store and installed Hybrid Backup Sync (HBS 3) and PHPMyAdmin.


### Setting up the backup of S3

To set up the backup from S3 I logged into our AWS account and created a new IAM used with full rights to the bucket I needed to mirror.

Back on the QNAP I opened Hybrid Backup Sync and configured a Sync job (with an action set to "Copy") that will run every few hours to copy any new files onto a shared folder on the QNAP.

To be honest, this was the easy part. After a few hours (and a few hundred dollars of AWS fees) we had a mirror of the terabytes of data from S3.

### Setting up the database mirror


I opened PHPMyAdmin from the QNAP and created a new database and a user that had full rights to the new database.

Next I opened the RDS menu in the AWS console and followed the instructions [here](https://aws.amazon.com/premiumsupport/knowledge-center/rds-mysql-logs/) to enable the General Log on my cluster. For prosterity here are the steps from that page:

> Create a DB parameter group
>    1. Open the Amazon RDS console, and then choose Parameter groups from the navigation pane.
>    2. Choose Create parameter group.
>    3. From the Parameter group family drop-down list, choose a DB parameter group family.
>    4. For Type, choose DB Parameter Group.
>    5. Enter the name in the Group name field.
>    6. Enter a description in the Description field.
>    7. Choose Create.
>
> Modify the new parameter group
>    1. Open the Amazon RDS console, and then choose Parameter groups from the navigation pane.
>    2. Choose the parameter group that you want to modify.
>    3. Choose Parameter group actions, and then choose Edit.
>    4. Choose Edit parameters, and set the following parameters to these values:
>
>        General_log = 1 (default value is 0 or no logging)
>
>        Slow_query_log = 1 (default value is 0 or no logging)
>
>        Long_query_time = 2 (to log queries that run longer than two seconds)
>
>        log_output = FILE (writes both the general and the slow query logs to the file system, and allows viewing of logs from the Amazon RDS console)
>
>        log_output =TABLE (writes the queries to a table so you can view these logs with a query)
>    5. Choose Save Changes.
>
>        Note: You can't modify the parameter settings of a default DB parameter group. You can modify the parameter in a custom DB parameter group if Is Modifiable is set to true.
>
> Associate the instance with the DB parameter group
>    1. Open the Amazon RDS console, and then choose Databases from the navigation pane.
>    2. Choose the instance that you want to associate with the DB parameter group, and then choose Modify.
>    3. From the Database options section, choose the DB parameter group that you want to associate with the DB instance.
>    4. Choose Continue.
>
>        Note: The parameter group name changes and applies immediately, but parameter group isn't applied until you manually reboot the instance. There is a momentary outage when you reboot a DB instance, and the instance status displays as rebooting.


Next I had to create two scrips. One that would be located on one of our EC2 servers that would pull data from the General Log and display that data in JSON format. The second being a script that would sit on the QNAP's Web folder and hit the first script and then parse the returned data into the QNAP's database.

Here are versions of the two scripts with the credentials removed:

https://git.umycode.com/dave/Data_Warehouse


They are nothing crazy and likely could be optimized further, but they are currently working fine for me.

In the file that runs on the QNAP (warehouse_get.php) you will notice I have it send a message to my Matrix server (using the [incoming webhook plugin](https://git.umycode.com/dave/maubot-incoming_webhook)) with the amount of queries it ran. This is so I can monitor if the script ran successfully. Also the last thing the EC2 script does before returning the data is rotate the General Log so the QNAP script also dumps a copy of the JSON it received for any needed debugging afterwards.

The next step was to get PHP set up to my liking on the QNAP. By default the QNAP does not log access errors so I followed the steps on [this page](https://technedigitale.com/archives/407) to enable them. For prosterity here are the steps from that page:


> 1. Login to the QNAP device through SSH
> 2. Enable .htaccess usage on Apache. To do this you need to create a new Apache configuration file:
>
>    `vi /etc/config/apache/extra/apache-myconfig.conf`
>
>    And add the following commands:
>
>        `CustomLog logs/main_log combined`
>
>        `ErrorLog logs/error_log`
>
>        `LogLevel info`
> 3. Reference this new configuration file on Apache main configuration file. To do this, edit Apache configuration file:
>
>    `vi /etc/config/apache/apache.conf`
>
>    and add the following line at the end of the file:
>
>    `Include /etc/config/apache/extra/apache-myconfig.conf`
>
> Finally, restart Apache.
>
> `/etc/init.d/Qthttpd.sh restart`
>
> Now, this own’t work on QNAP running QTS 4.1.x onwards, as people from QNAP thought it was working far too good, and decided to through a challenge. For some reason, Apache configuration files are reset every time Apache is restarted by Qnap startup scripts. So until I get a stable solution, the workaround is to manually restart Apache:
>
> `/usr/local/apache/bin/apachectl restart`
>
> The files will be present on /mnt/ext/opt/apache/logs/ .
>
> Please note, all files will be restarted at 0:00, in order not to fill the partion .


The part they don't mention in the above steps is you will also need to adjust the QNAP's php.ini to enable logging too. That can be done by opening the Control Panel app, clicking on "Web Server" in the Applications section, and then scrolling down the bottom of the page and clicking on the "edit" button. Set `log_errors = On`

Next up was to get that script on the QNAP to be run daily (in the future I may run it more frequently but then that brings up some other issues I will mention at the end of this post) by adding it to the QNAP's cron. This has to be done via SSH'ing into the QNAP with your admin account. Once you have shell access open the crontab via the following command:

 `vi /etc/config/crontab`

 and add the following:

`0 1 * * * /mnt/ext/opt/apache/bin/php /share/CACHEDEV1_DATA/Web/warehouse_get.php apiKey=cC7d2DHDnEwJTwiHLakhyjZGKLyWR`

 obviously change the apiKey value to match what is in your own warehouse_get.php file. I am also not 100% sure if the path to the Web directory is the same across all QNAPs so I would suggest trying to run that command outside of the cron to make sure it completes successfully.

Once you save and exit the crontab file you will need to run the following command to reload the file:

`crontab /etc/config/crontab && /etc/init.d/crond.sh restart`

The next step was that I had to grab a copy of the most recent database backup. We use mysqldump along with [bup](https://github.com/bup/bup) to take hourly backups so one evening (I wanted to do this when there was nobody adding/editing records on RDS) I grabbed the latest backup file and inserted it into the QNAP. I then logged into MySQL on RDS and rotated the General Log a few times to make sure I wouldn't be duplicating entries when the script ran.

And that is it :)

The setup has been running for a few weeks now and performing really well. The one thing I will have to tweak at some point in the future is I would like to have the code run more frequently than once a day. Right now I run it in the middle of the night as that is when no one should be on the platform. The reason being is that there is a small window between when the EC2 script generates the JSON and runs the command to rotate the General Log. If a query should come in between those two commands I would lose it.

I need a way to not only grab all queries in the General Log but also any queries from the backup General Log that I do not currently have. That is where that timestamp parameter is supposed to come into play. If I eventually get that implemented I will post an update.