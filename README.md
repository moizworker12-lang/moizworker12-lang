# 👑 Reven 1 – Complete Boot2Root Walkthrough

## 🔰 Introduction

**Reven 1** is an intentionally vulnerable machine used to practice penetration testing, network enumeration, web application security, Linux authentication, and privilege escalation.

The objective of this challenge was to investigate the target, identify security weaknesses, obtain initial access, capture all four flags, and ultimately achieve root access.

### 🎯 Objectives

- Discover the target machine's IP address.
- Perform network scanning and service enumeration.
- Investigate the web application.
- Identify usernames and credentials.
- Obtain initial access through SSH.
- Discover all four flags.
- Investigate privilege escalation opportunities.
- Achieve root access.

**Attacker Machine:** Kali Linux
**Target Machine:** Reven 1
**Virtualization:** VirtualBox
**Network:** Host-Only Adapter
**Target IP:** `192.168.56.103`

---

# 🔍 Phase 1 – Reconnaissance

## Step 1 – Network Discovery

First, I needed to identify the target machine's IP address on the virtual network.

For this purpose, I used Netdiscover, a network discovery tool that identifies active devices on a local network.

**Command:**

```bash
sudo netdiscover -r 192.168.56.0/24
```

Alternative:

```bash
sudo netdiscover
```

### What does this command do?

- `sudo` – Runs the command with administrative privileges.
- `netdiscover` – Starts network discovery.
- `-r` – Specifies the network range.
- `192.168.56.0/24` – The Host-Only network range.

After examining the discovered devices, I identified the target machine.

**Target IP:**

```text
192.168.56.103
```

📸 Screenshot: Add your Netdiscover output here.

---

## Step 2 – Nmap Scanning

After discovering the target IP, I performed an Nmap scan to identify open ports, running services, and service versions.

**Command:**

```bash
sudo nmap -sC -sV -p- 192.168.56.103
```

### Command Breakdown

- `sudo` – Runs Nmap with elevated privileges.
- `nmap` – Network scanning tool.
- `-sC` – Runs default NSE scripts.
- `-sV` – Detects service versions.
- `-p-` – Scans all TCP ports.
- Target IP – Specifies the machine being scanned.

### Scan Results

The important findings included:

| Port      | Service | Version / Details |
| --------- | ------- | ----------------- |
| 22/tcp    | SSH     | OpenSSH 6.7p1     |
| 80/tcp    | HTTP    | Apache 2.4.10     |
| 56642/tcp | Status  | RPC service       |

### Additional RPC Enumeration

I also used:

```bash
rpcinfo -p 192.168.56.103
```

This command helps enumerate RPC services running on the target.

### Why was this step important?

Port scanning helped identify the available services and provided possible directions for further investigation.

📸 Screenshot: Add your Nmap results here.

---

# 🌐 Phase 2 – Web Enumeration

## Step 3 – Website Investigation

Since port 80 was open, I opened the target's web server in Firefox.

**URL:**

```text
http://192.168.56.103
```

The website displayed the title:

```text
Raven Security
```

I inspected the website, navigated through available pages, and examined the HTML source code.

### What I investigated

- Website structure.
- Available pages.
- HTML source code.
- Interesting information exposed in webpages.
- WordPress-related content.

I also investigated the web technology using WhatWeb.

**Command:**

```bash
whatweb http://192.168.56.103
```

WhatWeb identifies technologies and software used by a website.

📸 Screenshot: Add your website and source-code screenshots here.

---

## Step 4 – Directory Enumeration

Next, I performed directory enumeration to discover directories and files that were not immediately visible on the website.

### Tool: Gobuster

**Command:**

```bash
gobuster dir -u http://192.168.56.103 -w /usr/share/wordlists/dirb/common.txt -t 10
```

### Command Breakdown

- `gobuster` – Directory enumeration tool.
- `dir` – Enables directory discovery mode.
- `-u` – Specifies the target URL.
- `-w` – Specifies the wordlist.
- `-t 10` – Uses 10 concurrent threads.

### Interesting Findings

The scan identified several directories:

```text
/wordpress/
/manual/
/css/
/fonts/
/img/
/js/
/vendor/
```

