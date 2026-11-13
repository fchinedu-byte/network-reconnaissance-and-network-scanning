# Network Reconnaissance, Footprinting & Network Scanning

## Project Overview

This repository documents my practical work on network reconnaissance, footprinting, DNS enumeration and local network scanning as part of my Week 2 cybersecurity internship project at NetworkWalks.

The project focused on understanding how security professionals gather information about digital infrastructure, identify technologies and services, examine DNS configurations, and discover active devices within a network.

The practical exercises were divided into two phases:

1. **Domain reconnaissance and footprinting:** Gathering and analysing information about a domain using DNS enumeration and website reconnaissance tools.
2. **Local network discovery and scanning:** Identifying active hosts, IP addresses and available MAC address information using Nmap and Zenmap.

The project emphasised understanding tool outputs, interpreting findings and documenting observations rather than simply executing commands.

---

## Project Objectives

- Understand the reconnaissance and footprinting process.
- Identify publicly accessible domain and website information.
- Examine DNS records and understand their functions.
- Understand email authentication mechanisms, particularly SPF.
- Analyse HTTP response headers.
- Attempt to identify Web Application Firewall protection.
- Discover active devices on a local network.
- Identify IP addresses and available MAC addresses.
- Generate a network topology using Zenmap.
- Document findings, limitations and security recommendations.

---

## Tools and Technologies

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration information gathering |
| WhatWeb | Website technology fingerprinting |
| NSLOOKUP | DNS record enumeration |
| cURL | HTTP response header analysis |
| DNSRecon | DNS enumeration and information gathering |
| WAFW00F | Web Application Firewall detection |
| Nmap | Network discovery and scanning |
| Zenmap | Graphical network scanning and topology visualisation |
| Windows CMD | Network configuration and ARP table inspection |

---

# PHASE 1: DOMAIN RECONNAISSANCE AND FOOTPRINTING

## 1. Understanding Reconnaissance and Footprinting

Reconnaissance is the broader process of collecting information about a target before a security assessment.

Footprinting is an information-gathering activity within reconnaissance that focuses on identifying and documenting a target's digital presence and infrastructure.

Reconnaissance may involve identifying domains, IP addresses, technologies, services and potential areas requiring further investigation.

Footprinting helps build a profile of the target by collecting information such as domain registration details, DNS records, mail servers and web technologies.

## 2. WHOIS – Domain Registration Information

WHOIS is used to retrieve information about domain registration, including available registrar details, registration dates, nameservers and domain status.

**Command:**

```bash
whois networkwalks.com
```

**Purpose:**

To understand the domain's registration information and identify publicly available administrative and technical details.

**Key learning:**

WHOIS information can help establish domain ownership-related details and identify infrastructure information useful during reconnaissance.

*Note: Include the actual WHOIS output and observations in the evidence folder if available.*

## 3. WhatWeb – Website Technology Fingerprinting

**Command:**

```bash
whatweb https://networkwalks.com
```

WhatWeb was used to identify technologies and components associated with the target website.

### Observed findings

| Finding | Result |
|---|---|
| HTTP status | 200 OK |
| Web server fingerprint | Apache |
| Content management system | WordPress |
| jQuery | 3.7.1 |
| WordPress Download Manager | 3.3.76 |
| Google Tag Manager | Detected |
| Website title | Networkwalks Academy |
| Reported IP address | 192.232.216.135 |

### Analysis

The HTTP 200 response indicated that the website successfully responded to the request.

The WordPress fingerprint suggested that the website uses WordPress as its content management system.

The detected JavaScript libraries, plugins and server information provided additional insight into the website's technology stack.

**Security relevance:**

Technology fingerprinting helps security analysts understand the software and components used by a website. This information can support further security assessment and vulnerability research.

However, identifying a technology or software version does not automatically establish that a vulnerability exists. Additional verification is required.

## 4. cURL – HTTP Response Header Analysis

