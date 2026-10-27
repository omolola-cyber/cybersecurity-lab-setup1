# 🖥️ Cybersecurity & Pentesting Lab Setup

**Networkwalks | Project 1 | Tools: VirtualBox + Kali Linux**

---

## 📌 Project Overview

This project sets up a **virtual cybersecurity and penetration-testing laboratory** using VirtualBox and Kali Linux.

The lab gives me a controlled place to practice security tools, network scanning, and reconnaissance without touching my main operating system or home network.

The lab runs on a private virtual network, so more machines can be added later and used as targets for authorized security testing.

---

## 🎯 Objectives

- Install and configure VirtualBox.
- Import Kali Linux as a virtual machine.
- Create a private **NAT Network** for the lab.
- Attach Kali Linux to that network.
- Assign Kali a static IP address.
- Verify connectivity, DNS resolution, and tool installation.
- Take a clean VM snapshot for recovery.
- Document the full setup, including the problems I hit.

---

## 🛡️ Purpose of the Lab

The lab can be used for activities such as:

- Network reconnaissance
- Port scanning
- Vulnerability assessment
- Packet analysis
- Web security testing
- Security-tool experimentation

> **Note on isolation:** A NAT Network keeps the lab separate from my host network, but Kali can still reach the Internet through it. For testing that needs full isolation (for example, analyzing malware), a separate Internal Network with no Internet access should be used.

⚠️ **Important:** This lab must only be used on systems I own or have explicit permission to test.

---

## 🏗️ Lab Architecture


Additional target machines can be added to the same virtual network in future projects.

---

## ⚙️ Lab Configuration

| 🧩 Component       | ⚙️ Configuration   |
| ----------------- | ------------------ |
| 🖥️ Host OS        | Windows 10         |
| 🧠 Host RAM        | 8 GB               |
| ⚡ Processor       | Intel Core i5      |
| 🧰 Hypervisor      | VirtualBox 7.0.20  |
| 🐉 Security OS     | Kali Linux 2026.2  |
| 🧠 Kali RAM        | 2048 MB            |
| 🌐 Virtual Network | NAT Network (`NatNetwork`) |
| 📡 Network Address | 10.0.0.0/24        |
| 🐧 Kali IP Address | 10.0.0.2/24 (static) |
| 🚪 Default Gateway | 10.0.0.1           |
| 🌍 DNS Server      | 8.8.8.8            |

---

# 🪜 Lab Setup Procedure

## Step 1. Install 7-Zip

7-Zip was used to extract the Kali Linux virtual-machine package, which was downloaded as a `.7z` archive.

---

## Step 2. Install VirtualBox

VirtualBox was installed as the hypervisor.

---

## Step 3. Create the NAT Network

A dedicated NAT Network was created in VirtualBox under **File → Tools → Network Manager → NAT Networks**.

```
Network Name: NatNetwork
IPv4 Prefix:  10.0.0.0/24
DHCP:         Enabled
IPv6:         Disabled
```


VirtualBox suggests `10.0.2.0/24` by default, so the prefix was changed to `10.0.0.0/24` and applied.

A **NAT Network** was chosen because virtual machines on the same NAT Network can talk to each other while still having outbound access. This lets future attacker and target VMs communicate inside the lab.

<img width="1070" height="576" alt="nat-network-settings" src="https://github.com/user-attachments/assets/40f35d66-c5cf-469e-bc4d-f0388c26137a" />

---

## Step 4. Attach Kali Linux to the NAT Network

With the Kali VM powered off, its network adapter was configured under **Settings → Network → Adapter 1**:

```
Attached to:  NAT Network
Name:         NatNetwork
Adapter Type: Intel PRO/1000 MT Desktop
RAM:          2048 MB
```


After starting Kali, DHCP assigned it `10.0.0.4`, which confirmed it had joined the new network.

---

## Step 5. Configure a Static IP in Kali

The connection name was found with:

```
nmcli connection show
```

The connection was `Wired connection 1` on `eth0`. The static address was then set and applied:

```
sudo nmcli con mod "Wired connection 1" ipv4.method manual ipv4.addresses 10.0.0.2/24 ipv4.gateway 10.0.0.1 ipv4.dns 8.8.8.8
sudo nmcli con up "Wired connection 1"
```

A consistent IP makes the lab easier to document and reference in later exercises.


---

