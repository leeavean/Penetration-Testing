# 🛡️ Penetration Test Report: Metasploitable 2 Infrastructure
## 📝 Executive Summary
This directory contains the comprehensive security assessment documentation for the Internet-facing systems of Security HolesRUs, conducted by Pentest4Peanuts. The engagement uncovered a total of 15 vulnerabilities across two target hosts, highlighting severe risks to business operations including unauthenticated remote code execution and widespread credential exposure.
* Assessment Methodology: Penetration Testing Execution Standard (PTES)
## 🎯 Target Scope
The assessment was confined strictly to the following assets within the local lab infrastructure:
| **Target IP**   | **Operating System**   | **Primary Role / Identifiers**            | **Status**              |
| --------------- | ---------------------- | ----------------------------------------- | ----------------------- |
| `192.168.2.186` | Linux (Debian 4.x/5.x) | Apache HTTP / CGI (Hostname: THELONIUS)   | **Fully Compromised**   |
| `192.168.2.192` | Windows Server         | IIS 10.0 / MSSQL 2022 (Hostname: BRISEIS) | **Partially Exploited** |
### 🛑 Rules of Engagement
* Metasploit Restriction: Exactly one use of the Metasploit framework was permitted during the entire engagement (reserved exclusively for the Windows target environment).
* Destructive Testing: Actions resulting in a Denial-of-Service (DoS) or service crashes were strictly prohibited.
## 📊 Findings Dashboard
### Vulnerability Breakdown
| **Severity** | **Overall count** | **Unique Count** | **Key Exploits / Findings**                          |
| ------------ | ----------------- | ---------------- | ---------------------------------------------------- |
| `🔴 High`    | 4                 | 3                | Shellshock (RCE), Unauthenticated SMB, Anonymous FTP |
| `🟡 Medium`  | 6                 | 4                | Null Session RPC, Exposed MSSQL, Outdated Apache     |
| `🔵 Low`     | 5                 | 3                | Default Page Disclosure, Web File Discovery Attempts |

### 🏳️ Captured Flags

Each machine contained three hidden flags (`flag_easy.txt`, `flag_intermediate.txt`, `flag_hard.txt`) serving as proof of exploitation.

1. **Linux (`192.168.2.186`)**: 🟩 Easy (Exfiltrated via Shellshock exploit string) | 🟨 Intermediate (Pending) | 🟥 Hard (Pending)

2. **Windows (`192.168.2.192`)**: 🟩 Easy (Retrieved via anonymous SMB2 Public share) | 🟨 Intermediate (Pending) | 🟥 Hard (Pending)

🚀 Key Attack Path & Proof of Concept (PoC)
-------------------------------------------

### 1. Linux Remote Code Execution (Shellshock)

Initial service enumeration revealed an Apache web server hosting a `/cgi-bin/` directory. The host was confirmed vulnerable to the **Shellshock flaw (CVE-2014-6271)**.

The vulnerability was leveraged to execute arbitrary system commands via `nmap` script arguments to read data directly from the backend file structure:

nmap -p80 --script http-shellshock --script-args uri=/cgi-bin/test,cmd="cat /srv/ftp/flag_easy.txt" 192.168.2.186

### 2. Windows SMB Null Session Access

The Windows target misenforced share restrictions. By bypassing default protocol negotiation constraints and specifying an older minimum dialect, unauthenticated guest access was established to dump user directories:

smbclient //192.168.2.192/Public -N --option='client min protocol=SMB2'

🛠️ Critical Remediation Actions
--------------------------------

For complete mitigation details, please refer to **Section 4 & 7** of `LeevanAhmed-PTD-Part2-Report.pdf`. Immediate high-priority recommendations include:

1. **Patch Bash Vulnerabilities:** Upgrade `bash` on the Linux asset to resolve the Shellshock vulnerability.

2. **Disable Insecure Null Sessions:** Restrict anonymous or unauthenticated access across all network shares (SMB Public) and RPC end-points on the Windows domain host.

3. **Enforce Network ACLs:** Tighten network boundaries to isolate port 1433 (MSSQL) and port 5985 (WinRM) from public exposure.
