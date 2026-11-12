# -NETWORKWALKS-B084-WK2-CYBERSECURITY-PROJECTS
# PENETRATION TESTING REPORT

## FOOTPRINTING & NETWORK SCANNING PHASES

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| | |
|---|---|
| **Pentester Name (Cybersecurity Professional)** | Ojediran Timothy Temidayo |
| **Program/Batch** | NW-084-0Z6 |
| **Date** | 11 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools)<br>W2-PM5 (Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks (authorised as part of the assigned internship lab)<br>2. My own lab network (VirtualBox NAT Network 10.0.0.0/24) |
| **Permission secured from client?** | Yes |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting<br>**Phase 2:** Scanning & Network Discovery<br>**Phase 3-5:** In Progress |

---

## 1. Liability Disclaimer

I have performed these activities only on the systems and devices where I had secured written permission or the devices/systems that I own myself. All these materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

---

## 2. Introduction

This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning my own lab network with Zenmap (W2-PM5). One module covers the footprinting phase and the other covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my ongoing internship program at Networkwalks.

All commands were run in Kali Linux inside an Oracle VirtualBox lab. Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.

---

## 3. Tools Used

The table below lists each tool used in this report and its purpose.

| Tool | Purpose |
|------|---------|
| **Kali Linux & Oracle VirtualBox** | Operating system and virtual lab used for all reconnaissance and scanning activities. |
| **WHOIS** | Find domain registration details (registrar, dates, name servers). |
| **whatweb** | Fingerprint web technologies (server, CMS, plugins, IP). |
| **nslookup** | Resolve the domain name to its IP address using DNS. |
| **curl -I** | Read the HTTP response headers of the website. |
| **wafw00f** | Detect whether a Web Application Firewall protects the site. |
| **dnsrecon** | Enumerate DNS records (SOA, NS, MX, A, TXT/SPF, SRV). |
| **Zenmap (Nmap GUI)** | Scan the lab subnet to find live hosts and generate a network topology. |

---

## 4. Activities Performed

### 4.1 Footprinting & Reconnaissance

I performed reconnaissance against the `networkwalks.com` domain using six Kali Linux tools: **WHOIS, WhatWeb, Nslookup, Curl, Wafw00f and DNSRecon**. Each tool was used to collect a different type of information about the target.

First, I ran `whois networkwalks.com` to obtain publicly available domain registration information. The domain is registered through **GoDaddy** (created 6 November 2019, expiring 6 November 2027), the registrant details are hidden behind a privacy service (Domains By Proxy), and the name servers are **NS6135.HOSTGATOR.COM** and **NS6136.HOSTGATOR.COM**. The record also shows that DNSSEC is unsigned.

I then ran `whatweb networkwalks.com` to identify the technologies used by the website. The results identified **WordPress 7.1.4** and **WordPress Download Manager 3.3.76**, running on **Apache** with jQuery 3.7.1 and Google Tag Manager. The site redirects HTTP to HTTPS, the page title is "Networkwalks Academy", and the contact address info@networkwalks.com is exposed.

Using `nslookup networkwalks.com`, I resolved the domain name to its IP address. The result identified **192.232.216.135** through Google's DNS server (8.8.8.8).

I used `curl -I https://networkwalks.com` to inspect the HTTP response headers. The server returned **HTTP/2 200** and revealed the WordPress REST API endpoint **/wp-json/**, caching headers (x-endurance-cache-level, x-nginx-cache), a Permissions-Policy header and a secure, HttpOnly cookie. The response did not include HSTS, X-Frame-Options, CSP or X-Content-Type-Options headers.

Next, I ran `wafw00f networkwalks.com` to determine whether a Web Application Firewall was protecting the website. The result identified **ModSecurity (SpiderLabs)**.

Finally, I ran `dnsrecon -d networkwalks.com` to enumerate DNS records. It found the SOA and NS records (both on HostGator), the mail server **mail.networkwalks.com**, the A record, a Google site-verification TXT record, an SPF record (`v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`), and eight cPanel autodiscover SRV records. DNSRecon also disclosed the DNS software version, **BIND 9.16.23-RH**, on both name servers.

### 4.2 Network Scanning with Zenmap

For the second activity, I used **Zenmap** to perform network discovery on my own lab network. The practical required me to identify my local IP address and subnet, discover live hosts and generate a network topology.

The Kali VM is attached to the VirtualBox NAT Network configured in Week 1, with the static address 10.0.0.2/24 and gateway 10.0.0.1. I entered the subnet 10.0.0.0/24 into Zenmap and ran a Ping Scan to identify active hosts.

The scan identified two live hosts:

- `10.0.0.1` – the NAT Network gateway
- `10.0.0.2` – the Kali Linux VM itself

After completing the scan, I opened the Topology section in Zenmap and saved the network topology as a PDF, as required by the practical task.

---

## 5. Risk Analysis / Impact

Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|----------------|------------------------|------------------|------------|
| 1 | Web technology information exposed | WhatWeb identified WordPress 7.1.4, WordPress Download Manager 3.3.76, jQuery 3.7.1 and Apache | Attackers may use exposed software and version information to identify components needing further security review | 🟠 Medium |
| 2 | Server IP address identifiable | Nslookup resolved the domain to 192.232.216.135 | Provides information about the network location of the web service | 🟢 Low |
| 3 | HTTP technical information exposed | Curl returned response headers including the /wp-json/ REST API link and caching headers. HSTS, X-Frame-Options, CSP and X-Content-Type-Options were not seen in the response | May assist technology fingerprinting and further enumeration; missing security headers reduce browser-side protection | 🟢 Low |
| 4 | WAF technology identifiable | Wafw00f identified ModSecurity (SpiderLabs) | Reveals information about the web application's security architecture | 🟢 Low |
| 5 | DNS infrastructure and DNS software version exposed | DNSRecon listed NS, MX, SPF, TXT and SRV records and disclosed BIND version 9.16.23-RH on both name servers | DNS information helps build an infrastructure profile; a disclosed software version can help an attacker research known weaknesses | 🟠 Medium |
| 6 | DNSSEC not enabled | WHOIS showed DNSSEC: unsigned and DNSRecon got no answer to its DNSSEC query | DNS responses cannot be cryptographically validated, leaving room for spoofing attacks | 🟢 Low |
| 7 | Public email address and SPF soft fail | WhatWeb found info@networkwalks.com; the SPF record ends in ~all (soft fail) | Public addresses can be targeted by phishing; a soft-fail SPF policy is less strict against spoofed email | 🟢 Low |
| 8 | Live hosts visible on lab network | Zenmap found two live hosts: 10.0.0.1 and 10.0.0.2 | Both are expected lab devices, but the same technique would reveal any unknown device on a network | 🟢 Low |

**Risk level key:** 🔴 Critical &nbsp; 🟠 Medium &nbsp; 🟢 Low

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities.

The practical exercises primarily involved information gathering and host discovery. No exploitation or vulnerability validation was performed as part of these two modules.

Therefore, the presence of information such as a software version, IP address or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

---

## 6. Recommendations

Based on the observations from these activities, I recommend the following security improvements:

1. **Review publicly exposed technology information**
   Organizations should regularly review what information about their web technologies, CMS and plugins is publicly visible.

2. **Keep software updated**
   WordPress, the WordPress Download Manager plugin and other web components should be updated regularly and checked against current security advisories.

3. **Review and harden HTTP headers**
   Add security headers such as Strict-Transport-Security, X-Frame-Options (or CSP frame-ancestors) and X-Content-Type-Options, and review whether the public /wp-json/ REST API endpoints need to stay exposed.

4. **Review DNS records regularly**
   DNS records should be checked periodically so that only required information and services are public. Ask the hosting provider whether the BIND version string can be hidden on the name servers.

5. **Consider enabling DNSSEC**
   Signing the domain with DNSSEC lets resolvers validate DNS answers and reduces spoofing risk.

6. **Strengthen email authentication**
   Verify that DKIM and DMARC records are published, and consider moving the SPF policy from soft fail (~all) to hard fail (-all) once all legitimate senders are covered.

7. **Properly configure and monitor the WAF**
   Keep the WAF (ModSecurity) enabled and tuned, since it already helps block basic attacks.

8. **Perform regular internal network discovery**
   Organizations should periodically scan their own networks to identify active devices, and investigate any unexpected device that appears.

9. **Maintain network documentation**
   Network topology and device information should be documented and kept up to date.

10. **Perform security testing with authorization**
    Reconnaissance and scanning should only be performed against systems and networks where appropriate authorization has been provided.

---

## 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activity, I used six Kali Linux tools to collect information about the target domain. I learned how WHOIS can provide domain information, WhatWeb can identify web technologies, Nslookup can resolve domain names, Curl can inspect HTTP headers, Wafw00f can identify a WAF, and DNSRecon can provide detailed DNS information.

In the network scanning activity, I used Zenmap to discover active hosts on my lab network and generate a network topology.

The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses.

I also learned that technical findings should be documented clearly. A good cybersecurity report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk.

Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. These activities were completed as part of the assigned educational cybersecurity lab.

---

## 8. Evidences Collected

**Figure 1: WHOIS lookup for networkwalks.com**

<img width="1920" height="891" alt="whoisscreenshot" src="https://github.com/user-attachments/assets/8b7b8601-f1ad-4f64-9ab6-e9068857b200" />


**Figure 2: WhatWeb technology fingerprint**

<img width="1920" height="891" alt="whatweb_result" src="https://github.com/user-attachments/assets/0879f832-e3dc-4664-a041-9770c4109c9b" />


**Figure 3: Nslookup resolving the domain to its IP address**

<img width="1920" height="891" alt="nslookup_ss" src="https://github.com/user-attachments/assets/7f989e9c-ec81-472b-a1f9-798b580e1dcb" />


**Figure 4: curl -I HTTP response headers**

<img width="1920" height="891" alt="curl -l ss" src="https://github.com/user-attachments/assets/2cf440b7-40a2-445b-a012-7077f63ccc25" />


**Figure 5: Wafw00f WAF detection**

<img width="1920" height="891" alt="wafw00f ss" src="https://github.com/user-attachments/assets/c182ccb0-57e5-4b9f-82ea-6531de7965f8" />


**Figure 6: DNSRecon DNS enumeration**

<img width="1920" height="891" alt="dnsrecon_ss" src="https://github.com/user-attachments/assets/6e764323-f38a-478b-aeb6-3591fd87c1b5" />


**Figure 7: Zenmap network topology of the lab subnet**

<img width="384" height="168" alt="image" src="https://github.com/user-attachments/assets/17dd6b39-cf38-4684-af2e-d128b427dca5" />
