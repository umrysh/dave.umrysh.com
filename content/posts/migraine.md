---
title: "Migraine"
date: 2019-02-14T20:38:51-07:00
draft: false
---
I woke up this morning with a migraine and it would not go away. It has been almost a year since I have had such pain and light sensitivity which must mean it is caused by something seasonal. In the past I would blame a sudden change in humidity as that has usually accompanied my migraines but this year we have been entrenched in a cold snap for weeks so.... ¯\\_(ツ)_/¯

I decided to take the afternoon off as any code I write with a cloudy head just means I have more bugs to find and fix later. This meant I was able to catch up on Raw and Smackdown from this week and there were two fantastic matches I need to talk about. The Raw tag team titles changed hands in a bout I would say is a potential match of the year contender. The Revival, Bobby Roode and Chad Gable put on an incredible showing. Great spots and they timed the pace of the match perfectly. 

The other match that stood out was the gauntlet match on Smackdown. The reason it stood out was due to the fantastic chemistry Kofi had with Daniel Bryan, Jeff Hardy, Samoa Joe and AJ. Kofi was a last minute addition to the match due to an injury, but you would never know it. Plus they let Kofi work for over an hour and defeat Bryan, Hardy, and Joe. The back and forth between him and AJ before they started was done so well. Having two faces battle can always be tough as who are the fans supposed to cheer? But the way AJ and Kofi put that scene together worked in such a way you could cheer both of them and it still fit well into both of their over arching character development. I don't think this match will be in the running for match of the year, but it will definitely bring Kofi back onto the map for a lot of fans. Maybe we will even see him get a much deserved push in 2019.

In non WWE news I did get some time to add in a new feature to yet another project I have been working on. I am building a PLC monitoring device that logs register data so that clients can log into a web portal and view live sensor data in their production systems. I am writing the device code to be fully automated and configurable from the web portal. You just install one of these devices (did I mention they have SIM cards so no need to hook them into your office network) the device automatically connects to my server and registers itself. I then assign it to the customer's account and they can configure all the data logging for the device within the web portal. 

I have a beta device currently running in a client's manufacturing plant and it has been working quite well. The code I was adding tonight was the ability for it to not only poll on a time interval and log what was in the registers but for it to log batches. So the program now has the option of polling the PLC every 15 seconds and looking for a decimal register to be set to 1. If it finds a 1 it reads the rest of the registers and sends it off to the server. Lastly, it resets that decimal register to zero. This way we can also program the PLC to check if batch data has been read or not. Cool, right?

And if you are reading this and are interested in the PLC logging platform (I'm calling it [Modlog](https://modlog.ca)) for your own manufacturing process, please send me an email. I'm currently offering the service for free until I have the web portal all nice and polished