The `/wordpress/` directory was particularly interesting and required further investigation.

### Alternative Tool: DIRB

The following command can also perform directory enumeration:

```bash
dirb http://192.168.56.103
```

📸 Screenshot: Add your Gobuster output here.

---

## Step 5 – WordPress Enumeration

After discovering the WordPress directory, I investigated the installation to identify available user accounts.

### Tool: WPScan

**Command:**

```bash
wpscan --url http://192.168.56.103/wordpress --enumerate u
```

### Command Breakdown

- `wpscan` – WordPress security scanner.
- `--url` – Specifies the WordPress installation.
- `--enumerate u` – Enumerates WordPress users.

### Discovered Usernames

During enumeration, I identified usernames including:

```text
micheal
steven
```

These usernames provided useful information for further authentication investigation.

📸 Screenshot: Add your WPScan results here.

---

# 🔐 Phase 3 – Initial Access

## Step 6 – Credential Investigation

After discovering usernames, I investigated the authentication attack surface to identify valid credentials.

The SSH service was available on port 22.

During the authorized lab investigation, I used credential testing against the vulnerable target and identified account information.

**SSH service:**

```text
22/tcp - OpenSSH 6.7p1
```

The following is a command recovered from your earlier lab notes for the SSH credential-testing stage:

```bash
hydra ssh://192.168.56.103 -l micheal -P /usr/share/wordlists/rockyou.txt -V -t 4
```

### Command Breakdown

- `hydra` – Login authentication testing tool.
- `ssh://` – Specifies SSH as the service.
- `-l micheal` – Specifies a username.
- `-P` – Specifies a password wordlist.
- `-V` – Displays detailed attempts.
- `-t 4` – Uses four parallel tasks.

*This records the command from your lab notes; verify which account actually produced the successful login before publishing the result.*

### Credentials Obtained

The account you later confirmed accessing was:

```text
Username: steven
Password: pink84
```

Never publish your actual password in a public GitHub repository.

📸 Screenshot: Add your credential discovery evidence here, with sensitive values hidden.

---

## Step 7 – SSH Login

After obtaining the valid credentials, I established a remote connection to the target machine through SSH.

**Command:**

```bash
ssh steven@192.168.56.103
```

### Command Explanation

- `ssh` – Secure Shell client.
- `steven` – Target username.
- `192.168.56.103` – Target IP address.

After entering the password, I successfully logged into the target machine.

### Verify Initial Access

Once logged in, I used:

```bash
whoami
```

To identify the current user.

I also used:

```bash
id
```

To check the user's UID, GID, and group memberships.

**Result:** Initial access successfully obtained.

📸 Screenshot: Add your successful SSH login here.

---

# 🚩 Phase 4 – Flag Collection

## Step 8 – Filesystem Exploration

After gaining SSH access, I started investigating the target's filesystem to locate the available flags.

### Useful Linux Commands

Check the current directory:

```bash
pwd
```

List directory contents:

```bash
ls -la
```

Navigate to the web directory:

```bash
cd /var/www
```

Explore its contents:

```bash
ls -la
```

Search for flag-related files in accessible locations:

```bash
find /var/www -type f -iname '*flag*' 2>/dev/null
```

Read an identified flag file:

```bash
cat flag2.txt
```

*The filename above is an example from your earlier walkthrough format. Keep it only if it matches your actual Reven findings.*

### Flag Collection

During the challenge, I successfully captured all four flags.

| Flag   | Status  |
| ------ | ------- |
| Flag 1 | ✅ Found |
| Flag 2 | ✅ Found |
| Flag 3 | ✅ Found |
| Flag 4 | ✅ Found |

**Total: 4/4 Flags**

📸 Screenshot: Add your flag collection screenshots here.

---

# 🗄️ Phase 5 – WordPress Database Investigation

## Step 9 – WordPress Configuration Investigation

I continued investigating the WordPress installation and its configuration files.

The WordPress configuration file is:

```text
/var/www/html/wordpress/wp-config.php
```

To navigate to the directory:

```bash
cd /var/www/html/wordpress
```

Inspect the configuration file:

