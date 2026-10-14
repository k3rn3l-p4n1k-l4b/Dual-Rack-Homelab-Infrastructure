# Dual-Rack Homelab Infrastructure

This repository documents a multi-server home lab designed to simulate real-world IT infrastructure, lifecycle management, and operational scenarios. 

To maintain uptime for live services while allowing total freedom to experiment, the infrastructure is split across **two separate physical environments**:

*   **Production Host (19-Inch Rack):** A full-sized rack deployment running Proxmox VE. This environment hosts stable, everyday infrastructure services that require high availability, such as identity management, core networking services, central storage, and automated backup routines.
*   **Tinker & Lab Host (10-Inch Rack):** A compact, space-constrained physical 10-inch rack deployment. This serves as a dedicated hardware sandbox used to safely build, break, and test configurations—including custom security policies, intrusion prevention, and incident response simulations.

While this repository documents projects built across both form factors, **completed implementations span across both environments** depending on their current operational purpose. This dual-rack setup provides a safe, realistic simulation of staging systems across separate development and production baselines.

Links provided below to jump to specific areas of interest.


### Overall Lab Completion

Progress: **10 / 21 Projects Completed**

[───────────────▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒] 50%


## 📈 Lab Progress Tracker

This homelab is an ongoing infrastructure project designed to simulate real-world IT environments.
The tracker below highlights the current progress of implemented systems and planned expansions.

| Category               | Project                                  | Status    |
| ---------------------- | ---------------------------------------- | --------- |
| Infrastructure         | Proxmox Infrastructure Host Deployment   | ✅ Completed |
| Infrastructure         | VLAN Network Segmentation                | ✅ Completed |
| Infrastructure         | Linux Server Deployment                  | 🟨 In Progress |
| Infrastructure         | Infrastructure Documentation             | 🟨 In Progress |
| Windows Infrastructure | Active Directory Domain Lab              | ✅ Completed |
| Windows Infrastructuee | Provisioning Win11 VM and Domain Join    | ✅ Completed |
| Linux Administration   | Secure Access Management (SSH Hardening) | ✅ Completed |
| Linux Administration   | Host-Based Firewall Implementation       | ✅ Completed |
| Linux Administration   | Internal DNS & DHCP Services             | ✅ Completed |
| Linux Administration   | Centralized File Server                  | ✅ Completed |
| Security               | SSH Brute-Force Mitigation (Fail2ban)    | ✅ Completed |
| Security               | SELinux Policy Management                | ⬜ Planned |
| Security               | Network Traffic Analysis                 | ⬜ Planned |
| Monitoring             | Infrastructure Monitoring Stack          | ⬜ Planned |
| Automation             | Backup Automation Scripts                | ✅ Completed |
| Automation             | System Health Check Automation           | ⬜ Planned |
| Automation             | Configuration Management with Ansible    | ⬜ Planned |
| Automation             | Service Lifecycle Automation             | ⬜ Planned |
| Incident Response      | Disk Exhaustion Incident Simulation      | ⬜ Planned |
| Incident Response      | Critical Service Outage Recovery         | ⬜ Planned |
| Incident Response      | Security Incident Investigation          | ⬜ Planned |

### Status Legend

| Symbol         | Meaning                          |
| -------------- | -------------------------------- |
| ⬜ Planned      | Project has not started          |
| 🟨 In Progress | Currently being implemented      |
| ✅ Completed    | Fully implemented and documented |



---

# Lab Architecture

![Network](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/images/network_diagram.jpg)


![10in Rack](https://github.com/JNegron-Tech/10inch-rack-homelab-infrastructure/blob/0be826285660b6cf773dbf00b5930c92a877f707/images/10in_Rack_Actual.jpg)

---

# Infrastructure Projects

| Project | Description |
|------|------|
| [Proxmox Infrastructure Host](infrastructure-projects/ProxMox-Infrastructure-Host.md) | Deploying and configuring the virtualization host |
| [VLAN Segmented Network](infrastructure-projects/VLAN-Segmented-Network.md) | Implementing network segmentation |
| [Linux Server Deployment](infrastructure/linux-server-deployment.md) | Multi-distribution Linux infrastructure |

---

# Linux Administration

| Project | Description |
|------|------|
| [Secure Access Management](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/infrastructure-projects/SSH-Brute-Force-Mitigation-Fail2ban.md) | SSH hardening and access control |
| [Host Firewalls](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/infrastructure-projects/linux-administration-host-based-firewall.md) | Implementing UFW and Firewalld |
| [DNS & DHCP Services](infrastructure-projects/Internal-DNS-DHCP/OPNSense-VLAN10-Setup.md) | Internal network services |
| [Centralized Storage Server](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/infrastructure-projects/centralized-file-server.md) | Network Attached Storage |

---

# Security Projects

| Project | Description |
|------|------|
| [Fail2ban Intrusion Prevention](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/infrastructure-projects/SSH-Brute-Force-Mitigation-Fail2ban.md) | Protecting SSH from brute-force attacks |
| [SELinux Policy Management](security/selinux-policy-management.md) | Enforcing mandatory access controls |
| [Network Traffic Analysis](security/network-traffic-analysis.md) | Packet inspection and troubleshooting |

---

# Automation

| Project | Description |
|------|------|
| [Backup Automation](https://github.com/k3rn3l-p4n1k-l4b/10inch-rack-homelab-infrastructure/blob/main/infrastructure-projects/backup-automation.md) | Automated backup scripts |
| [System Health Checks](automation/system-health-checks.md) | Monitoring via scripts |
| [Ansible Automation](automation/ansible-automation.md) | Infrastructure configuration management |

---

# Windows Infrastructure

| Project | Description |
|------|------|
| [Active Directory Lab](infrastructure-projects/active_directory_deployment.md) | Windows domain services and group policy |
| [Windows 11 VM Provisioning & Domain Join](infrastructure-projects/windows11-vm-setup-and-Domain-Join.md) | Adding End Point To Domain Controller |

---

# Incident Response Scenarios

| Scenario | Description |
|------|------|
| [Disk Exhaustion Incident](incidents/disk-exhaustion-incident.md) | Responding to storage failures |
| [Service Outage Recovery](incidents/service-outage-recovery.md) | Recovering critical services |
| [Security Incident Simulation](incidents/security-incident-response.md) | Investigating unauthorized access |

⚠️ Note on Security (OpSec): Internal IP addresses, subnets, and hostnames have been intentionally omitted, masked, or blurred throughout this documentation to protect the integrity of the live environment
