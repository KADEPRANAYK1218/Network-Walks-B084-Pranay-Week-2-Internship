# 🔐 Penetration Testing Report: Footprinting & Network Scanning Phases

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| | |
|---|---|
| **Pentester Name (Cybersecurity Professional)** | K. Pranay Kumar |
| **Program/Batch** | B084-Networkwalks |
| **Date** | 13 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools)<br>W2-PM4 (theHarvester)<br>W2-PM5 (Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks (secured written permission already)<br>2. microsoft.com (public OSINT sources only)<br>3. My own local LAN Network |
| **Permission secured from client?** | Yes |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting<br>**Phase 2:** Scanning & Network Discovery<br>**Phase 3-5:** In Progress |

## 📑 Table of Contents
1. [Liability Disclaimer](#1-liability-disclaimer)
2. [Introduction](#2-introduction)
3. [Tools Used](#3-tools-used)
4. [Activities Performed](#4-activities-performed)
5. [Risk Analysis / Impact](#5-risk-analysis--impact)
6. [Recommendations](#6-recommendations)
7. [Conclusion](#7-conclusion)
8. [Evidences Collected](#8-evidences-collected)

---

## 1. Liability Disclaimer

I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself. All these materials are for education and research purpose only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

## 2. Introduction

This report explains what I did in three practical tasks from Week 2 of my Cybersecurity internship at Networkwalks. In **W2-PM1** I collected public information about the website networkwalks.com using different Kali Linux tools. In **W2-PM4** I used a tool called theHarvester to find public information (emails, subdomains and IP addresses) about microsoft.com. In **W2-PM5** I used Zenmap to scan my own home network and see which devices are connected to it.

Together, these tasks show how an attacker usually starts: first collect as much public information as possible (footprinting), then look at a network to find live devices and open ports (scanning). For every step I have written the command I used, what I saw, why it matters, and added a screenshot as proof in [section 8](#8-evidences-collected).

## 3. Tools Used

| Tool | What it does |
|---|---|
| Kali Linux & Windows | The two operating systems I worked on (Kali for footprinting, Windows for the network scan). |
| WHOIS | Shows who registered a domain, when, and which name servers it uses. |
| whatweb | Tells what technology a website is built with (server, CMS, plugins). |
| nslookup | Finds the IP address that belongs to a domain name. |
| curl -I | Reads the hidden "header" information a website sends back. |
| wafw00f | Checks whether a website is protected by a firewall (WAF). |
| dnsrecon | Lists the DNS records of a domain (name servers, mail servers, SPF, etc.). |
| theHarvester | Collects emails, subdomains, IP addresses and ASNs of a domain from public sources. |
| Zenmap (Nmap GUI) | Scans my network to find live devices, their IP and MAC addresses and open ports. |
| Windows CMD | Used the `ipconfig /all` command to see my own network settings. |

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance (W2-PM1)

In this task I studied the website `networkwalks.com` using six Kali Linux tools: **WHOIS, WhatWeb, Nslookup, Curl, Wafw00f and DNSRecon**. Each tool answers a different question, and when I put the answers together I got a clear picture of the website and how it is set up.

**a) WHOIS: who owns the domain?**
I ran `whois networkwalks.com`. It showed that the domain is registered with **GoDaddy** and was created on **6 November 2019**. It will expire on **6 November 2027**. The name servers are **NS6135.HOSTGATOR.COM** and **NS6136.HOSTGATOR.COM**. The owner's personal details are hidden behind a privacy service, which is good. I also noticed that **DNSSEC is "unsigned"**, which means the domain does not use that extra DNS protection.

**b) WhatWeb: what is the website built with?**
I ran `whatweb networkwalks.com`. The site first redirects from HTTP to the secure HTTPS version. WhatWeb then showed that it runs on an **Apache** server and uses **WordPress 7.1.4** with the **WordPress Download Manager 3.3.76** plugin. It also found jQuery, Bootstrap, Google Tag Manager, the contact email `info@networkwalks.com` and the server IP **192.232.216.135**.

**c) Nslookup: what is the IP address?**
I ran `nslookup networkwalks.com`. It gave the IP address **192.232.216.135**, the same IP that WhatWeb showed. This confirms that the website is hosted on that server.

**d) Curl: what does the website say in its headers?**
I ran `curl -I https://networkwalks.com`. The website replied with **HTTP/2 200**, which means it is working normally. The headers also told me that the server is Apache, that caching is used, and that the WordPress API address **/wp-json/** exists. A cookie called `__wpdm_client` is set and is protected with the Secure and HttpOnly flags.

**e) Wafw00f: is there a firewall?**
I ran `wafw00f networkwalks.com`. After only two requests it told me that the site is protected by a **ModSecurity (SpiderLabs)** Web Application Firewall. This is a good sign, because the site is not completely open to attacks.

**f) DNSRecon: what do the DNS records say?**
I ran `dnsrecon -d networkwalks.com`. It found 8 records. These include the name servers (**ns6135.hostgator.com** and **ns6136.hostgator.com**), the mail server **mail.networkwalks.com**, an SPF record that controls who may send email for the domain, a Google verification record and several cPanel autodiscover records. It also showed that both name servers run **BIND version 9.16.23-RH**. Showing the software version is not ideal because attackers can use it to look for known weaknesses.

**Summary of PM1 results**

| Command | What I found |
|---|---|
| `whois networkwalks.com` | Registered with GoDaddy; created 06-Nov-2019; expires 06-Nov-2027; HostGator name servers; DNSSEC not enabled |
| `whatweb networkwalks.com` | Apache server, WordPress 7.1.4, WP Download Manager 3.3.76, IP 192.232.216.135 |
| `nslookup networkwalks.com` | IP address 192.232.216.135 |
| `curl -I https://networkwalks.com` | HTTP/2 200 OK; Apache; /wp-json/ endpoint; caching headers; one cookie |
| `wafw00f networkwalks.com` | ModSecurity (SpiderLabs) firewall detected |
| `dnsrecon -d networkwalks.com` | 8 records found; BIND 9.16.23-RH shown on both name servers |

### 4.2 OSINT Gathering with theHarvester (W2-PM4)

OSINT means "open-source intelligence", which is information anyone can find on the internet. In this task I used **theHarvester 4.11.1** in Kali Linux to collect public information about `microsoft.com`. The tool only reads public sources such as search engines and certificate logs, and it does not attack or touch the target, so this activity is passive and safe.

**a) Reading the help menu**
First I ran `theHarvester -h` to understand how the tool works. The three options I needed were `-d` (the domain), `-l` (how many results to collect) and `-b` (which source to use).

**b) Search with only one source (baidu)**
I ran `theHarvester -d microsoft.com -l 1000 -b baidu`. This used only the Baidu search engine, and it found **nothing**: no IPs, no emails, no people and no hosts. So one single source may not be enough, even for a very big company.

**c) Search with all sources**
Then I ran `theHarvester -d microsoft.com -l 50 -b all`, which uses every source. The results were much better. Some sources need a paid or personal API key (for example Censys, Hunter and BuiltWith) and showed a "Missing API key" message, so they were skipped. The free sources worked fine.

The all-source search found the following:

| Item | Result for microsoft.com |
|---|---|
| Hosts found | **9,960** hosts and subdomains (many look like test or pre-production servers) |
| IPs found | **170** IP addresses |
| ASNs found | **8** (AS13335, AS16625, AS206834, AS20940, AS27647, AS398101, AS40034, AS8075) |
| Interesting URLs | **7** (documentation pages and Microsoft login URLs) |
| Emails found | **3** (dotnet-docker-bot@, opencode@ and secure@ microsoft.com) |
| People / LinkedIn | None found |

### 4.3 Network Scanning with Zenmap (W2-PM5)

In this task I used **Zenmap** to look at my own home network. I wanted to know my own IP address, which other devices are connected, what their IP and MAC addresses are, and which ports are open on them.

**a) Finding my own network details**
I opened Command Prompt and ran `ipconfig /all`. The computer name is **KPK** and it is connected through Wi-Fi (Intel Wi-Fi 6 AX201). My IP address is **192.168.1.5** and the subnet mask is **255.255.255.0**, so my network is `192.168.1.0/24`, which can hold 256 addresses. The router is at **192.168.1.1**. In the screenshots I blurred my IP and MAC addresses for privacy.

**b) Ping Scan: which devices are online?**
I typed `192.168.1.0/24` as the target and chose the **Ping scan** profile. Zenmap checked all 256 addresses in 3.31 seconds and found three devices online: the **router** at 192.168.1.1 (made by Zyxel), a device called **I2302** at 192.168.1.2, and my own computer **KPK**.

**c) Quick Scan: which ports are open?**
Next I ran the **Quick scan** profile, which checks the 100 most common ports. It finished in 5.66 seconds and again found three devices, this time including a device called **V2027** at 192.168.1.3. The table below shows what I found.

| IP address | Device name | MAC address | Open ports / notes |
|---|---|---|---|
| 192.168.1.1 | Router / gateway | 30:BD:13:CC:C2:78 (Zyxel) | 53/tcp domain, 443/tcp https |
| 192.168.1.2 | I2302 | 46:FA:D0:FE:8B:A3 (Unknown) | Found in Ping scan only |
| 192.168.1.3 | V2027 | D6:62:BF:AF:5D:00 (Unknown) | All 100 scanned ports are closed |
| 192.168.1.5 | KPK (my PC) | Hidden for privacy | 135/tcp msrpc, 139/tcp netbios-ssn, 445/tcp microsoft-ds |

**d) Network topology**
Finally I opened the **Topology** tab in Zenmap, turned on the legend and saved the map. It shows `localhost` in the middle, connected to four devices: **192.168.1.1**, **I2302 (192.168.1.2)**, **V2027 (192.168.1.3)** and **KPK (192.168.1.5)**.

## 5. Risk Analysis / Impact

From everything I collected, I listed the possible risks below. The level (Critical, High, Medium or Low) shows how much damage the information could cause if someone misused it.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Compromised credentials reported for the domain | theHarvester's Hudson Rock source reported about 621,000 compromised accounts for microsoft.com (a third-party figure that I did not verify) | Stolen passwords can let attackers log in to real accounts, which can lead to a full account or data takeover | 🔴 **Critical** |
| 2 | Large public footprint found through OSINT | theHarvester found 9,960 hosts, 170 IPs, 8 ASNs, 7 URLs and 3 email addresses for microsoft.com | Attackers can map the whole attack surface, find test servers and pick email targets for phishing | 🟠 **High** |
| 3 | Open file-sharing ports on my PC | Zenmap Quick scan found ports 135, 139 and 445 open on KPK | These Windows services are often targeted by attackers on the same network if they are not restricted | 🟠 **High** |
| 4 | Web technology information exposed | WhatWeb identified WordPress 7.1.4 and WP Download Manager 3.3.76 | Attackers can search for known problems in the exact software and versions the site uses | 🟡 **Medium** |
| 5 | DNS information exposed | DNSRecon showed NS, MX, SPF and SRV records and the BIND 9.16.23-RH version | Helps an attacker build a picture of the infrastructure and check the DNS software version | 🟡 **Medium** |
| 6 | Server IP address visible | Nslookup resolved the domain to 192.232.216.135 | Shows where the web server is located on the internet | 🟢 **Low** |
| 7 | HTTP header information exposed | Curl showed the Apache server, caching headers and the /wp-json/ address | Gives small hints that help an attacker identify the website technology | 🟢 **Low** |
| 8 | Firewall type identifiable | Wafw00f identified ModSecurity (SpiderLabs) | Tells an attacker which firewall they would need to get past | 🟢 **Low** |
| 9 | DNSSEC not enabled | WHOIS shows DNSSEC unsigned and DNSRecon got no DNSSEC answer | DNS answers cannot be checked for tampering | 🟢 **Low** |
| 10 | Several devices visible on the local network | Zenmap found four devices, two with an unknown MAC vendor | An unknown or unauthorized device could be connected without the owner noticing | 🟢 **Low** |

**Risk level key:** 🔴 Critical &nbsp; 🟠 High &nbsp; 🟡 Medium &nbsp; 🟢 Low

These are observations from information gathering and scanning, not confirmed vulnerabilities. The levels show how serious each item could be if it were misused, not that something has already been hacked.

In these three modules I did not try to exploit anything or break into any system. Seeing a software version, an IP address, a DNS record or an open port does not mean the system is vulnerable. More testing, with proper permission, would be needed to confirm any real weakness.

## 6. Recommendations

Based on what I found, I suggest the following improvements:

1. **Protect accounts against stolen passwords**: Use strong, unique passwords and turn on multi-factor authentication. Check regularly whether company accounts appear in leaks.
2. **Limit what is visible in public**: Review which subdomains, test servers and email addresses are public, and remove the ones that are not needed.
3. **Close unnecessary open ports**: Block ports such as 135, 139 and 445 with the Windows Firewall unless they are really needed.
4. **Review exposed technology information**: Check regularly what a visitor can learn about the CMS, plugins and server software.
5. **Keep software updated**: Update WordPress, plugins and other software regularly and check security advisories.
6. **Review HTTP headers**: Remove headers that give away technical details that visitors do not need.
7. **Review DNS records and hide the DNS version**: Check DNS records from time to time, and hide the BIND version so it is not shown publicly.
8. **Consider enabling DNSSEC**: DNSSEC helps protect users from fake or changed DNS answers.
9. **Keep the firewall (WAF) switched on and tuned**: ModSecurity already blocks simple attacks, so it should stay enabled and be monitored.
10. **Scan your own network regularly**: Run a network discovery now and then to see which devices are connected.
11. **Investigate unknown devices**: Any device you do not recognise, for example one with an unknown MAC vendor, should be checked.
12. **Keep network documentation**: Keep an up-to-date list or map of devices and the network layout.
13. **Test only with permission**: Only perform reconnaissance and scanning on systems where you have written permission or that you own.

## 7. Conclusion

In Week 2 of my Cybersecurity & Ethical Hacking internship I completed three practicals: footprinting (W2-PM1), OSINT with theHarvester (W2-PM4) and network scanning with Zenmap (W2-PM5).

In the footprinting task I learned what each tool tells us: WHOIS shows who registered a domain, WhatWeb shows the technologies, Nslookup gives the IP address, Curl shows the headers, Wafw00f detects the firewall and DNSRecon lists the DNS records.

With theHarvester I learned that a lot of information about a company is public, such as thousands of hosts, hundreds of IP addresses and some email addresses, and that the results change a lot depending on the source used.

With Zenmap I learned how to find my own network details, discover the devices connected to it, see which ports are open and draw a network map.

These exercises showed me that attackers (and defenders) can learn a lot before any real attack starts. They also showed me that findings must be written down clearly: what was done, what was found, why it matters and how to reduce the risk.

Most importantly, I learned that reconnaissance and scanning must always stay within an authorized scope. All of these activities were done only as part of the educational lab.

## 8. Evidences Collected

### 8.1 W2-PM1: Footprinting with Kali Linux tools

**Figure 1: WHOIS lookup of networkwalks.com (part 1 of 3)**

<img width="1600" height="740" alt="image" src="https://github.com/user-attachments/assets/46fc19be-ce31-4d0e-8714-46cb3b29968a" />


**Figure 2: WHOIS lookup of networkwalks.com (part 2 of 3)**

<img width="1600" height="741" alt="image" src="https://github.com/user-attachments/assets/5ca6d7d8-2cc6-4a9a-afcd-98e6545c0920" />


**Figure 3: WHOIS lookup of networkwalks.com (part 3 of 3)**

<img width="1600" height="700" alt="image" src="https://github.com/user-attachments/assets/6ec36c29-2697-477c-9f8d-981526e19f0a" />


**Figure 4: WhatWeb technology fingerprint of networkwalks.com**

<img width="1600" height="742" alt="image" src="https://github.com/user-attachments/assets/f81da988-70fb-444c-ac0d-16dc0b580623" />


**Figure 5: Nslookup resolving networkwalks.com to 192.232.216.135**

<img width="1600" height="185" alt="image" src="https://github.com/user-attachments/assets/efd58dd8-61b0-4e5b-ac5d-3b0b7a03bb17" />


**Figure 6: Curl -I HTTP response headers**

<img width="1600" height="742" alt="image" src="https://github.com/user-attachments/assets/1a8f5563-3a94-43c3-bb00-db6d47a7371a" />


**Figure 7: Wafw00f detecting ModSecurity (SpiderLabs)**

<img width="1600" height="739" alt="image" src="https://github.com/user-attachments/assets/e504e6a2-5585-4f21-ab39-611c8b185d3d" />

**Figure 8: DNSRecon DNS enumeration**

<img width="1600" height="744" alt="image" src="https://github.com/user-attachments/assets/b34b1ba2-9b4e-426b-9e95-58746baf9157" />


### 8.2 W2-PM4: OSINT with theHarvester

**Figure 9: theHarvester help menu (`theHarvester -h`)**
!

<img width="1600" height="742" alt="image" src="https://github.com/user-attachments/assets/68f3e093-b643-4ae5-9e33-fb2d637fa159" />


**Figure 10: theHarvester search on microsoft.com with source baidu and limit 1000**


<img width="1600" height="746" alt="image" src="https://github.com/user-attachments/assets/d85bbead-87b1-4951-8b0f-2038b84bd779" />

**Figure 11: theHarvester search on microsoft.com with limit 50 and all sources (start)**


<img width="1600" height="738" alt="image" src="https://github.com/user-attachments/assets/d7fe5dab-5439-4e75-8257-7b9e0f01ce2a" />


**Figure 12: theHarvester all-source run (sources being queried, part 2)**


<img width="1600" height="743" alt="image" src="https://github.com/user-attachments/assets/6876d23a-751a-4165-914d-5ae8566be8ed" />


**Figure 13: theHarvester all-source run (sources being queried, part 3)**



<img width="1600" height="738" alt="image" src="https://github.com/user-attachments/assets/61a88536-f56f-492a-9a6e-f11b359fc769" />


### 8.3 W2-PM5: Network scanning with Zenmap

**Figure 14: ipconfig /all on Windows (part 1), MAC addresses blurred for privacy**

<img width="1475" height="752" alt="image" src="https://github.com/user-attachments/assets/68fa6743-39e7-4b15-a933-a9a750f6150f" />


**Figure 15: ipconfig /all on Windows (part 2), IP and MAC addresses blurred for privacy**

<img width="1475" height="756" alt="image" src="https://github.com/user-attachments/assets/b81a1829-e169-419a-8f7b-081815124244" />

**Figure 16: Zenmap Ping scan of 192.168.1.0/24**

<img width="1600" height="849" alt="image" src="https://github.com/user-attachments/assets/7dfb96e8-7487-4ee8-a992-c7a20deb5e34" />


**Figure 17: Zenmap Quick scan of 192.168.1.0/24 showing open ports**

<img width="1600" height="850" alt="image" src="https://github.com/user-attachments/assets/4671528b-c96b-46e0-9134-c2574f6222be" />


**Figure 18: Zenmap network topology with legend**

<img width="621" height="494" alt="image" src="https://github.com/user-attachments/assets/fc5ce429-e7a4-453a-8cc0-3ae367298840" />


---

*-End-*

### 🧑‍💻 Author
🛡️ **K. Pranay Kumar**
Cybersecurity Professional B084

### 📌 Project Information
🎓 **Program Name:** Cybersecurity program at Networkwalks | **Week:** 02 | **Modules:** W2-PM1, W2-PM4, W2-PM5

