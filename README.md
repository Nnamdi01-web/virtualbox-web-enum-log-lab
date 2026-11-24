# VirtualBox Lab: Host-Only Recon, Web Enumeration & Apache Log Analysis

## Overview
A hands-on local security lab in Oracle VirtualBox connecting a **Kali Linux** testing VM (`192.168.56.104`) to an **Ubuntu** target VM (`192.168.56.101`) running **Apache** over a Host-Only network (`192.168.56.0/24`). The objective was to verify network reachability, perform service discovery, probe HTTP headers, run directory enumeration, and correlate all activity inside Apache's server access logs.

## Lab Environment
- **Hypervisor:** Oracle VirtualBox (Host-Only Network `192.168.56.0/24`)
- **Target VM:** Ubuntu (`192.168.56.101`), running Apache HTTP Server `2.4.58`
- **Testing VM:** Kali Linux (`192.168.56.104`)

---

## Steps & Results

### 1. Connectivity Check (`ping`)
Tested network reachability from Kali to Ubuntu (`192.168.56.101`):
```bash
ping -c 4 192.168.56.101

Result:
PING 192.168.56.101 (192.168.56.101) 56(84) bytes of data.
64 bytes from 192.168.56.101: icmp_seq=1 ttl=64 time=5.17 ms
64 bytes from 192.168.56.101: icmp_seq=2 ttl=64 time=2.55 ms
64 bytes from 192.168.56.101: icmp_seq=3 ttl=64 time=2.66 ms
64 bytes from 192.168.56.101: icmp_seq=4 ttl=64 time=2.92 ms

--- 192.168.56.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 2.547/3.324/5.173/1.075 ms


2. Service & Version Discovery (nmap)
Scanned 192.168.56.101 from Kali to identify open ports and running services:
nmap -sV 192.168.56.101

Result:
Nmap scan report for 192.168.56.101
Host is up (0.0041s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
MAC Address: 08:00:27:F3:5A:3E (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

3. HTTP Header Inspection (curl)
Sent an HTTP HEAD request from Kali to inspect server headers:
curl -I [http://192.168.56.101](http://192.168.56.101)

Result:
HTTP/1.1 200 OK
Date: Mon, 23 Nov 2026 14:47:57 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Tue, 24 Feb 2026 13:49:57 GMT
ETag: "29af-64b9229baddf2"
Accept-Ranges: bytes
Content-Length: 10671
Vary: Accept-Encoding
Content-Type: text/html

4. Directory Enumeration (gobuster)
Created a small test wordlist on Kali and enumerated web paths:
printf "index.html\nadmin\nrobots.txt\nserver-status\n" > ~/testlist.txt
gobuster dir -u [http://192.168.56.101](http://192.168.56.101) -w ~/testlist.txt -t 5

Result:
[+] Url:                     [http://192.168.56.101](http://192.168.56.101)
[+] Method:                  GET
[+] Threads:                 5
[+] Wordlist:                /home/angelika/testlist.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8

/index.html           (Status: 200) [Size: 10671]
/server-status        (Status: 403) [Size: 279]
Progress: 4 / 4 (100.00%)
Finished

5. Log Correlation on Ubuntu (/var/log/apache2/access.log)
Inspected Apache's access log on Ubuntu to confirm all incoming requests from Kali (192.168.56.104):
sudo tail -n 10 /var/log/apache2/access.log

Result:
192.168.56.104 - - [23/Nov/2026:15:43:35 +0100] "GET /HNAP1 HTTP/1.1" 404 456 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; [https://nmap.org/book/nse.html](https://nmap.org/book/nse.html))"
192.168.56.104 - - [23/Nov/2026:15:47:57 +0100] "HEAD / HTTP/1.1" 200 255 "-" "curl/8.15.0"
192.168.56.104 - - [23/Nov/2026:16:24:58 +0100] "GET / HTTP/1.1" 200 3404 "-" "gobuster/3.8"
192.168.56.104 - - [23/Nov/2026:16:24:58 +0100] "GET /admin HTTP/1.1" 404 437 "-" "gobuster/3.8"
192.168.56.104 - - [23/Nov/2026:16:24:58 +0100] "GET /robots.txt HTTP/1.1" 404 437 "-" "gobuster/3.8"
192.168.56.104 - - [23/Nov/2026:16:24:58 +0100] "GET /index.html HTTP/1.1" 200 3404 "-" "gobuster/3.8"
192.168.56.104 - - [23/Nov/2026:16:24:58 +0100] "GET /server-status HTTP/1.1" 403 440 "-" "gobuster/3.8"


<img width="1200" height="896" alt="ubuntu-ping-ss png" src="https://github.com/user-attachments/assets/f6b6d575-35dd-4945-ab21-353416f7ea0c" />
<img width="1200" height="896" alt="ubuntu-access-log png" src="https://github.com/user-attachments/assets/daa1c11e-1d91-4c49-9f52-7f7bf5eb6c99" />
<img width="1200" height="896" alt="gobuster v3 8" src="https://github.com/user-attachments/assets/16b0bdca-1410-42e4-9671-6f69043087d0" />



