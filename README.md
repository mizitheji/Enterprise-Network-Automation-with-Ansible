# Enterprise-Network-Automation-with-Ansible

## 📌 Overview
This project demonstrates **enterprise-grade network automation** using **Ansible**, designed around a realistic **HQ + Branch + ISP topology**.

The automation focuses on:
- Secure configuration backups
- SNMP configuration deployment
- SSH security compliance validation
- OSPF neighbor health checks
- Centralized compliance reporting

The Ansible Control Node is hosted on **Proxmox at HQ**, reflecting real-world data center practices.

---

## 🏗 Network Topology

### 🏢 HQ
- **Cisco C8300** – WAN Edge
- **Cisco C9300** – Core & Access Switch
- **Proxmox** – Hosts Ansible Control Node (Ubuntu VM)

### 🏬 Branch 1
- **MikroTik Router** – Core & WAN Edge
- **Cisco C9300L** – Access Switch

### 🏬 Branch 2
- **Cisco C9300** – WAN Edge, Core & Access

### 🌐 ISP
- **Cisco C8300**
- Provides connectivity between all sites using **OSPF**

### OSPF WAN Links

| Site     | Local IP | Subnet      | Remote IP | Remote Site |
|----------|----------|-------------|-----------|-------------|
| HQ       | 10.0.0.1 | 10.0.0.0/30 | 10.0.0.2  | ISP         |
| Branch 1 | 10.0.0.5 | 10.0.0.4/30 | 10.0.0.6  | ISP         |
| Branch 2 | 10.0.0.9 | 10.0.0.8/30 | 10.0.0.10 | ISP         |

### ISP To Internet

| Site      | Local IP     | Subnet          | Remote IP     | Remote Site |
|-----------|--------------|-----------------|---------------|-------------|
| ISP       | 10.255.255.2 | 10.255.255.0/30 | 10.255.255.1  | INTERNET    |

### HQ VLAN Design

| VLAN ID | VLAN Name | Purpose           | Subnet          |
|---------|-----------|-------------------|-----------------|
| 1       | Mgmt      | Device Management | 172.20.1.0/24   |
| 10      | Wired     | Wired Users       | 192.168.11.0/24 |
| 20      | Wireless  | Wireless Clients  | 172.30.30.0/24  |
| 40      | Server    | Server Network    | 192.168.40.0/24 |

### Branch 1 VLAN Design

| VLAN ID | VLAN Name | Purpose           | Subnet          |
|---------|-----------|-------------------|-----------------|
| 1       | Mgmt      | Device Management | 172.20.2.0/24   |
| 10      | Wired     | Wired Users       | 192.168.12.0/24 |

### Branch 2 VLAN Design

| VLAN ID | VLAN Name | Purpose           | Subnet          |
|---------|-----------|-------------------|-----------------|
| 1       | Mgmt      | Device Management | 172.20.3.0/24   |
| 10      | Wired     | Wired Users       | 192.168.13.0/24 |
> All sites participate in OSPF area 0 via point-to-point /30 WAN links through the ISP core router.  
> Internal VLAN subnets are advertised into OSPF for full inter-site reachability.

---

## 🧠 Architecture Concept

![diagrams](diagrams/diagrams.jpg)

---

## 📁 Repository Structure
```structure
Network-Automation-With-Ansible
├── inventory/
│ ├── hosts.yml
│ └── group_vars/
│   ├── all/
│   | └── vault.yml # Encrypted credentials
│   ├── cisco.yml
│   └── mikrotik.yml
├── playbooks/
| ├── roles/
│ ├── cisco_compliance/
| | └── tasks/
| |  └── main.yml
│ ├── mikrotik_compliance/
| | └── tasks/
| |   └── main.yml
│ ├── backup.yml # Config backups
│ ├── snmp.yml # SNMP deployment
│ ├── compliance.yml # SSH compliance check
│ ├── ospf_check.yml # OSPF neighbor validation
│ └── compliance_report.yml # CSV compliance report
├── roles/
│ └── cisco_ospf/
|   └── tasks/
|     └── main.yml
├── templates/
│ ├── cisco_snmp.j2
│ └── mikrotik_snmp.j2
├── backups/
│ ├── cisco/
│ └── mikrotik/
├── reports/
│ └── compliance_report.csv
├── diagrams/
│ └── diagrams.jpg
└── README.md
```

---

## ⚙️ Prerequisites

### Ansible Control Node
- Ubuntu 22.04 / 24.04 (VM on Proxmox)
- Python 3.10+
- Ansible Core

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv -y
sudo git clone https://github.com/mizitheji/Enterprise-Network-Automation-with-Ansible
cd Enterprise-Network-Automation-with-Ansible
python3 -m venv .venv
source .venv/bin/activate
pip install ansible
```

### Install System Dependencies
These packages are required to build **pylibssh**. **Install in .venv environment.**
```bash
sudo apt install -y libssh-dev python3-dev build-essential
pip install ansible-pylibssh
```

### Required Ansible Collections
```bash
ansible-galaxy collection install cisco.ios
ansible-galaxy collection install community.routeros
```
To verify installed collection:
```bash
ansible-galaxy collection list
```

---

## 🔐 Secrets Management (Ansible Vault)
All device credentials are stored securely using Ansible Vault.
```bash
sudo ansible-vault encrypt group_vars/all/vault.yml
```
Run playbooks with:
```bash
ansible-playbook -i inventory/hosts.yml playbooks/backup.yml --ask-vault-pass
```

---

## 💾 Configuration Backup
### Cisco Devices
- Backup uses **show running-config** only

### MikroTik Devices
- Full configuration backup using **/export verbose**
```bash
ansible-playbook -i inventory/hosts.yml playbooks/backup.yml --ask-vault-pass
```

---

## 📡 SNMP Configuration Deployment
Deploys standardized SNMP settings across **Cisco** and **MikroTik** devices.
```bash
ansible-playbook -i inventory/hosts.yml playbooks/snmp.yml --ask-vault-pass
```
Features:
- Unified SNMP community
- Site-based location tagging
- NOC contact configuration

---

## 🔐 SSH Compliance Validation
### Cisco
- Ensures SSH version 2 is configured

### MikroTik
- Ensures SSH service is enabled
```bash
ansible-playbook -i inventory/hosts.yml playbooks/compliance.yml --ask-vault-pass
```

---

## 🔄 OSPF Neighbor Health Check (Cisco)
Validates that all Cisco devices participating in OSPF have neighbors in **FULL** state.
```bash
ansible-playbook -i inventory/hosts.yml playbooks/ospf_check.yml --ask-vault-pass
```
Used for:
- WAN stability verification
- ISP connectivity validation
- Pre-change health checks

---

## 📊 Compliance Reporting
Generates a centralized CSV compliance report covering:
- SSH compliance
- OSPF neighbor status
```bash
ansible-playbook -i inventory/hosts.yml playbooks/compliance_report.yml --ask-vault-pass
```
Output will store here:
```bash
reports/compliance_report.csv
```

---

## 🚨 Disclaimer
This project is intended for lab and controlled enterprise environments.
Always validate automation in a staging environment before deploying to production.

---
