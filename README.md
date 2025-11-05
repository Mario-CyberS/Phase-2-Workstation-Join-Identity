# Phase 2 — Workstation Join + Identity  
This phase covers adding user accounts, creating organizational security groups, provisioning a Windows 11 client VM, joining it to the Active Directory domain, and applying a baseline Group Policy Object (GPO) to enforce domain security standards.  

---

## 🎯 Objective  
To simulate real-world IAM operations by creating domain user identities, joining a workstation to the `lab.local` domain, and applying centralized password and lockout policies through Group Policy.

---

## 🔍 Why Implement Workstation Join + Identity?  
This phase connects identity management to actual endpoint devices, enabling:  
- Centralized authentication for all users  
- Group-based access control  
- Domain security enforcement via Group Policy  
- Hands-on practice with provisioning and lifecycle management  

---

## 📚 Skills Learned  
- Creating domain user and security group objects  
- Joining a Windows 11 client to an AD domain  
- Verifying DNS and network trust relationships  
- Applying baseline domain-wide security policies  

---

## 🛠️ Tools Used  
<div>
  <a href="https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025" target="_blank">
    <img src="https://img.shields.io/badge/-Windows_Server_2025-0078D4?style=for-the-badge&logo=windows&logoColor=white"/>
  </a>
  <a href="https://www.microsoft.com/en-us/software-download/windows11" target="_blank">
    <img src="https://img.shields.io/badge/-Windows_11_Pro-00adef?style=for-the-badge&logo=windows&logoColor=white"/>
  </a>
  <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc" target="_blank">
    <img src="https://img.shields.io/badge/-Active_Directory_Users_&_Computers-3333CC?style=for-the-badge"/>
  </a>
  <a href="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy" target="_blank">
    <img src="https://img.shields.io/badge/-Group_Policy_Management-008272?style=for-the-badge"/>
  </a>
</div>

---

## 📝 Deployment Steps  

### 6️⃣ Create Test User Accounts  
In **Active Directory Users & Computers (ADUC)** → `LAB Users` → Right-click → **New → User**  

Example users:  

| Name | Username | Member Of | Password |
|------|-----------|-----------|-----------|
| Alex Admin | alex.admin | Domain Admins | pass |
| Helpdesk Bob | bob.hd | Helpdesk_Team | pass |
| Standard Sue | sue.user | Domain Users | pass |

To add users to groups:  
Right-click the user → **Add to a group** → Type group name (e.g., *Administrators*, *Helpdesk_Team*) → OK  

Move `alex.admin` to the **LAB Admins** OU.  

---

#### 🧩 Create Security Groups  
**Helpdesk Group**  
- Location: LAB Groups  
- Action → New → Group  
- Name: `Helpdesk_Team`  
- Scope: Global  
- Type: Security  
- Add: `Helpdesk Bob`  

**Admin Group**  
- Name: `LAB Admins`  
- Scope: Global  
- Type: Security  
- Add: `Alex Admin` and add him to built-in Administrators group  

---

### 7️⃣ Create Windows 11 Client and Join Domain  

#### VM Setup  
- Create a **Windows 11 VM** in VMware Workstation  
- Network: **Same NAT or Internal Network** as DC (`lab.local`)  
- Install OS and complete Windows setup  

---

#### ✅ Step 1 — Network Configuration  
1. Power off Windows 11 VM  
2. Go to **VM > Settings → Network Adapter**  
3. Select **NAT**  
4. Enable **Connect at power on**  
5. Power on VM  

---

#### ✅ Step 2 — Verify Network Connectivity  
In **CMD**:  
```powershell
ipconfig
```
You should see an IP in the 192.168.163.xxx range (from your DHCP scope).
If yes, awesome — the DC issued the address.


Then test ping to DC:
```powershell
ping 192.168.163.10
```
Then test DNS:
```powershell
ping lab.local
```
And:
```powershell
nslookup lab.local
```
You should see your Domain Controller return.
If these work → your VM is talking to your AD network ✅

---

#### ✅ Step 3 — Rename the PC
Settings → Rename (At Top)

When it asks for PC name, set:
```powershell
WIN11-LAB01
```
(This is clean, enterprise-style naming.)

Click Next → Restart later (we'll join domain next).

---

#### ✅ Step 4 — Join Domain
After rename reboot:

Follow these steps inside your Win11 VM:

### 1️⃣ Open Network Settings
Settings → Network & Internet → Ethernet (or Wi-Fi if using)
Click Edit DNS assignment.

### 2️⃣ Set DNS Manually
Set:
```powershell
Preferred DNS: 192.168.163.10
Alternate DNS: 8.8.8.8   (optional)
```
Click Save

### 3️⃣ Flush DNS & verify
Open PowerShell and run:
```powershell
ipconfig /flushdns
ipconfig /release
ipconfig /renew
```
Then retry:
```powershell
nslookup lab.local
```
Expected result:
```powershell
Server:  WIN-96HUEHD76S4.lab.local
Address: 192.168.163.10
Name:    lab.local
```
Next perform these steps:

- Control Panel -> Search domain -> System (Join a Domain)
- Select Change
- Select Member of Domain and enter AD domain name
- Enter:
```powershel
lab.local
```
- When prompted for creds:
```powershell
Username: LAB\Administrator
Password: <your DC admin password>
```
If successful — you will see:
```powershell
Welcome to the LAB domain
```
Then Reboot.

---

#### ✅ Step 5 — Move Computer to OU
Back on your Domain Controller:
- Open Active Directory Users and Computers
- Go to LAB Computers OU
- If your WIN11-LAB01 machine is in Computers container,
right-click it → Move → send it to LAB Computers

#### 🎯 Success Check
Log into Win11 with a domain account:
On login screen:
```powershell
Other User →
LAB\standard.user
Password: *****
```
If it logs in → 🎉 domain join complete

---

#### 8️⃣ GPO — Baseline Domain Policy
Create New GPO
Tools → Open Group Policy Management
Open Domains → drop down lab.local → Right click Group Policy Objects → New → Create GPO: LAB-Security-Baseline
Right-click LAB-Security-Baseline → Edit and set:


1. Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy
 - Enforce history: 24
 - Max age: 90 days
 - Min age: 1 day
 - Min length: 12
 - Complexity: Enabled
 - Reversible encryption: Disabled

2. Account Lockout Policy
 - Threshold: 5
 - Duration: 15 minutes
 - Reset counter after: 15 minutes

3. Close editor.
4. Now link it to the domain:
 - In the left tree, right-click lab.local (the domain, not “Domains”) → Link an Existing GPO…
 - Pick LAB-Security-Baseline → OK.


Force apply the GP on both your DC VM terminal and Windows 11 VM as alex.admin:
```powershell
gpupdate /force
gpresult /r
```
You should see the Group policy was applied.

---

### 👨‍💻 Author
Mario Tagaras | Cyber Security Specialist