**Command:**

```bash
curl -I https://networkwalks.com
```

The purpose of this exercise was to inspect HTTP response headers and understand what information the web server returned.

### Observed findings

| Header | Observation |
|---|---|
| HTTP Status | 200 OK |
| Server | Apache |
| Set-Cookie | `__wpdm_client` |
| Secure | Present on the observed cookie |
| HttpOnly | Present on the observed cookie |
| Referrer-Policy | no-referrer-when-downgrade |
| Permissions-Policy | Present |
| X-nginx-cache | WordPress |
| Content-Type | text/html; charset=UTF-8 |

### Analysis

The response indicated that the website was accessible and returned several HTTP security-related headers.

The cookie included the Secure and HttpOnly attributes:

- **Secure:** Restricts the cookie to secure HTTPS connections.
- **HttpOnly:** Prevents direct access to the cookie through client-side JavaScript.

The response also included a Referrer-Policy and Permissions-Policy header.

The following headers were not observed in this particular response:

- Strict-Transport-Security (HSTS)
- Content-Security-Policy (CSP)
- X-Frame-Options

Their absence in this response is worth investigating, but does not independently prove a security vulnerability.

## 5. NSLOOKUP – DNS Enumeration

NSLOOKUP was used to investigate DNS records and understand how the domain's website and email infrastructure were configured.

### 5.1 A Record

**Command:**

```bash
nslookup -type=A networkwalks.com
```

**Result:**

```text
networkwalks.com
Address: 192.232.216.135
```

**Explanation:**

An A record maps a domain name to an IPv4 address.

### 5.2 AAAA Record

**Command:**

```bash
nslookup -type=AAAA networkwalks.com
```

**Result:**

```text
Can't find networkwalks.com: No answer
```

**Explanation:**

An AAAA record maps a domain name to an IPv6 address.

No AAAA record was returned for the queried hostname during this test.

This does not establish that the entire hosting infrastructure lacks IPv6 support.

### 5.3 MX Record – Mail Exchange

**Command:**

```bash
nslookup -type=MX networkwalks.com
```

**Result:**

```text
networkwalks.com
mail exchanger = 0 mail.networkwalks.com
```

**Explanation:**

MX records identify the mail servers responsible for receiving emails on behalf of a domain.

The result identified:

- Mail server: `mail.networkwalks.com`
- Priority: `0`

**Understanding MX priority**

MX priority determines the preference of mail servers. A lower numerical value represents a higher preference.

For example:

| Mail server | Priority |
|---|---:|
| mail1.example.com | 0 |
| mail2.example.com | 10 |
| mail3.example.com | 20 |

The sending mail server normally attempts delivery to the server with the lowest priority number first.

If that server is unavailable, it may attempt another configured MX server.

### 5.4 NS Record – Nameservers

**Command:**

```bash
nslookup -type=NS networkwalks.com
```

**Result:**

```text
ns6136.hostgator.com
ns6135.hostgator.com
```

**Explanation:**

NS records identify the nameservers responsible for providing authoritative DNS information for a domain.

The returned records indicated that the domain's DNS delegation used HostGator nameservers.

This does not necessarily mean that the website itself is hosted on the same infrastructure as its nameservers.

### 5.5 TXT Record – SPF and Domain Verification

**Command:**

```bash
nslookup -type=TXT networkwalks.com
```

**Result:**

```text
"v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all"

"google-site-verification=rr04eRmqHoWY3XemnizDNVK4q75X-Ij-mjgEeg-UsYI"
```

Two TXT records were returned.

One contained an SPF policy, while the other was a Google domain verification record.

### Understanding SPF (Sender Policy Framework)

SPF is an email authentication mechanism that allows domain owners to specify which mail servers are authorised to send emails on behalf of their domain.

The observed SPF record was:

```text
v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all
```

**Breakdown:**

