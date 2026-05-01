# active-directory-az-vm-lab

# Active Directory Lab on Azure  
Deploying AD DS on a Windows Server 2025 VM

## 📌 Overview
This lab walks through deploying a Windows Server virtual machine in Microsoft Azure and configuring it as a Domain Controller using Active Directory Domain Services (AD DS).  
It also includes creating Organizational Units, users, groups, and applying Group Policy settings.

---

## 🧰 Prerequisites
- Basic understanding of Azure
- An active Azure account (Free Tier works)


---

## 🚀 Step 1 — Create the Azure Virtual Machine
1. Go to https://azure.microsoft.com/free and create an account.
2. Sign in at https://portal.azure.com.
3. Search for **Virtual machines** → **Create**.
4. Use the following configuration:

| Setting | Value |
|--------|--------|
| Region | East US |
| Image | Windows Server 2025 Datacenter — Gen2 |
| Size | Standard_B2s (2 vCPU, 4GB RAM) |
| Authentication | Password |
| Public inbound ports | Allow RDP (3389) |
| OS Disk | Standard SSD |

5. Click **Review + Create**, then **Create**.

---

## 🛠️ Step 2 — Install Active Directory Domain Services
1. RDP into the VM.
2. Open **Server Manager**.
3. Go to **Manage → Add Roles and Features**.
4. Select **Active Directory Domain Services**.
5. Add required features → Install.
6. Do not restart yet.

### Install Group Policy Management Console

Open PowerShell and run:

Install-WindowsFeature -Name GPMC

---

## 🛠️ Step 3 — Promote the Server to a Domain Controller

Installing AD DS adds the role, but promotion creates the actual domain.
1.	In **Server Manager**, click the yellow warning flag.
2.	Select **Promote this server to a domain controller** .
3.	Choose **Add a new forest**.
4.	Set the **domain name** to: lab1.local.
5.	Click **Next** and set a **DSRM password** (save it).
6.	Accept the default DNS and NetBIOS settings.
7.	Click **Install** — the server will restart automatically.

---

## 🛠️ Step 4 — Build the Organizational Structure

Open **Active Directory Users and Computers (ADUC)** from **Tools** in Server Manager.

## Create Organizational Units (OUs)
Right click **lab1.local**
Select → **New → Organizational Unit**. 
Create OUs for each department and one for Computers.

## Create Security Groups
Right click an OU 
Select **New → Group**. 
Set:
•	**Group scope**: Global
•	**Group type**: Security

## Create User Accounts
Use a consistent naming format, such as: **firstname.lastname**

---

## 🛠️ Step 5 — Configure Group Policy

Open **Group Policy Management** from the **Tools** menu.
1.	Expand **Forest: lab.local** → Domains → lab.local.
2.	Right click the **IT OU → Create a GPO in this domain and link it here**.
3.	Name it: **IT Security Policy**.
4.	Right click the GPO → **Edit**.
5.	Apply the settings below.

Policy Path	Setting	Value
Computer Config → Windows Settings → Security → Account Policies → Password Policy	Minimum password length	12
Computer Config → Windows Settings → Security → Account Policies → Password Policy	Password must meet complexity requirements	Enabled
Computer Config → Windows Settings → Security → Local Policies → Security Options	Interactive logon: Machine inactivity limit	900 seconds
Computer Config → Administrative Templates → System → Removable Storage Access	All removable storage classes: Deny all access	Enabled

