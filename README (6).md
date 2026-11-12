# PENETRATION TESTING REPORT

**FOOTPRINTING & NETWORK SCANNING PHASES**

B084D-WK2-PM1 | CYBERSECURITY | NETWORKWALKS

| | |
|---|---|
| **Pentester Name (Cybersecurity Professional)** | **OYERU RACHAEL EMALOHI** |
| **Program/Batch** | B084D-Networkwalks |
| **Date** | 11–12 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools & Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks (`networkwalks.com`) <br> 2. My own local network (`172.20.x.x/28`, address partly hidden for privacy) |
| **Permission secured from client?** | **YES** |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting <br> **Phase 2:** Scanning & Network Discovery <br> **Phase 3-5:** In Progress |

---

## 1. Liability Disclaimer

I have performed these activities only on the systems and devices where I had secured written permission, or on devices and systems that I own myself. All materials here are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

## 2. Introduction

This report covers footprinting the `networkwalks.com` domain using multiple Kali Linux tools and scanning my own local network with Zenmap. One part covers the footprinting phase and the other covers the scanning phase. Together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my ongoing internship program at Networkwalks.

Footprinting was run in Kali Linux (VirtualBox, `kali-linux-2025.4`). Scanning was run with Zenmap on my own PC. Every step below includes the command used, the result observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.

> **Privacy note:** IP addresses, MAC addresses and other details that could be used to locate or target my own network have been hidden or blurred in this report and in the screenshots.

## 3. Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux & Windows | Operating systems used for reconnaissance activities |
| WHOIS | Find domain registration details (registrar, dates, name servers). |
| WhatWeb | Fingerprint web technologies (server, CMS, plugins, IP). |
| nslookup | Resolve the domain name to its IP address using DNS. |
| curl -I | Read the HTTP response headers of the website. |
| wafw00f | Detect whether a Web Application Firewall protects the site. |
| dnsrecon | Enumerate DNS records (attempted; see 4.1). |
| Maltego | Graph-based OSINT: find email addresses linked to the domain. |
| theHarvester | Gather emails/subdomains from public sources (attempted; see 4.1). |
| Zenmap (Nmap GUI) | Scan the local subnet to find live hosts, IPs and MAC addresses. |

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance

I performed reconnaissance against the `networkwalks.com` domain using Kali Linux tools and Maltego. Each tool collected a different type of information.

1. **WHOIS** – Public registration data. The domain is registered with **GoDaddy.com, LLC**, created **2019-11-06**, expires **2027-11-06**, uses name servers **NS6135/NS6136.HOSTGATOR.COM** (and NS29/NS30.DOMAINCONTROL.COM), has **DNSSEC: unsigned**, and the owner's details are hidden behind the privacy service **Domains By Proxy, LLC**.
2. **WhatWeb** – Identified **Apache**, **WordPress 7.1.4**, **WordPress Download Manager 3.3.76**, jQuery 3.7.1, Google Tag Manager, the server IP (hidden: `192.232.xxx.xxx`), the contact email `info@networkwalks.com`, and the site title "Networkwalks Academy". The raw output is in [`evidence/pentest_result_wk2.docx`](evidence/pentest_result_wk2.docx).
3. **Nslookup** – Resolved `networkwalks.com` to its server IP (hidden: `192.232.xxx.xxx`) using DNS server 8.8.8.8.
4. **Curl `-I`** – HTTP/2 200 response from an **Apache** server. Headers exposed the WordPress REST API endpoint `/wp-json/`, a `__wpdm_client` cookie, and `x-nginx-cache: WordPress`.
5. **Wafw00f** – Detected that the site is behind a **ModSecurity (SpiderLabs)** WAF.
6. **Maltego** – Ran the *To Emails @domain* transform on `networkwalks.com`; it returned **info@networkwalks.com**.
7. **DNSRecon** – *Attempted, did not complete.* `dnsrecon -d networkwalks.com` failed with `OSError: [Errno 30] Read-only file system: '/home/kali/.config/dnsrecon/dnsrecon.log'` because the Kali VM could not write its log file.
8. **theHarvester** – *Attempted, did not complete.* Two runs (`-d microsoft.com -1 1000 -b baidu` and `-d microsoft.com -1 50 -b all`) failed with `unrecognized arguments`. The limit option is a lowercase letter `-l`, not the digit `-1`.

### 4.2 Network Scanning with Zenmap

**I used Zenmap (the graphical interface for Nmap) to perform network discovery on my own local network.**

- **Scan profile:** **Ping scan**, equivalent to `nmap -sn 172.20.x.x/28` (address hidden for privacy). This scan only checks which devices are online. It does not probe their ports.
- **Started:** 2026-11-11 15:44 (+0100) with Nmap 7.991.
- **Result:** **16 IP addresses scanned, 2 hosts up**, completed in 2.82 seconds.
  - Host 1: `172.20.x.1`, MAC address hidden (vendor unknown)
  - Host 2: `172.20.x.2`
