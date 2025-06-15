# Wireless2Wired_Tethering
Tethering Wireless Ubuntu to Wired Debian system.
Some notes pertaining to a method used to allow tethering via a hotspot (Wi-Fi) to a shared wired ethernet connection on Ubuntu/ Debian.

*Disclaimer: These are personal notes to allow a wirelessly tethered machine to be connected to a wired system to allow internet access on the wired network. I am not responsible for any issues that you may encounter from trying to follow these instructions.*

Firstly, setup a Wi-Fi tethering hotspot from your phone or MiFi device. 

Click on the Wi-Fi network with the name of your newly created hotspot and type in the applicable PSK for your hotspot network.

Wireless machine: 
Wireless > MyNewAP click on the broadcasted SSID and type in the PSK you created.

Open network connections on your machine with wireless. Go to the ethernet port network on your wireless machine to be a shared port.

network connections > Ethernet (eth0) or netplan- > IPv4 Settings >  Shared to other computers.

You “should” get an address in the 10.42.0.* range eg 10.42.0.42 auto assigned from the wireless connected machine on the wired machine, I found that setting a DNS server to the shared connection IP helped with the initial setup e.g 10.42.0.1

Note: On the wireless machine you will also need to allow ports for DHCP and DNS to go through your firewall and allow forwarding from the wireless interface to the ethernet interface.

```
sudo ufw route allow in on wlan0 out on eth0
sudo ufw allow bootps comment 'Allow 67/UDP'
sudo ufw allow bootpc comment 'Allow 68/UDP'
sudo ufw allow 53/udp comment 'Allow DNS_53/UDP'
sudo ufw allow 53/tcp comment 'Allow DNS_53/TCP'
```

Open a terminal and check with ip addr or ifconfig on the wirelessly connected machine you should see three interfaces.

For instance: 
```
lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo

wlan0  <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 
    link/ether 00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.X.X.X brd 10.X.X.X  scope global noprefixroute wlan0
       valid_lft forever preferred_lft forever

eth0 <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 
    link/ether 00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.1/24 brd 10.42.0.255  scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
```
The parts your interested in are on the eth0 interface. This is your ethernet port.
 
Go to your wired connected machine and launch a terminal and run the same command ```ip addr``` output should look something like the below code.
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
5: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.35/24 brd 10.42.0.255 scope global noprefixroute eth0
       valid_lft forever preferred_lft forever
```
You can see here that your ip address is 10.42.0.35 on a 24 bit subnet.

Open network connections select the ethernet connection.

Set the DNS servers to 10.42.0.1 or the corresponding address on your wireless machine.

Try to ping the wireless machine 

Open a terminal > ```ping 10.42.0.1```
```
ping 10.42.0.1
PING 10.42.0.1 (10.42.0.1) 56(84) bytes of data.
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=0.892 ms
64 bytes from 10.42.0.1: icmp_seq=2 ttl=64 time=1.08 ms
64 bytes from 10.42.0.1: icmp_seq=3 ttl=64 time=0.955 ms
64 bytes from 10.42.0.1: icmp_seq=4 ttl=64 time=0.889 ms
```
Now try to ping googles name servers using the same command from an open terminal. 

```ping google.com
PING google.com (142.250.187.238) 56(84) bytes of data.
64 bytes from lhr25s34-in-f14.1e100.net (142.250.187.238): icmp_seq=1 ttl=109 time=32.0 ms
64 bytes from lhr25s34-in-f14.1e100.net (142.250.187.238): icmp_seq=2 ttl=109 time=268 ms
```
**Great! You should now have connectivity to the internet.**

*If you wish to you can also install AdGuard adblocker from the snap store on the wirelessly connected machine details below. Thanks to the the AdGuard home team. Donate if you can.*

```sudo snap install adguard-home```

After installing the snap packages on the wireless machine goto 127.0.0.1:3000 and set the interfaces to all, unplug the ethernet from the wired machine to allow adguard to access the port.

Click next and finish and reconnect your wired connection. 

You should be prompted to create a password to login to AdGuard home. Keep this handy as you will need it later.

Check the ping to the wireless machine from the wired machine. 

Open terminal: ```ping 10.42.0.1```
```
 PING 10.42.0.1 (10.42.0.1) 56(84) bytes of data.
64 bytes from 10.42.0.1: icmp_seq=1 ttl=64 time=1.07 ms
64 bytes from 10.42.0.1: icmp_seq=2 ttl=64 time=0.947 ms
64 bytes from 10.42.0.1: icmp_seq=3 ttl=64 time=1.16 ms
```
Browse to a couple of websites on your wired machine ensuring that you still have internet connectivity on your wired connection.

So now try to open adguards dashboard on your wireless machine. 

Web browser: http://127.0.0.1 you should see queries coming in from the wired machine. 

From here you can enable the DHCP server on adguard and set a static IP on the MAC address for the wired machine. 

Open a web browser on your wired machine: goto http://10.42.0.1 type in the username and password created eariler.

Settings > DHCP settings > Add static address: 10.42.0.35  take the MAC address from the wired machine. 

Wired machine > open terminal > ```ip addr``` 
```
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether XX:XX:XX:XX:XX:XX brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.35/24 brd 10.42.0.255 scope global noprefixroute eth0
```
Copy your link/ether address and paste it into your DHCP leases along with the address you would like to statically set the wired machines address to.

Optional: set static address on wired machine.

Network connections > ethernet connection > ipv4 settings > manual 
```
Address     	Netmask	Gateway
10.42.0.35 	24 		10.42.0.1

DNS Servers:
10.42.0.1
```
That’s it! You should be able to now browse the internet on both your wired and wirelessly connected machines via hotspot with AdBlocking included. 
