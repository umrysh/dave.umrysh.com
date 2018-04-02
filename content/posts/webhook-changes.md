---
title: "Webhook Changes"
date: 2018-04-02T12:34:19-06:00
draft: false
---
After sleeping on it I thought it best to revamp the Github webhook that deploys this weblog. There is just something about allowing PHP to execute git commands and deploy to web roots that seems really bad. The original example I  had followed to set up the webhook may have worked well for the  author but giving PHP that much power really bugged me. I actually had trouble sleeping last night just ruminating about the current set up. #NerdDreams

After a bit of tweaking the  webhook now simply updates a flag in a database when there is a new version of the website ready to be deployed.  I then have a cron job run every 5 minutes that calls a Python script which checks if the flag is set and will then deploy the new website. Much better :)