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
| Alex Admin | alex.admin | Domain Admins | ajfieH7943f |
| Helpdesk Bob | bob.hd | Helpdesk_Team | groi984Hf9 |
| Standard Sue | sue.user | Domain Users | jurnbf(&98e |

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
- Add: `Alex Admin` and built-in Administrators group  

📸 *Screenshot Example:* ADUC showing LAB Users and Groups  

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