```bash
cat wp-config.php
```

This file contains important WordPress configuration settings, including database connection information.

**Security Observation:**

Exposed configuration files can reveal sensitive information, including database credentials.

📸 Screenshot: Add your configuration investigation screenshot here. Hide passwords and other secrets.

---

## Step 10 – MySQL Database Investigation

After identifying the database configuration information, I investigated the WordPress database.

**Command:**

```bash
mysql -u root -p
```

### Command Explanation

- `mysql` – MySQL command-line client.
- `-u root` – Specifies the database username.
- `-p` – Prompts for the database password.

After successfully connecting, I selected the WordPress database.

```sql
SHOW DATABASES;
```

Select the database:

```sql
USE wordpress;
```

List the available tables:

```sql
SHOW TABLES;
```

The WordPress database contained tables storing application information, including user accounts and posts.

### Important Tables

```text
wp_users
wp_posts
```

📸 Screenshot: Add your MySQL database and table screenshots here.

---

## Step 11 – Flag Investigation in WordPress Database

I continued investigating the WordPress database and examined the `wp_posts` table.

**Command:**

```sql
SELECT * FROM wp_posts;
```

This query retrieves records stored in the WordPress posts table.

I examined the available records and identified information related to the remaining flags.

### Important Findings

- Investigated WordPress post records.
- Examined stored application content.
- Identified the remaining flag information.
- Successfully completed the four-flag objective.

**Result:**

All four flags were successfully captured.

📸 Screenshot: Add your actual database findings here. Hide flag values if you don't want to publish spoilers.

---

## Step 12 – WordPress User Credential Investigation

Next, I examined the WordPress user table to investigate stored user credentials.

**SQL Command:**

```sql
SELECT user_login, user_pass FROM wp_users;
```

This query retrieves usernames and their stored password hashes.

### What is a Password Hash?

WordPress stores password hashes instead of directly storing users' plaintext passwords.

A password hash is produced using a one-way cryptographic hashing process.

The stored hash can be investigated through authorized password-auditing tools.

### Saving a Hash for Investigation

I used a text file to store the relevant hash for local analysis.

```bash
nano hash.txt
```

After saving the hash, I used John the Ripper to perform password auditing.

**Command:**

```bash
john hash.txt
```

To display recovered results:

```bash
john --show hash.txt
```

### Command Explanation

- `john` – John the Ripper password-auditing tool.
- `hash.txt` – File containing the password hash.
- `--show` – Displays previously recovered password results.

The credential investigation helped me obtain the password associated with the `steven` account.

**Note:** Do not publish recovered passwords or reusable credentials.

📸 Screenshot: Add your hash investigation screenshots here, with sensitive information hidden.

---

# 👑 Phase 6 – Privilege Escalation

## Step 13 – User Privilege Investigation

After obtaining the credentials for `steven`, I investigated the user's permissions and available privileged commands.

First, I switched to the user account:

```bash
su steven
```

After entering the password, I checked the current identity:

```bash
whoami
```

Then I checked the user's permissions:

```bash
id
```

I also investigated the system's sudo configuration:

```bash
sudo -l
```

### Why is sudo -l Important?

The `sudo -l` command displays which commands the current user is permitted to execute with elevated privileges.

This is important during Linux privilege enumeration because incorrectly configured sudo permissions can expose a system to privilege escalation.

### Additional Local Enumeration Commands

Check operating system information:

```bash
cat /etc/os-release
```

Check kernel information:

```bash
uname -a
```

Check current hostname:

```bash
hostname
```

Search for files with SUID permissions:

```bash
find / -perm -4000 -type f 2>/dev/null
```

These commands help investigate the system's configuration and possible privilege escalation weaknesses.

📸 Screenshot: Add your actual `sudo -l` output here.

---

## Step 14 – Root Access

After investigating the available permissions, I identified a privilege escalation opportunity and successfully obtained root-level access.

**The exact sudo permission and escalation command should be documented from your original lab output.** I don't have enough evidence to state the exact executable or reproduce the precise exploitation sequence accurately.

### Verify Root Access

After obtaining elevated access, I verified the current user:

```bash
whoami
```

Expected output:

```text
root
```

Check user identity:

```bash
id
```

Navigate to the root directory:

```bash
cd /root
```

List its contents:

```bash
ls -la
```

At this point, the final objective was achieved.

\<details>
\<summary>👑 Root Access Achievement\</summary>

**Status:** Successfully Rooted

**Initial User:** steven

**Final User:** root

**Flags:** 4/4

\</details>

📸 Screenshot: Add your actual root terminal screenshot here.

---

# 🛠️ Tools Used

| Tool            | Purpose                               |
| --------------- | ------------------------------------- |
| Kali Linux      | Penetration-testing operating system  |
| VirtualBox      | Virtual lab environment               |
| Netdiscover     | Network discovery                     |
| Nmap            | Port scanning and service enumeration |
| RPCinfo         | RPC service enumeration               |
| WhatWeb         | Website technology identification     |
| Gobuster        | Directory enumeration                 |
| WPScan          | WordPress enumeration                 |
| Hydra           | SSH credential testing                |
| SSH             | Remote system access                  |
| MySQL           | Database investigation                |
| John the Ripper | Password hash auditing                |
| Linux CLI       | System and privilege enumeration      |

---

# 🧠 Skills Gained

Through this practical challenge, I improved my understanding of:

- Network reconnaissance.
- IP address discovery.
- TCP port scanning.
- Service and version detection.
- Web application enumeration.
- WordPress security investigation.
- SSH authentication.
- Linux filesystem navigation.
- MySQL database investigation.
- Password hash auditing.
- Linux user and permission management.
- Sudo permission enumeration.
- Privilege escalation concepts.
- CTF methodology and problem-solving.

---

# 🛡️ Remediation & Security Recommendations

The security concepts demonstrated by this challenge highlight the importance of the following practices:

**1. Strong Authentication**

Use strong passwords and enforce secure authentication policies.

**2. Secure SSH Configuration**

Restrict SSH access and disable unnecessary authentication methods.

**3. Secure Configuration Files**

Protect sensitive application configuration files from unauthorized access.

**4. Database Security**

Use dedicated database accounts with limited permissions instead of unnecessary administrative privileges.

**5. Password Security**

Store passwords using secure password-hashing algorithms and appropriate configurations.

**6. Least Privilege Principle**

Regular users should only have the permissions required for their tasks.

**7. Sudo Configuration**

Regularly review sudo permissions and remove unnecessary privileged command access.

**8. Patch Management**

Keep operating systems, web servers, and applications updated.

---

# 📚 Final Conclusion

Successfully completing Reven was an important milestone in my cybersecurity learning journey.

Throughout this challenge, I practiced different stages of a penetration-testing assessment, beginning with network discovery and continuing through service enumeration, web investigation, initial access, database investigation, flag collection, and privilege escalation.

The challenge helped me understand how different tools and techniques work together during a real-world-style security assessment.

Most importantly, I successfully:

- 🔍 Identified the target machine.
- 📡 Enumerated exposed services.
- 🌐 Investigated the web application.
- 🔐 Obtained initial SSH access.
- 🚩 Captured all four flags.
- 👑 Achieved root access.

**Machine Status: PWNED ✅**

---

# ⚠️ Disclaimer

This documentation is intended for educational purposes only.

All testing was performed against an intentionally vulnerable machine in an isolated virtual lab environment.

Never perform penetration testing against systems without explicit authorization.

---

# 👨‍💻 About Me

I'm a cybersecurity beginner currently developing my practical skills in ethical hacking, penetration testing, and Linux security.

### My Technical Interests

- 🔐 Ethical Hacking
- 🌐 Web Application Security
- 🐧 Linux & Kali Linux
- 🔍 Network Reconnaissance
- 🛡️ Vulnerability Assessment
- 🧪 CTF & Boot2Root Challenges
- 📡 Network Security

My goal is to continuously improve my technical knowledge through hands-on labs, practical exercises, and documenting my cybersecurity learning journey.

**Learning | Practicing | Documenting | Improving 🚀**

\<p align="center">
&#x20; \<b>One Machine at a Time. One Skill at a Time. 🔥\</b>
\</p>
