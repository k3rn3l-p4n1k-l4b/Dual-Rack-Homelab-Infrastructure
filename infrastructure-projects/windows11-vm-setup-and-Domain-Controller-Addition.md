Windows 11 Guest Deployment and Domain Join

## 📌 Project Overview
This project documents the architectural design, deployment, and configuration of an isolated enterprise testing environment within a Proxmox VE hypervisor topology. The project focuses on deploying a Windows 11 client workstation template inside a dedicated network zone (**VLAN 40**) and successfully binding it to a pre-configured Windows Server Active Directory Domain Controller (DC). 

Additionally, this document contains an infrastructure engineering case study detailing the isolation and remediation of host-level WAN routing and upstream DNS resolution blocks encountered while staging deployment media.

---

## 🛠️ Section 1: Proxmox Host Troubleshooting Case Study
*(Resolving GUI Media Acquisition Failures)*

### 1. Symptom Identification
During the initial staging phase of the guest operating system media, a critical infrastructure blocker occurred when utilizing the Proxmox Web GUI **Download from URL** feature to fetch the upstream installation assets. 

The hypervisor framework aborted the handshake, throwing an `invalid server response: '500 Can't connect to ://microsoft.com (Temporary failure in name resolution)' (500)` error, despite the asset target being fully reachable via an external workspace browser session.

![Proxmox 500 DNS Error](images/Windows11DNSissueError.png)


### 2. Isolation & Root Cause Analysis
To isolate whether the error originated from an upstream ISP DNS drop or an internal host routing table misconfiguration, an SSH session was established to the Proxmox node shell to map standard ICMP and Anycast paths.

Running a basic ICMP echo request against an external domain confirmed local resolution failure:
```bash
ping -c 3 google.com
# Output: ping: google.com: Temporary failure in name resolution
```

Testing direct network layer exit paths to a public Anycast IP endpoint (`1.1.1.1`) yielded an unreachable network route. This validated that the issue was two-fold:
1. **Null or Invalid Nameserver Mappings:** Misconfigured local resolver targets within the host's `/etc/resolv.conf`.
2. **Missing Gateway Traversal:** An unmapped default gateway routing entry preventing the host bridge interface from egressing to the local WAN border edge router.

### 3. Remediation & Implementation Steps

#### Step 1: Repairing Node DNS Configurations
System nameserver records were manually shifted to high-availability Anycast public DNS resolvers (Cloudflare and Google).

1. Navigated to **[Your_Node_Name]** -> **System** -> **DNS** within the Proxmox Web GUI.
2. Modified the resolver configuration table to the following target values:
   * **First DNS Server:** `1.1.1.1` (Cloudflare Primary)
   * **Second DNS Server:** `8.8.8.8` (Google Primary)
   * **Search Domain:** `local` (or corresponding lab internal search path)

#### Step 2: Validating Linux Network Interfaces
To ensure the primary software bridge interface (`vmbr0`) maintained physical interface abstraction and possessed a designated local gateway route:
1. Navigated to **System** -> **Network**.
2. Evaluated the default gateway property for `vmbr0` to verify it correctly pointed to the core router (`e.g., 192.168.1.1`).

#### Step 3: Network Verification via CLI
Following network interface configuration reloads, diagnostics were re-run inside the node shell to confirm complete path resolution loop closure:

```bash
# Verifying external name resolution and transit metrics
ping -c 3 google.com
```

**Expected Successful Output:**
```text
PING google.com (142.251.40.174) 56(84) bytes of data.
64 bytes from lga25s79-in-f14.1e100.net (142.251.40.174): icmp_seq=1 ttl=116 time=11.2 ms
64 bytes from lga25s79-in-f14.1e100.net (142.251.40.174): icmp_seq=2 ttl=116 time=10.9 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 10.911/11.082/11.234/0.134 ms
```

With host routing operational, the Proxmox hypervisor successfully pulled down the Microsoft source images directly into the local storage pools.

---

## 🖥️ Section 2: Provisioning the Windows 11 VM in Proxmox

With hypervisor WAN reachability restored, virtualization parameters were configured to meet strict Windows 11 hardware abstraction requirements.

### Step 1: VM Provisioning Configuration
1. Click **Create VM** in the Proxmox Web GUI.
2. **General Tab:** Assign a **VM ID** (e.g., `201`) and a descriptive **Name** (e.g., `LAB-WIN11-01`).
3. **OS Tab:** 
   * Select your target **Windows 11 ISO** image.
   * Set Type to **Microsoft Windows** and Version to **11/2022**.
   * Check **Add additional drive for VirtIO drivers** and map the `virtio-win.iso` image.
4. **System Tab:**
   * Machine: **Q35**
   * BIOS: **OVMF (UEFI)** -> Check *Add EFI Disk* and assign it to local storage.
   * TPM Storage: Check **Add TPM** -> Select **v2.0** and map its state storage.

![Proxmox 'System' configuration](images/Windows11SystemTab.png)

5. **Disks Tab:**
   * Bus/Device: **SCSI** (Device `0`).
   * SCSI Controller: **VirtIO SCSI single**.
   * Disk size: Allocated **64 GB+** with **Discard** enabled for system SSD arrays.
6. **CPU & Memory Tabs:**
   * Cores: Allocated **2 to 4 cores**. Set Type to **Host** for direct instruction-set execution passthrough.
   * Memory: Allocated **4096 MB** to **8192 MB** capacity.
7. **Network Tab:**
   * Model: **VirtIO (paravirtualized)**.
   * VLAN Tag: Set to **`40`** to enforce logical isolation into the Active Directory environment network zone.

![Network tab configuration](images/Windows11NetConfig.png)

---

## 💿 Section 3: Guest OS Installation & Driver Injection

1. **Boot** the VM machine profile and interface with the **NoVNC Console**. Initialize setup when prompted.
2. Advance to the system disk partitioning layout utility. 
3. **Injecting the Storage Driver:** To map the virtual disk array, select **Load driver** -> **Browse**.
4. Navigate to the mounted VirtIO directory path: `vioscsi\w11\amd64`.
5. Select the storage driver signature. The unallocated space will dynamically map to the provisioning window. Complete setup.

![VirtIO SCSI driver injected](images/Windows11virtioInjected.png)

---

## 🔓 Section 4: Bypassing OOBE Network Requirements

Because the guest is isolated within **VLAN 40** prior to local interface initialization, the Windows 11 *Out-of-Box Experience (OOBE)* network restriction must be bypassed.

1. At the network configuration terminal point, execute **`Shift + F10`** to bring up an elevated command processor.
2. Pass the following system command override:
   ```cmd
   oobe\bypassnro
   ```
3. The guest environment will instantly soft-reboot. 
4. Upon return to the setup window, select **"I don't have internet"** followed by **"Continue with limited setup"**.
5. Establish a local fallback administrative account (e.g., `LocalAdmin`) to reach the main desktop shell.

![Windows 11 OOBE Prompt](images/Windows11ByPassNRO.png)

*Note: In modern deployments, Red Hat VirtIO peripheral and network card driver integration occurs cleanly alongside storage mapping phases, resulting in automated device deployment upon landing on the system desktop.*

---

## 🔗 Section 5: Static Networking & Domain Controller Join

To securely link the client node with the enterprise directory infrastructure, its network configuration must explicitly target the Windows Server Domain Controller for DNS resolution loops.

### 1. Configure Static IPv4 Parameters
1. Invoke the run window (`Win + R`), execute `ncpa.cpl`, and access the properties framework of the active VirtIO network interface card.
2. Double-click **Internet Protocol Version 4 (TCP/IPv4)**.
3. Establish a static logical assignment constrained inside the VLAN 40 block:
   * **IP Address:** `10.10.40.20` (or similar allocated client IP)
   * **Subnet Mask:** `255.255.255.0`
   * **Default Gateway:** `10.10.40.10` (VLAN Gateway Router)
   * **Preferred DNS Server:** `10.10.40.10` (**[CRITICAL]** The exact IP address of your Active Directory Domain Controller).

![TCP/IPv4 properties](images/Windows11IPv4Config.png)

### 2. Verify Infrastructure Interoperability
Open an administrative PowerShell window to ensure name resolution pipelines cleanly reach the directory engine before submitting changes:
```powershell
# Validate lookup behaviors against the domain target
nslookup yourdomain.local

# Validate point-to-point network layer connection
ping yourdomain.local
```

### 3. Join the Target Domain
1. Invoke the run menu (`Win + R`), execute `sysdm.cpl` to enter system properties, and click **Change**.
2. Select the **Domain** toggle box, input your fully qualified enterprise domain string (e.g., `yourdomain.local`), and click **OK**.
3. Input authorized Domain Administrator system credentials (`YOURDOMAIN\Administrator`) when prompted by the security interface.
4. Confirm the presence of the system welcome dialogue window, close open configuration windows, and perform a final node restart.

![Welcome to the... domain success popup](images/Windows11WelcomeToDomanScreen.png)