| Component | Meaning |
|---|---|
| `v=spf1` | Identifies the SPF version |
| `+a` | Authorises the domain's A record address |
| `+mx` | Authorises the domain's MX hosts |
| `+ip4:50.87.144.87` | Explicitly authorises this IPv4 address |
| `+include:websitewelcome.com` | Includes another domain's SPF policy |
| `~all` | Marks unmatched senders as softfail |

### Understanding SPF Softfail

The `~all` mechanism is known as softfail.

It means that a sender that does not match the authorised SPF mechanisms should be treated as a softfail.

However, softfail does not explicitly instruct the receiving mail server to reject the message. The receiving server can consider this result alongside other authentication checks and its own policies.

**Comparison of SPF mechanisms:**

| Mechanism | Meaning |
|---|---|
| `+all` | All senders are authorised |
| `?all` | Neutral result |
| `~all` | Softfail for unmatched senders |
| `-all` | Hardfail for unmatched senders |

SPF is not the only email authentication mechanism. It works alongside:

- **DKIM:** Uses cryptographic signatures to help verify email authenticity and message integrity.
- **DMARC:** Allows domain owners to specify how receiving servers should handle messages that fail authentication checks and provides reporting mechanisms.

Understanding these mechanisms helped me see how DNS configuration contributes to email security and helps reduce domain spoofing.

## 6. DNSRecon – DNS Enumeration

**Command:**

```bash
dnsrecon -d networkwalks.com
```

DNSRecon was used to gather additional DNS information beyond the initial NSLOOKUP queries.

### Observed findings

The output identified:

- SOA record information.
- Two nameservers.
- Nameserver IP addresses.
- MX record information.
- TXT records.
- SRV records.
- A reported BIND version banner: `9.16.23-RH`.

### Nameserver information

| Nameserver | IP address |
|---|---|
| ns6135.hostgator.com | 50.87.144.87 |
| ns6136.hostgator.com | 192.232.216.131 |

### Additional observations

DNSRecon reported SRV records associated with service discovery, including mail-related services.

It also returned a BIND version banner. This provides software identification information, but does not prove that the software is vulnerable.

An important observation was a difference between the A record results:

- Earlier WhatWeb and NSLOOKUP output returned `192.232.216.135`.
- DNSRecon reported `192.232.216.131`.

This discrepancy requires further investigation using consistent DNS resolvers and repeated queries. It could reflect multiple DNS answers, resolver differences or DNS changes.

The DNSSEC query returned an error indicating no answer. This alone is insufficient to establish whether DNSSEC is enabled or disabled.

## 7. WAFW00F – Web Application Firewall Detection

**Command:**

```bash
wafw00f https://networkwalks.com
```

### Result

The attempt returned a DNS resolution error:

```text
Failed to resolve 'networkwalks.com'
Temporary failure in name resolution
```

### Analysis

WAFW00F could not resolve the domain during this attempt, preventing the detection process from completing.

This result does not establish that the website lacks WAF protection or that the website was unavailable.

The unsuccessful attempt was documented as a DNS resolution issue.

---

# PHASE 2: LOCAL NETWORK DISCOVERY USING NMAP AND ZENMAP

## 8. Identifying the Local Network

Before performing network discovery, I used Windows Command Prompt to inspect the local network configuration.

**Command:**

```cmd
ipconfig
```

The network configuration identified the following:

| Parameter | Value |
|---|---|
| IPv4 address | 192.168.1.210 |
| Subnet mask | 255.255.255.0 |
| Network address | 192.168.1.0 |
| CIDR notation | /24 |

The subnet mask `255.255.255.0` corresponds to a `/24` network.

The target subnet for discovery was therefore:

```text
192.168.1.0/24
```

This subnet contains 256 possible IPv4 addresses.

## 9. ARP Table Inspection

**Command:**

```cmd
arp -a
```

The ARP table was inspected to identify IP-to-MAC address mappings already known to the Windows computer.

Several entries were observed on the `192.168.1.0/24` network, including:

