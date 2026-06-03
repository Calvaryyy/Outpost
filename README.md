# Outpost

An independent, full-chain Red Teaming and Adversary Emulation home lab environment built entirely from scratch.

---

## The Backstory: Why I Built This

I wanted to practice my red team skills in a truly realistic environment, so I tried simply installing GOAD. It just didn't work, from one error to another. After messing around with broken automation, I finally just said, "Lemme just do it myself." And viola—we are here. 

The entire purpose of this Active Directory environment is to provide a realistic scenario for people to practice full-chain red teaming. Although it makes use of relatively weak passwords and there is a very minimum configuration present right now (there isn't a single file share set up yet), it gives you a proper, unweakened corporate baseline. 

Windows Defender isn't turned off, there isn't any intentional weakening of the operating systems, and because it's pretty basic, there are considerably fewer chances of a weird automated misconfiguration breaking your tools. The email addresses are fully active, which means your initial access vector is phishing—you literally have to phish your way into the network, establish a beachhead, evade live defenses, move laterally, and escalate your privileges to become Domain Controller or achieve whatever your APT objective is. 

---

## Lab Architecture & Topology

This environment consists of three interconnected virtual machines deployed on **Oracle VirtualBox** via a unified `.ova` appliance. 

| VM Name | OS | Hostname / Role | Network Mode | Default Setup |
| :--- | :--- | :--- | :--- | :--- |
| **Windows 2025 Server** | Windows Server 2025 | `DC-01` (Domain Controller) | Bridged | Static IP: `192.168.1.100` / DNS: `127.0.0.1` |
| **Active Directory Machine 1 (Windows 10)** | Windows 10 | `John` (Workstation) | Bridged | Primary DNS points to DC |
| **Active Directory Machine 2 (Windows 10)** | Windows 10 | Workstation | Bridged | Primary DNS points to DC |

### Network Configuration & Customization
By default, the virtual machines are exported using **Bridged Adapter** settings with the Domain Controller configured to a static IP of `192.168.1.100`. 

To make the lab work seamlessly in your personal environment:
* Adjust the IP addresses of the VMs inside Windows to match your own home local gateway subnet.
* **Crucial Rule:** You **must** ensure that the **Primary DNS Server** field on both Windows 10 client machines is manually set to the exact IP address of the Domain Controller (`192.168.1.100` or your customized DC IP). Otherwise, the clients will not be able to locate the domain or route internal mail.

### Defensive Monitoring & Logging
* **Windows Defender:** Fully active across all nodes. No artificial weakening or exclusion rules have been introduced.
* **Sysmon (System Monitor):** Installed and active across all 3 VMs to provide comprehensive endpoint logging for blue team analysis, log hunting, and operational telemetry.

---

## Identity & Access Management (Credentials Matrix)

### Active Directory Infrastructure
* **Domain Name:** `DC-01.local`
* **NetBIOS Domain Name:** `DC-010`
* **DSRM (Directory Services Restore Mode) Password:** `Adm1n$tr@tor24`

### Account Database

| Target Persona | Account Type | UPN / Domain Username | Password | Notes / Context |
| :--- | :--- | :--- | :--- | :--- |
| **Domain Administrator** | Enterprise Admin | `DC-01\Administrator` | `Ad1m$tr@tor24` | Full Domain Admin control. |
| **IT Admin** | Non-Domain Admin Group | `itadmin@DC-01.local` | `Password123!@` | Local administrator privileges on endpoints. |
| **John Doe** | Regular User (Machine 1) | `jdoe@DC-01.local` | `S3cur3Admin!` (Same password for Domain and Local) | Logged into `Active Directory Machine 1`. |
| **Joe Smith** | Regular User (Machine 2) | `jsmith@DC-01.local` | `Dallas2024@` *(Domain)*<br>`Contoso2024#` *(Local)* | Logged into `Active Directory Machine 2`. |
| **SQL Service** | Service Account | `svc_sql@DC-01.local` | `P@S$w0rd!` | **Has an active SPN configured** for Kerberoasting practice. |

---

## Mail Infrastructure (Initial Access Perimeter)

To facilitate realistic **Initial Access Simulation**, a functional email ecosystem is integrated directly into the core of the domain.

* **Mail Server Engine:** hMailServer installed locally on the Domain Controller (`DC-01`).
* **hMailServer Administrator Password:** `hMail2024!`
* **Mail Clients:** Mozilla Thunderbird installed on both Windows 10 Workstations.
* **Active Mailboxes:**
  * `jdoe@DC-01.local` (Password matches Domain credential: `S3cur3Admin!`)
  * `jsmith@DC-01.local` (Password matches Domain credential: `Dallas2024@`)

---
## Download Virtual Machines

Refer to Download-Links.txt to find the download links for the virtual machines

## Target Attack Path & Learning Objectives

Because this range doesn't rely on obvious vulnerabilities or disabled defenses, operators must utilize foundational Active Directory exploitation and post-exploitation methodologies:

1. **Initial Access via Spear-Phishing:** Utilize the operational hMailServer ecosystem to deliver payloads or credential-harvesting links from one internal persona to another via Thunderbird.
2. **Local Reconnaissance & Evasion:** Land a low-privileged session (`jdoe` or `jsmith`) and execute local enumeration safely under active Windows Defender restrictions.
3. **Active Directory Enumeration:** Query the Active Directory infrastructure to map domain objects, trust groups, and high-value internal targets.
4. **Lateral Movement & Domain Escalation:** Move horizontally through the network or exploit service account relationships—such as executing a **Kerberoasting attack** against `svc_sql` to extract and crack its TGS ticket—to pivot upward and achieve complete Domain Controller administrative dominance.
