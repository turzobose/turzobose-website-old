---
layout: post
title: Share WiFi through Ethernet Port with Raspberry Pi
author: Turzo Bose
tags: RaspberryPi DNS_Server
subtitle: DNS Server on Raspberry Pi
category: blogpost
toc: true
toc_sticky: true
#icon: fa-connectdevelop
#icon-style: regular
---

Imagine your WiFi onboard your laptop broke down and you cannot use the internet even though the router and modem works fine. Or maybe you have a desktop computer without a WiFi adapter or dongle. How can you hack through this situation without having to invest in a dongle in the quickest possible way?

You can easily do this if you have a Raspberry Pi and a spare ethernet cable lying around. You can hack to serve a DNS server on your Pi, which will act a gateway between your router and your device. The WiFi onboard the Raspberry Pi will communicate with your router and in turn transmit packages to your device.

So, how do we do this? I will assume you have the wlan connection on your Raspberry Pi already. If not, see here to get that done(link to other post). Lets get started:

**Step 1:** Before we get started with installing and setting up our packages, we will first run an update on the Raspberry Pi by entering the following two commands into the terminal

`sudo apt-get update && apt-get upgrade`

**Step 2:** Then we will install dnsmasq, which is the only package we will be using that will act as both the DHCP and DNS server
 
`sudo apt-get install dnsmasq`

>Note: At this point, we should have the wlan0 connection set up and running already. If not, select your preferred WiFi network or add your network details to the wpa_supplicant.conf file
  
**Step 3:** With the wlan0 running, we would now want to set up the eth0 interface and force it to use a static IP address every time. 

We do that by running the following command

`sudo nano /etc/dhcpcd.conf`

Within this file, we need to change the following so that the Rapsberry Pi assigns the same static IP everytime when the ethernet cable is connected

{% highlight python %}
interface eth0
static ip_address=192.168.220.1/24
static routers=192.168.220.0
{% endhighlight %}

Save the file by pressing Ctrl X then Y and Enter

> Most Wifi routers use what's called a Private Network and set the IP range to something similar to:
> `192.168.1.1`
> 
> So remember to chose an address that won't interfere with the routers ability to assign addresses to the ethernet adapter

**Step 4:** With our changes to the DHCPCD configuration, we should now restart the service using

`sudo service dhcpcd restart`

**Step 5:** Before we start changing the dnsmasq configuration file, we should make a copy of the original

`sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`

**Step 6:** Now we can safely tinker with the configurations by creating a new configuration file

`sudo nano /etc/dnsmasq.conf`

We have to add these lines to the file which basically tells the dnsmasq package how to handle DNS and DHCP traffic.

{% highlight python %}
interface=eth0 #Use interface eth0

listen-address=192.168.220.1 #Specify the address to listen on

bind-interfaces #Bind to the interface

server=8.8.8.8 #Use Google DNS

domain-needed #Don't forward short names

bogus-priv #Drop the non-routed address spaces

dhcp-range=192.168.220.50,192.168.220.150,12h # IP range and lease time
{% endhighlight %}

Save and quit this file by pressing Ctrl X then Y and Enter

**Step 7:** Then, we need to configure the Raspberry Pi's firewall so that it forwards all traffic from our eth0 connection to the wlan0 connection. 

Before we do this we must first enable ipv4 IP forwarding through the sysctl.conf configuration file, so let’s begin editing it with the following command

`sudo nano /etc/sysctl.conf`

Find this line 

`#net.ipv4.ip_forward=1`

and uncomment it to:

`net.ipv4.ip_forward=1`

Save and quit this file by pressing Ctrl X then Y and Enter

**Step 8:** We run the following command to enable the IPv4 forwarding immediately and not having to wait until the next reboot

`sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"`

**Step 9:** With our IPv4 Forwarding enabled, we can reconfigure our firewall so that traffic is forwarded from our eth0 interface over to our wlan0 connection. 

>**Use iptables to configure a NAT setting to share the WiFi connection with the ethernet port**
>
>NAT stands for Network Address Translation. This allows a single IP address to server as a router on a network. So in this case the ethernet adapter on the RPi will serve as the router for whatever device you attach to it. The NAT settings will route the ethernet requests through the Wifi connection.
Run the following commands to add our new rules to the iptable:

Basically this means that anyone connecting to the ethernet will be able to utilise our wlan0 internet connection

{% highlight python %}
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
{% endhighlight  %}

>Note: If you get errors when entering the above lines simply reboot the Pi using sudo reboot.

**Step 10:** On every boot of the Raspberry Pi, the iptables are flushed. So we need a way to save our new rules somewhere so they are loaded back in on every boot

To save our new set of rules run the following command

`sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`

Now with our new rules safely saved somewhere we need to make this file be loaded back in on every reboot. The most simple way to handle this is to modify the rc.local file by modifying the rc.local file

`sudo nano /etc/rc.local`

Find `exit 0`

and add this line above `exit 0`:

`iptables-restore < /etc/iptables.ipv4.nat`

Save and quit this file by pressing Ctrl X then Y and Enter

**Step 11:** Finally, run this command to start your dnsmasq services

`sudo systemctl start dnsmasq`

Now you should finally have a fully operational Raspberry Pi WiFi Bridge, you can ensure this is working by plugging any device into its Ethernet port, the bridge should provide an internet connection to the device you plug it into.

To ensure everything will run smoothly, it’s best to try rebooting now. This will ensure that everything will successfully re-enable when the Raspberry Pi is restarted. Run the following command to reboot the Raspberry Pi:

`sudo reboot`

#### Happy hacking!!