- `192.168.1.1`
- `192.168.1.160`
- `192.168.1.195`
- `192.168.1.234`

The ARP table provided preliminary information about devices communicating on the network.

However, the ARP cache is not a complete network inventory because it only contains entries currently known to the computer.

## 10. Network Host Discovery with Zenmap

Zenmap, the graphical interface for Nmap, was used to perform host discovery on the local subnet.

**Target network:**

```text
192.168.1.0/24
```

### Scan result

```text
Nmap done: 256 IP addresses (23 hosts up) scanned in 4.90 seconds
```

### Analysis

The scan examined 256 possible IP addresses and identified 23 live hosts.

This means 23 devices responded to the discovery probes at the time of scanning.

Host discovery is useful for identifying active devices and building an initial picture of a network.

It does not independently identify vulnerabilities or establish that a device is compromised.

## 11. Discovered Hosts and MAC Addresses

The scan returned the following host information:

| IP address | MAC address | Manufacturer reported by Nmap |
|---|---|---|
| 192.168.1.1 | 74:24:9F:7B:BE:E5 | Tibro |
| 192.168.1.21 | BA:60:40:A1:6E:D7 | Unknown |
| 192.168.1.33 | 3C:F0:11:84:3A:DC | Intel Corporate |
| 192.168.1.38 | 1C:4D:70:B6:2E:ED | Intel Corporate |
| 192.168.1.57 | 84:3A:4B:12:D0:76 | Intel Corporate |
| 192.168.1.60 | 16:45:16:3E:E9:1F | Unknown |
| 192.168.1.63 | 1A:77:DA:A8:4D:A1 | Unknown |
| 192.168.1.66 | EE:CA:75:DE:F6:AA | Unknown |
| 192.168.1.87 | FA:58:88:EC:19:86 | Unknown |
| 192.168.1.90 | C2:D2:C9:3B:53:3F | Unknown |
| 192.168.1.132 | 22:A4:95:EF:37:44 | Unknown |
| 192.168.1.133 | 48:45:20:EA:FD:30 | Intel Corporate |
| 192.168.1.137 | E2:D4:0F:DB:32:0E | Unknown |
| 192.168.1.139 | E0:9D:31:39:50:C0 | Intel Corporate |
| 192.168.1.160 | AA:81:04:B7:86:A2 | Unknown |
| 192.168.1.168 | 7C:2A:31:B1:25:6A | Intel Corporate |
| 192.168.1.195 | 40:74:E0:EF:6A:F5 | Intel Corporate |
| 192.168.1.199 | C4:D9:87:A3:A2:91 | Intel Corporate |
| 192.168.1.210 | Not reported | Not reported |
| 192.168.1.219 | CA:F1:79:FC:56:44 | Unknown |
| 192.168.1.224 | 64:BC:58:EC:10:5D | Intel Corporate |
| 192.168.1.225 | 08:11:96:EA:73:98 | Intel Corporate |
| 192.168.1.234 | F4:28:9D:D6:7C:F3 | Cloud Network Technology Singapore PTE. |

### MAC address observations

Nmap displayed MAC addresses for 22 of the 23 discovered hosts.

The address `192.168.1.210` was reported as active, but no MAC address was displayed in the supplied scan output.

Earlier ARP information and the local IP configuration suggested that this address belonged to the scanning computer.

The manufacturer information was based on MAC address vendor identification. An unknown manufacturer does not necessarily indicate a hidden or suspicious device. Some devices use locally administered or randomized MAC addresses.

## 12. Network Topology Visualisation

Zenmap includes a Topology tab that provides a visual representation of discovered hosts and their scan relationships.

The topology feature helps present network discovery results in a more understandable format.

**Evidence to include:**

- Zenmap host discovery screenshot.
- Topology visualisation.
- Saved Nmap scan output.
- Exported topology PDF.