- **Topology:** I then opened the **Topology** tab and saved the network map as a PDF: [`evidence/zenmap_topology.pdf`](evidence/zenmap_topology.pdf).

**Why it matters:** this is how an attacker maps a network before choosing a target. For a defender, it shows which devices are visible and whether any are unexpected.

## 5. Risk Analysis / Impact

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Web technology and version information exposed | WhatWeb identified WordPress 7.1.4 and WordPress Download Manager 3.3.76 | Attackers may use exposed technology/version information to identify software requiring further security review | 🟠 Medium |
| 2 | Server IP address identifiable | Nslookup resolved the domain to a public IP (hidden) | Provides information about the network location of the web service | 🟢 Low |
| 3 | HTTP technical information exposed | Curl returned response headers and exposed `/wp-json/` | May assist technology fingerprinting and further enumeration | 🟢 Low |
| 4 | WAF technology identifiable | Wafw00f identified ModSecurity (SpiderLabs) | Reveals information about the web application's security architecture | 🟢 Low |
| 5 | Domain/DNS information exposed | WHOIS showed registrar, dates and name servers; DNSSEC unsigned | Helps build a broader infrastructure profile; unsigned DNSSEC offers no protection against DNS spoofing | 🟠 Medium |
| 6 | Public contact email discoverable | Maltego returned `info@networkwalks.com` | Could be used for phishing or social engineering | 🟢 Low |
| 7 | Live hosts visible on local network | Zenmap identified two live hosts in the scanned subnet | Unknown or unauthorized devices may be present on a network | 🟢 Low |

**Risk level key:** 🔴 Critical | 🟠 Medium | 🟢 Low

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities. No exploitation or vulnerability validation was performed. The presence of a software version, IP address or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

## 6. Recommendations

1. **Review publicly exposed technology information** – Regularly review what information about web technologies, CMS and plugins is publicly visible.
2. **Keep software updated** – Keep CMS platforms and plugins updated and reviewed against current security advisories.
3. **Review HTTP headers** – Check whether unnecessary technical information is being exposed.
4. **Review DNS records and enable DNSSEC** – Check DNS records periodically and consider signing the zone.
5. **Properly configure and monitor the WAF** – Keep ModSecurity enabled and tuned.
6. **Limit exposure of public email addresses** – Use contact forms or role-based addresses and train staff on phishing.
7. **Perform regular internal network discovery** – Periodically scan your own network to identify active devices.
8. **Investigate unknown devices** – Verify any unexpected device discovered.
9. **Maintain network documentation** – Keep topology and device information up to date.
10. **Perform security testing with authorization** – Only scan systems where authorization has been given.

## 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning. I used WHOIS, WhatWeb, Nslookup, Curl, Wafw00f and Maltego to collect information about the target domain, and Zenmap to discover live hosts on my local network and create a topology.

Two tools, DNSRecon and theHarvester, did not complete. DNSRecon failed because of a read-only file system on the Kali VM, and theHarvester failed because of a mistyped option. Both are documented here and will be re-run once fixed.

The exercises showed me that information gathering is an important part of cybersecurity. A good report should explain what was performed, what was discovered, what it means, what risk it may create, and how to reduce that risk. Reconnaissance and scanning must always be performed within an authorized scope.

## 8. Evidence Collected

| # | Activity | Screenshot |
|---|---|---|
| 1 | Zenmap ping scan | ![Zenmap ping scan](screenshots/01_zenmap_ping_scan.png) |
| 2 | WHOIS (part 1) | ![WHOIS 1](screenshots/02_whois_part1.png) |
| 3 | WHOIS (part 2) | ![WHOIS 2](screenshots/03_whois_part2.png) |
| 4 | WHOIS (part 3) | ![WHOIS 3](screenshots/04_whois_part3.png) |
| 5 | Nslookup | ![Nslookup](screenshots/05_nslookup.png) |
| 6 | Curl -I | ![Curl -I](screenshots/06_curl_headers.png) |
| 7 | Curl (alternate capture) | ![Curl](screenshots/07_curl_headers_alt.jpg) |
| 8 | Wafw00f | ![Wafw00f](screenshots/08_wafw00f.png) |
| 9 | DNSRecon (error) | ![DNSRecon error](screenshots/09_dnsrecon_error.png) |
| 10 | Maltego | ![Maltego](screenshots/10_maltego_domain_to_email.png) |
| 11 | theHarvester (error 1) | ![theHarvester 1](screenshots/11_theharvester_error_1.png) |
| 12 | theHarvester (error 2) | ![theHarvester 2](screenshots/12_theharvester_error_2.png) |
| 13 | Zenmap topology | ![Zenmap topology](screenshots/13_zenmap_topology.png) |

-End-

**👤 Author**

**OYERU RACHAEL EMALOHI**
Cybersecurity Professional B084D

**📌 Project Information**

**Program Name:** Cybersecurity program at Networkwalks | **Week:** 02 | **Repository:** GitHub
