# Active Directory PowerShell Setup & Configuration Guide

This guide contains all the PowerShell commands used to deploy, configure, and manage Active Directory in an Azure-hosted Windows Server VM.  
It includes installation, domain creation, OU structure, users, groups, and common help desk tasks.

---

## 🚀 Step 1 — Create the Azure VM (Free Tier)

Using Azure allows you to run the lab without local hardware. The VM runs in Microsoft’s datacenter and is accessible via RDP.

### VM Configuration

| Setting | Value | Why |
|--------|--------|------|
| **Region** | East US | Cheapest region, most VM sizes available |
| **Image** | Windows Server 2025 Datacenter — Gen2 | Latest OS, includes 180‑day evaluation |
| **Size** | Standard_B2s | Smallest size that runs AD smoothly |
| **Authentication** | Password | Needed for RDP access |
| **Public inbound ports** | Allow RDP (3389) | Required to connect |
| **OS disk** | Standard SSD | Good performance, free tier eligible |

---

## 🛠️ Step 2 — Install Active Directory Domain Services (AD DS)

After connecting via RDP, open **PowerShell as Administrator** and run:

Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