The topology visualisation should not be interpreted as a complete physical network diagram. It represents discovered hosts and inferred relationships based on the scan.

---

# PHASE 3: FINDINGS AND SECURITY RECOMMENDATIONS

## 13. Summary of Findings

| Assessment area | Key observation |
|---|---|
| Website fingerprinting | Apache and WordPress detected |
| HTTP analysis | HTTP 200 OK and several response headers observed |
| DNS A record | IPv4 address returned |
| DNS AAAA record | No answer returned |
| MX record | mail.networkwalks.com, priority 0 |
| NS records | Two HostGator nameservers identified |
| SPF | SPF policy included `~all` |
| DNS enumeration | Additional SOA, SRV and nameserver information identified |
| WAF detection | Inconclusive due to DNS resolution failure |
| Local network discovery | 23 live hosts discovered |
| MAC identification | MAC addresses displayed for 22 hosts |

## 14. Security Recommendations

The following are general recommendations based on the information gathered. They are not claims that the target has confirmed vulnerabilities.

### Website security

- Keep WordPress, themes and plugins updated.
- Remove unused plugins and components.
- Review HTTP security header configuration.
- Monitor exposed services.
- Verify WAF protection through an authorised assessment.

### Email security

- Review SPF records to ensure all legitimate sending sources are authorised.
- Evaluate whether SPF hardfail (`-all`) is appropriate after confirming legitimate mail sources.
- Configure and validate DKIM.
- Publish and review a DMARC policy.
- Monitor email authentication results.

Changing SPF from softfail to hardfail should only be considered after verifying all legitimate email-sending services.

### Network security

- Maintain an inventory of authorised network devices.
- Investigate unfamiliar devices.
- Use strong Wi-Fi encryption and authentication.
- Keep router firmware and connected devices updated.
- Disable unnecessary network services.
- Restrict administrative access to trusted devices.
- Monitor network traffic for unusual activity.

---

# PHASE 4: LIMITATIONS AND REFLECTION

## 15. Limitations

Several limitations were encountered:

1. WAFW00F could not complete its detection attempt because of a DNS resolution failure.
2. DNSRecon and NSLOOKUP returned different A record information, requiring further verification.
3. Some MAC addresses could not be associated with a manufacturer.
4. The local host discovery scan did not establish the operating system, open ports or vulnerabilities of all 23 devices.
5. No exploitation, vulnerability validation or post-exploitation activities were performed.
6. The scan represents a snapshot of devices responding at the time of testing.

These limitations demonstrate why security findings should be verified before conclusions are made.

## 16. What I Learned

This project improved my understanding of how reconnaissance supports cybersecurity assessments.

Some of my most important learning outcomes were:

- Understanding the difference between reconnaissance and footprinting.
- Learning how DNS records support website and email infrastructure.
- Understanding how MX priority affects mail delivery.
- Learning how SPF works and the difference between softfail and hardfail.
- Understanding how SPF, DKIM and DMARC contribute to email authentication.
- Identifying website technologies through fingerprinting.
- Interpreting HTTP response headers.
- Using Nmap and Zenmap to discover active network hosts.
- Understanding the relationship between IP addresses and MAC addresses.
- Documenting findings without making unsupported vulnerability claims.

## 17. Conclusion

The project provided practical exposure to domain reconnaissance, DNS enumeration, website fingerprinting and local network discovery.

Using tools such as WHOIS, WhatWeb, NSLOOKUP, cURL, DNSRecon, WAFW00F, Nmap and Zenmap helped me understand how different information-gathering techniques contribute to a broader security assessment.

A particularly valuable learning experience was examining DNS and email authentication, especially MX priority and SPF softfail.

The network scanning exercise also demonstrated how Nmap can identify active devices on a local network. A total of 23 live hosts were discovered on the `192.168.1.0/24` subnet.

Overall, the project strengthened my technical foundation in network security and reinforced the importance of interpreting results accurately, verifying observations and documenting security assessments responsibly.
