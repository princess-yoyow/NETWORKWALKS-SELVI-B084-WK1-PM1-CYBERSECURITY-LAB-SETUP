# 🔐 Cybersecurity Lab Environment Setup

**Building an isolated virtual lab for penetration testing and ethical hacking practice**

![Cybersecurity](https://img.shields.io/badge/Cybersecurity-Lab-red)
![Ethical Hacking](https://img.shields.io/badge/Ethical%20Hacking-Practice-darkred)
![VMware](https://img.shields.io/badge/VMware-Workstation-607078)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-Attacker%20VM-557C94)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-blue)
![NAT](https://img.shields.io/badge/VMnet8-NAT-informational)
![Report](https://img.shields.io/badge/Report-PDF-orange)
![Author](https://img.shields.io/badge/Author-Selvi%20Melani-success)

---

## 📌 Project Overview

This project sets up an isolated cybersecurity lab on my own laptop, with Kali Linux as the attacking machine. Kali runs on a private NAT network so that all testing stays inside the lab, while still having Internet access for updates and tools. The lab is the base for the next weeks of the Networkwalks Cybersecurity Internship, where target machines will join the same network.

---

## 🎯 Objectives

- Set up a hypervisor as the base of the lab (VMware Workstation, mapped to the VirtualBox requirements)
- Deploy Kali Linux as the attacking machine
- Create a NAT network in the 10.0.0.0/24 subnet
- Assign Kali the static address 10.0.0.2/24
- Give Kali full Internet access through the NAT gateway
- Enable shared clipboard and drag-and-drop
- Share the host's Downloads folder with the VM
- Document the setup and verify every requirement

---

## 🛡️ Purpose of the Lab

- Reconnaissance and information gathering practice
- Network and port scanning
- Packet capture and traffic analysis
- Vulnerability assessment of lab targets
- Safe practice of attack and defence techniques

> ⚠️ All activities are performed only inside this lab, on systems I own. No external systems are targeted.

---

## 🏗️ Lab Architecture

```
            Internet
                │
   ┌────────────┴────────────┐
   │   Host laptop (Windows) │
   │   VMware NAT gateway    │
   │        10.0.0.1         │
   └────────────┬────────────┘
                │   VMnet8 (NAT) · 10.0.0.0/24
                │   DHCP pool: 10.0.0.128 – 10.0.0.254
   ┌────────────┴────────────┐
   │       Kali Linux        │
   │   10.0.0.2/24 (static)  │
   └─────────────────────────┘
```

Windows and Android target machines will be added to the same network in Phase 2 (static range 10.0.0.2 – 10.0.0.99).

---

## ⚙️ Lab Configuration

| Component | Configuration |
| --- | --- |
| Host OS | Windows |
| Hypervisor | VMware Workstation |
| Attacker VM | Kali Linux (installed from ISO) |
| VM resources | 6 GB RAM · 4 vCPU · 40 GB disk |
| Network type | VMnet8 (NAT) |
| Subnet | 10.0.0.0/24 |
| Gateway | 10.0.0.1 |
| Kali IP | 10.0.0.2/24 (static) |
| DNS | 8.8.8.8 |
| DHCP pool | 10.0.0.128 – 10.0.0.254 |
| Future VMs | 10.0.0.3 – 10.0.0.99 |

### VirtualBox → VMware mapping

| Task requirement (VirtualBox) | Equivalent used (VMware Workstation) |
| --- | --- |
| NAT Network 10.0.0.0/24 | VMnet8 (NAT) set to 10.0.0.0/24 |
| Adapter attached to NAT Network | Network Adapter → Custom: VMnet8 (NAT) |
| Shared Clipboard & Drag'n'Drop | Options → Guest Isolation |
| Shared Folders | Options → Shared Folders (`/mnt/hgfs/Downloads` in Kali) |

---

# 📄 Lab Setup Report

The full setup procedure, with every configuration step and screenshot, is documented in the report:

### 👉 [WK1-PM1_Lab_Setup_Report_Selvi_Melani.pdf](WK1-PM1_Lab_Setup_Report_Selvi_Melani.pdf)

The report covers:

1. Creating the Kali Linux VM
2. Configuring the NAT network (VMnet8) in 10.0.0.0/24
3. Checking the DHCP pool
4. Setting the NAT gateway to 10.0.0.1
5. Attaching the Kali adapter to VMnet8
6. Setting Kali's static IP 10.0.0.2/24
7. Enabling clipboard and drag-and-drop
8. Sharing the host Downloads folder

---

# 🔎 Lab Verification

| Test | Command | Result |
| --- | --- | --- |
| IP address | `ip -br addr` | ✅ eth0 UP, 10.0.0.2/24 |
| Default route | `ip route` | ✅ default via 10.0.0.1 |
| Gateway | `ping -c 4 10.0.0.1` | ✅ Replies received |
| Internet | `ping -c 4 8.8.8.8` | ✅ Replies received |
| DNS | `ping -c 4 google.com` | ✅ Name resolved, replies received |
| Shared folder | `ls /mnt/hgfs/Downloads` | ✅ Host files listed |
| Clipboard | Copy text host ↔ Kali | ✅ Works both ways |
| Drag-and-drop | Drag a file from host into Kali | ✅ File copied into the VM |

### Example Results

```
IP Address : 10.0.0.2/24
Gateway    : 10.0.0.1
DNS        : 8.8.8.8
Interface  : eth0 (UP)
```

---

# 🐞 Problems Encountered & Solutions

## Problem 1. Using VMware instead of VirtualBox

The task was written for VirtualBox, but the lab was built on VMware Workstation. The two hypervisors use different names for the same features.

**Solution:** each requirement was mapped to its VMware equivalent (see the mapping table above): VMnet8 (NAT) for the NAT Network, Guest Isolation for clipboard and drag-and-drop, and Shared Folders mounted under `/mnt/hgfs`.

## Problem 2. Gateway address conflict with Kali's IP

By default, VMware's NAT gateway uses the `.2` address of the subnet, which is the same address the task requires for Kali (10.0.0.2).

**Solution:** in Virtual Network Editor → VMnet8 → NAT Settings, the gateway was changed to `10.0.0.1`. Kali was then configured with `nmtui` and the connection reactivated:

```bash
sudo nmcli connection up "Wired connection 1"
```

> If Internet still does not work after a static IP change, the course fix for newer Kali versions is:
>
> ```bash
> sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
> sudo nmcli connection down "Wired connection 1"
> sudo nmcli connection up "Wired connection 1"
> ```

---

# 💡 What I Learned

### NAT networking in VMware
VMnet8 works like VirtualBox's NAT Network: VMs share a private subnet and reach the Internet through the host.

### Static IP vs DHCP
Keeping static lab addresses below the DHCP pool prevents IP conflicts when more VMs are added.

### Gateway planning
Default settings do not always fit the design; the gateway had to move to free the address required for Kali.

### Host–guest integration
Clipboard, drag-and-drop and shared folders make moving tools and results between host and VM much easier.

### Documentation
Writing down each setting makes the lab easy to rebuild and helps others with the same issues.

---

# 🔐 Security & Ethical Use

This lab is for educational purposes only. All testing is done inside my own isolated lab environment, never against systems without written permission.

---

# 🔗 Tools & Resources

- [VMware Workstation](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)
- [Kali Linux](https://www.kali.org/get-kali/)
- [Networkwalks](https://networkwalks.com)

---

# 👤 Author

**Selvi Melani**
Cybersecurity Intern, Networkwalks
[LinkedIn](https://www.linkedin.com/in/YOUR-LINKEDIN-USERNAME)

## 📌 Project Information

| | |
| --- | --- |
| Program | Networkwalks Cybersecurity Internship |
| Batch | B084 |
| Week | 1 |
| Project | WK1-PM1 – Cybersecurity Lab Setup |
| Repository | NETWORKWALKS-SELVI-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP |
