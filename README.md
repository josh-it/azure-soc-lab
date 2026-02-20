# 🛡️ Azure Home SOC Lab — Honeypot + Microsoft Sentinel

A hands-on cloud Security Operations Center (SOC) built in Microsoft Azure. This project deploys a Windows honeypot VM, intentionally exposes it to the internet, and uses Microsoft Sentinel (SIEM) to capture, forward, and analyze real-world attacker login attempts.

---

## 📌 Skills Demonstrated

- Cloud infrastructure deployment (Azure)
- SIEM integration (Microsoft Sentinel)
- Threat monitoring & log analysis
- Network security configuration
- Attack visualization

---

## 🏗️ Architecture Overview

```
Internet (Attackers)
        |
        ▼
  Windows 10 VM (Honeypot)
  Public IP | NSG: All Inbound Allowed
  Windows Firewall: Disabled
        |
        ▼
  Azure Monitoring Agent
        |
        ▼
  Log Analytics Workspace (law-soc-lab)
        |
        ▼
  Microsoft Sentinel (SIEM)
  → Log Queries (KQL)
  → Attack Visualization
```

![Soc Image](/images/SOC.png) 

---

## 🧰 Prerequisites

- Personal Microsoft account (not work/school)
- Azure free trial account with credit card
- Windows PC or Mac with Remote Desktop client

---

## 📋 Step-by-Step Walkthrough

### Part 1 — Cloud Infrastructure

#### Step 1 – Create Azure Account
Sign up at [portal.azure.com](https://portal.azure.com) using a personal email and complete identity + billing verification.

---

#### Step 2 – Create Resource Group

| Setting | Value |
|---|---|
| Subscription | Default |
| Resource Group | `rg-soc-lab` |
| Region | East US 2 |

![RG created successfully](/images/RG_Create.png)
Azure portal showing the `rg-soc-lab` resource group successfully created.

---

#### Step 3 – Create Virtual Network

| Setting | Value |
|---|---|
| Resource Group | `rg-soc-lab` |
| Name | `vnet-soc-lab` |
| Region | East US 2 |

---

#### Step 4 – Create Windows Virtual Machine (Honeypot)

| Setting | Value |
|---|---|
| Resource Group | `rg-soc-lab` |
| VM Name | `corpnet-east-1` |
| Region | East US 2 |
| Image | Windows 10 |
| Size | Standard B1s |
| Username | `labuser` |
| Virtual Network | `vnet-soc-lab` |
| Public IP | Enabled |

> ⚠️ Save your password before deploying!

![VM Overview](/images/vm-overview.png) Azure VM overview page showing `corpnet-east-1` running with its public IP address visible.

---

### Part 2 — Expose the Honeypot

#### Step 5 – Open Network Security Group (Firewall)

Navigate to the NSG attached to the VM. Delete the default RDP rule, then add a new inbound rule:

| Setting | Value |
|---|---|
| Source | Any |
| Destination | Any |
| Protocol | Any |
| Port | `*` |
| Action | Allow |
| Name | `DANGER-AllowAll` |

> ⚠️ This is intentional — it makes the honeypot visible and reachable by attackers.

![nsg-rules](/images/nsg-rules.png)NSG inbound security rules showing the `DANGER-AllowAll` rule active.

---

### Part 3 — Connect & Weaken the VM

#### Step 6 – RDP Into the VM

1. Copy the public IP from the VM overview page
2. Open Remote Desktop (`mstsc` on Windows)
3. Paste the IP and log in as `labuser`

---

#### Step 7 – Disable Windows Firewall (Inside the VM)

1. Press **Start** → type `wf.msc`
2. Open **Windows Defender Firewall Properties**
3. Set **Domain**, **Private**, and **Public** profiles to **Off**
4. Click **Apply → OK**

![Firewall disabled](/images/firewall-disabled.png) window showing all three firewall profiles set to "Off".

---

#### Step 8 – Verify External Connectivity

From your local machine, ping the VM's public IP:

```bash
ping <VM Public IP>
```

Successful replies confirm the VM is reachable from the internet.

![ping-test](/images/ping-test.png) Terminal/CMD showing successful ping replies to the VM's public IP.

---

### Part 4 — Generate Security Logs

#### Step 9 – Simulate Failed Login Attempts

Disconnect from the VM, then deliberately attempt to RDP with wrong credentials (e.g., username: `employee`, wrong password) 3–4 times before logging in correctly. This generates **Event ID 4625** (Failed Logon) entries in the Windows Security log.

---

#### Step 10 – View Logs Locally in Event Viewer

Inside the VM:

1. Open **Event Viewer**
2. Navigate to **Windows Logs → Security**
3. Click **Filter Current Log** → Event ID: `4625`

![event-viewer](/images/event-viewer.png) Event Viewer showing Event ID 4625 entries (failed login attempts), ideally with one expanded to show attacker IP details.

---

### Part 5 — Log Repository

#### Step 11 – Create Log Analytics Workspace

| Setting | Value |
|---|---|
| Name | `law-soc-lab` |
| Resource Group | `rg-soc-lab` |
| Region | East US 2 |

![law-workspace](/images/law-workspace.png) Log Analytics Workspace overview page for `law-soc-lab`.

---

### Part 6 — Deploy Microsoft Sentinel

#### Step 12 – Add Sentinel to the Workspace

1. Search **Microsoft Sentinel** in Azure
2. Click **Create**
3. Select workspace: `law-soc-lab`
4. Click **Add**

![sentinal-dashboard](/images/sentinel-dashboard.png) Microsoft Sentinel dashboard successfully connected to `law-soc-lab`.

---

### Part 7 — Connect VM Logs to Sentinel

#### Step 13 – Install the Windows Security Events Connector

1. Open Sentinel → **Content Hub**
2. Search: `Windows Security Events`
3. Click **Install → Manage → Open Connector Page**

---

#### Step 14 – Create Data Collection Rule (DCR)

| Setting | Value |
|---|---|
| Name | `DCR-Windows-Security` |
| Resource | `corpnet-east-1` (your VM) |
| Collect | All Security Events |

This installs the **Azure Monitoring Agent** on the VM and begins streaming logs to Sentinel.

![data-collection-rule](/images/dcr-config.png) Data Collection Rule configuration page showing the VM selected and "All Security Events" chosen.

---

### Part 8 — Monitor Live Attacks

Wait **30–60 minutes** after setup. Real attackers will begin probing your honeypot automatically.

Inside Sentinel, go to **Logs** and run a KQL query to see incoming attacks:

```kql
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(1hr)
| project TimeGenerated, Account, Computer, EventID, Activity, IpAddress
```

![sentinel-kql-results](/images/sentinel-kql-results.png) Sentinel Logs window showing the KQL query results with real attacker IPs and failed login counts.

![attack map results](/images/attack-map.png)Sentinel Workbook or map visualization showing geographic origin of attacks.

---

## 🔒 Cleanup (Important!)

To avoid unexpected charges, delete all resources after the lab:

1. Go to **Resource Groups**
2. Select `rg-soc-lab`
3. Click **Delete Resource Group**
4. Confirm by typing the name

---

## 📁 Project Structure

```
CYBER HOME LAB/
├── azure.soc-lab-README.md          ← This file
└── images/
    ├── 01-RG_Create.png
    ├── 02-vm-overview.png
    ├── 03-nsg-rules.png
    ├── 04-firewall-disabled.png
    ├── 05-ping-test.png
    ├── 06-event-viewer-4625.png
    ├── 07-law-workspace.png
    ├── 08-sentinel-dashboard.png
    ├── 09-dcr-config.png
    ├── 10-sentinel-kql-results.png
    └── 11-attack-map.png  (bonus)
```

---

## 📚 References

- [Microsoft Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/)
- [Azure Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [Windows Security Event IDs](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)

---

*Built as part of a hands-on cloud security portfolio project.*