## Step 6. Create a Clean VM Snapshot

With Kali powered off, a snapshot was taken from the VirtualBox Snapshots tab and named:

```
Clean Kali - Network Setup
```

This is the clean baseline of the lab. If a later exercise breaks the VM, I can restore it to this point.

<img width="1193" height="626" alt="snapshot" src="https://github.com/user-attachments/assets/e409a67a-cfed-4bda-88e5-a4caa31f20be" />

---

# 🔎 Lab Verification

| ✅ Test                       | 🧾 Command                  | 🎯 Result                                   |
| ---------------------------- | -------------------------- | ------------------------------------------ |
| 🌐 Check IP address           | `ip a`                     | `eth0` shows `10.0.0.2/24` (static)        |
| 📡 Test gateway               | `ping -c 3 10.0.0.1`       | 3 sent, 3 received, 0% packet loss         |
| 🌍 Test Internet connectivity | `ping -c 3 8.8.8.8`        | 3 sent, 3 received, 0% packet loss         |
| 🔎 Test DNS resolution        | `nslookup networkwalks.com`| Resolved using server `8.8.8.8`            |
| 🧰 Verify Nmap                | `nmap --version`           | Nmap 7.94 installed                        |
| 📸 Snapshot                   | VirtualBox Snapshots       | Clean baseline snapshot saved              |

<img width="1280" height="576" alt="WhatsApp Image 2026-10-27 at 5 06 42 AM (1)" src="https://github.com/user-attachments/assets/2ab471b3-d240-4d99-ac41-b0d88ca1cc67" />

---

# 🐞 Problems Encountered & Solutions

Documenting problems is an important part of the project.

## Problem 1. Wrong default network prefix

The NAT Network was first created with VirtualBox's default prefix, `10.0.2.0/24`, which did not match the lab plan.

**Fix:** I opened the network's Properties, changed the IPv4 prefix to `10.0.0.0/24`, and clicked **Apply**.

## Problem 2. Static IP did not take effect

After running `nmcli con mod`, `ip a` still showed the old DHCP address (`10.0.0.4`, marked `dynamic`) and DNS was still coming from DHCP.

**Cause:** `nmcli con mod` only saves the settings. The connection has to be restarted to apply them.

**Fix:** Running `sudo nmcli con up "Wired connection 1"` applied the static IP. `ip a` then showed `10.0.0.2/24` and `nslookup` used `8.8.8.8`.

## Problem 3. Connection name is case-sensitive

`nmcli con show "wired connection 1"` returned *no such connection profile*.

**Fix:** The name must match exactly, including the capital letter: `"Wired connection 1"`.

> Connection names can differ between systems, so always run `nmcli connection show` first.

---

# 💡 What I Learned

### 1. NAT vs NAT Network
A standard NAT gives each VM its own private connection to the Internet. A NAT Network lets multiple VMs on the same virtual network talk to each other while still reaching the outside, which suits a multi-machine lab.

### 2. Virtual Machine Networking
How VirtualBox network adapters attach a VM to different network types, and how that choice affects what the VM can reach.

### 3. Static IP Configuration
How to set and verify an IPv4 address, gateway, and DNS server with `nmcli`, and why settings must be applied (`nmcli con up`) before they take effect.

### 4. VM Snapshots
A clean snapshot should be taken **before** risky or experimental work, so there is always a known-good recovery point.

### 5. Documentation
Recording commands, configuration, screenshots, and mistakes is a core part of a professional cybersecurity project.

---

# 🔐 Security & Ethical Use

This laboratory is intended strictly for educational purposes. Only test systems you own or have explicit permission to test.

---

# 🔗 Tools & Resources

- **7-Zip:** <https://7-zip.org/download.html>
- **VirtualBox:** <https://virtualbox.org/wiki/Downloads>
- **Kali Linux:** <https://kali.org/get-kali>

---

# 👤 Author

**Kolawole Asmot Omowumi**  
Cybersecurity Intern  
**Networkwalks**

GitHub: <https://github.com/omolola-cyber>  
LinkedIn: https://www.linkedin.com/in/kolawole-omolola-8490033a7

---

## 📌 Project Information

**Internship:** Cybersecurity Internship  
**Week:** 01  
**Project:** Cybersecurity & Pentesting Lab Setup  
**Organization:** Networkwalks  
**Repository:** GitHub
