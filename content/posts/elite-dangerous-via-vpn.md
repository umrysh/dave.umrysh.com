---
title: "Elite Dangerous via Vpn"
date: 2020-10-24T10:08:40-06:00
draft: false
---

I am a huge fan of the game [Elite Dangerous](https://en.wikipedia.org/wiki/Elite_Dangerous). As a testament I bought my Xbox One for the sole reason of playing Elite from the comfort of my couch. So it was really frustrating when about a week ago I started getting "Taupe Cobra" and "Gold Cobra" errors every time I tried to jump into and out of hyperspace. I tried all of the suggestions on the Frontier support sites and was about ready to give up when I stumbled across this post:

[https://forums.frontier.co.uk/threads/bug-taupe-gold-cobra.556502/](https://forums.frontier.co.uk/threads/bug-taupe-gold-cobra.556502/)

The gist of the forum post is:


> Fdev pay AMAZON for the servers farms that are booting you out of game, If the amazon server farm that fdev pay amazon for sucks in the USA that you connect to but all the rest work perfectly fine in EU SA NZ AU.. then only the people who connect to that server in that specific state at the time you want to play will connect to that server... could get errors... every body else can have a fine game...
You can play Elite dangerous with nealy every thing broken on PSN because ITs not running on PSN servers they have they own independant server farms rented from Amazon Hosting...



This got me thinking, could this really be the issue? I have not changed anything in my home network to cause the issues and I can see there are [European streamers](https://www.twitch.tv/videos/778344653) with no issues. Could the issue be that Frontier has Elite servers near my geographical location that are just being junk, or that there is some network hardware between my home and those servers that has become flaky enough for my game to be crashing?

The only way to know for sure is to VPN my Xbox Internet connection to Europe and see if the problems go away :)

The tricky part is that the Xbox does not natively support VPN connections so I looked through my random computer closet (everyone has one of these right?) and found an old Rock64 single board computer and an old TP-Link WiFi usb dongle.

![Rock64](/img/rock64.jpg)

I flashed the Rock64 with a version of Ubuntu 18.4 onto an SD card, installed the necessary drivers to get the TP-Link dongle to work and then set to work converting this box into a WiFi hotspot that routes all traffic via VPN to Sweden.


I first installed hostapd so I can turn my WiFi dongle into a hotspot:

`sudo apt-get install hostapd`

Next I added a config file at /etc/hostapd/hostapd.conf with the following:

```
interface=wlx002586e926dd
ssid=xbox
hw_mode=g
channel=6
auth_algs=1
macaddr_acl=0
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

To find out the name of my WiFi interface I just ran:

`ip addr`

I then needed a DHCP server running for when my xbox connects to the WiFi:

`sudo apt-get install isc-dhcp-server`

As with hostapd I needed to add a config file for the DHCP sever at /etc/dhcp/dhcpd.conf

```
subnet 10.10.0.0 netmask 255.255.255.0 {
range 10.10.0.25 10.10.0.50;
option domain-name-servers 8.8.4.4;
option routers 10.10.0.1;
}
```

and then edit the file /etc/default/isc-dhcp-server and specify the WiFi interface:

`INTERFACES="wlx002586e926dd"`

Now onto the VPN connection itself. I use Express VPN so I header to their OpenVPN page and download the ovpn file for Sweden:

[https://www.expressvpn.com/setup#manual](https://www.expressvpn.com/setup#manual)

While on the same page I made note of my username and password, as I will need those shortly.

I copied the downloaded ovpn file to /home/rock64/.config/expressvpn/express.ovpn

and then created the file /home/rock64/.config/expressvpn/login.conf and put my ExpressVPN username on the first line and my password on the second.

All that was left was to create a file called start_xbox.sh with the following:

```
killall wpa_supplicant &
sleep 10s &&
ifconfig wlx002586e926dd 10.10.0.1 netmask 255.255.255.0 &&
systemctl start isc-dhcp-server &&
/usr/sbin/openvpn --config /home/rock64/.config/expressvpn/express.ovpn --auth-user-pass /home/rock64/.config/expressvpn/login.conf &
sleep 10s &&
echo "1" | tee /proc/sys/net/ipv4/ip_forward &&
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE &&
iptables -A FORWARD -o tun0 -i wlx002586e926dd -m conntrack --ctstate NEW -j ACCEPT &&
hostapd -dd /etc/hostapd/hostapd.conf 1>/dev/null &
```

After rebooting the Rock64 I SSH'd into it and ran:

`sudo ./start_xbox.sh`

After a few seconds the WiFi network "xbox" was available. I connected my phone to it and looked up my [IP address](https://whatthefuckismyip.com/) and I was in Sweden :)

So now came the test. I connected my Xbox to the new WiFi network and booted up Elite...........It worked like a dream!!!!!

I played for an hour with no errors whatsoever! A little lag here and there which is expected when running a game over a VPN, but absolutely playable.

#Success

![AridNevada](/img/elite.png)