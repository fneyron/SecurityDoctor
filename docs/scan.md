# NMAP

## Scan d'un ensemble de serveurs

	nmap -sn 192.168.1.0/24
	
	Nmap scan report for 10.0.2.2
	Host is up (0.00095s latency).
	MAC Address: 52:54:00:12:35:02 (QEMU virtual NIC)
	Nmap scan report for 10.0.2.3
	Host is up (0.00095s latency).
	MAC Address: 52:54:00:12:35:03 (QEMU virtual NIC)
	Nmap scan report for 10.0.2.4
	Host is up (0.00091s latency).
	MAC Address: 52:54:00:12:35:04 (QEMU virtual NIC)
	Nmap scan report for 10.0.2.15
	Host is up.

## Scan d'une IP en particulier

	nmap 102.176.178.22 
	nmap -top 50 102.176.178.22
	
	Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-27 16:17 CET
	Nmap scan report for 102.176.178.22
	Host is up (0.0076s latency).
	Not shown: 966 filtered ports, 33 closed ports
	PORT     STATE SERVICE
	1723/tcp open  pptp

## Analyse de version de service derrière un port
	nmap 102.176.178.22 -p 1723 -sV
	
	Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-27 16:34 CET
	Nmap scan report for 102.176.178.22
	Host is up (0.037s latency).
	
	PORT     STATE SERVICE VERSION
	1723/tcp open  pptp    MikroTik (Firmware: 1)
	Service Info: Host: MikroTik



