---
title: "Mastodon category search"
date: 2019-07-26T08:29:37-06:00
draft: false
---

A few months ago I decided to try out [Mastodon](https://joinmastodon.org) again. I had previously [posted](https://dave.umry.sh/posts/new-blog/) about my main issue with a federated social network being that I find it very similar to when free email services were just coming out in the 90's. You would sign up for an address like `cool.dude@freemailxyz.com` and then a year later `freemailxyz.com` would shut down. You would lose all your email and all your friends could no longer contact you as they only knew you as `cool.dude@freemailxyz.com`

I feel we will eventually get some very large Mastodon instances with years of uptime which will in turn build confidence for those wanting a quick and easy way to get onto the Fediverse. A `gmail.com` for the Fediverse, if you will. Until that day comes the only practical solution at the moment appears to be to self-host your own Mastodon instance under your own domain. The obvious issue with self-hosting is that you are now responsible with keeping your own instance up to date as well as the costs of the domain and the server.

But in the spirit of trying new things I decided to give it a go. I used a sub-domain of my [company's main domain](https://umycode.com) to save cost and spun up a new droplet on [Digital Ocean](https://m.do.co/c/e89b309736d5) to host it. So my total cost is about $6 a month, which is not too bad for an ad free social media experience.

Next came the second biggest issue I have with the Fediverse, how to find people to follow? Since I'm running my own instance I need to actively go out and find people to follow as the public timeline is practically useless for a self-hosted instance.

So I decided to build myself a crude [Python](https://www.python.org) script that would scrape [https://fediverse.network/mastodon](https://fediverse.network/mastodon) for a list of instances and then attempt to read each of their "`/explore`" pages for the keywords their users have tagged themselves with. It took a couple hours to scan the Fediverse and store all the information into a [SQLite](https://sqlite.org) database. Some domains have the "`/explore`" page disabled while others are behind [CloudFlare](https://www.cloudflare.com) which blocks my script. Even so it was able to get a pretty good list of categories and accounts.

Now that I have the data in an SQLite database I can use the script to search for categories I want to follow and get a list of accounts that have tagged themselves with that category, from anywhere on the Fediverse.

For those interested here is a link to the script: [masto_search.py](https://gist.github.com/umrysh/483f37e4bb8d0f573e10879df5da84b3)

Hopefully Mastodon itself will someday enable a Fediverse spanning search feature, but until then this script will have to do.

>If you are on the Fediverse come say hi. You can find me at [https://fed.umycode.com/@dave](https://fed.umycode.com/@dave)