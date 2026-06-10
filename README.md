# Active-Directory-GPO-lab
Implementing Group Policy Object to Users and PC under OU
# Active Directory GPO Implementation Lab (Windows Server 2025)

## Project Overview
This lab project demonstrates how to centrally manage, restrict, and configure client workstations in an enterprise network using Group Policy Objects (GPOs). Operating in a Windows Server 2025 environment, I built a domain infrastructure, created targeted Organizational Units (OUs), and deployed multiple GPOs to enforce corporate wallpaper standards, restrict access to sensitive system settings, configure logon banners, and automatically map network drives.

## Topology & Lab Environment
* **Domain Controller VM:** Windows Server 2025 (Hostname: `SERVER0` | Domain: `college.com`)
* **Client Workstation VM:** Windows 10 Enterprise (Testing with domain user account: `jeran rai`)
* **Network Setup:** Dedicated Layer 2 host-only network inside VMware Workstation Pro ensuring isolated, secure communication.

---

## Step-by-Step Implementation

### Step 1: Active Directory OU Setup
* Opened Active Directory Users and Computers (ADUC) on SERVER0.
* Created a new Organizational Unit (OU) named `Computer Science`.
* Provisioned the test user account `jeran rai` and added the Windows 10 computer object directly into the `Computer Science` OU to ensure targeted policy application.

![Users in OU Computer Science](./images/w1.png)
*Fig.1: Users in OU Computer Science*

---

### Step 2: Preparing the Centralized Storage (UNC Share)
To deploy the corporate wallpaper via GPO, the file had to be accessible over the network:
* Created a local folder on the drive `C:\wallpaper` and added `image2.jpg` inside it.

![Sharing Folder Asset](./images/w2.png)
*Fig.2: Wallpaper Image Source*

* Configured **Advanced Sharing** to share the folder as `wallpaper`, making the network path `\\SERVER0\wallpaper\image2.jpg`.

![Advanced Sharing Configuration](./images/w3.png)
*Fig.3: Advanced Sharing Properties*

* Adjusted **Share Permissions** to grant *Everyone* Read access. This prevents GPO deployment failures, as restrictive permissions block client machine processing.
* Adjusted **NTFS Security Permissions** to ensure Domain Users/Computers could explicitly read and execute the file.

![Granting Network Access](./images/w4.png)
*Fig.4: Granting Permissions to Everyone*

---

### Step 3: Configuring Group Policies
Opened the **Group Policy Management Console (GPMC)** and created/linked a series of policies on the `Computer Science` OU:

#### 3.1 Wallpaper Deployment (`wallpaper policy`)
* Navigated to: `User Configuration -> Administrative Templates -> Desktop -> Desktop -> Desktop Wallpaper`.
* Enabled the policy and set the wallpaper path to the UNC format: `\\SERVER0\wallpaper\image2.jpg`.
* Set the wallpaper style to **Fill**.

![Wallpaper Deployment Configuration](./images/w5.png)
*Fig.5: Wallpaper Policy Layout*

#### 3.2 System Hardening (Control Panel Restriction)
* Configured user restrictions to prevent standard users from tampering with local network adapters or system preferences.
* **Path 1 (Control Panel):** `User Configuration -> Policies -> Administrative Templates -> Control Panel -> Prohibit access to Control Panel and PC settings`
* **Path 2 (Network Connections):** `User Configuration -> Policies -> Administrative Templates -> Network -> Network Connections -> Prohibit access to properties of a LAN connection`

#### 3.3 Network Drive Mapping (`map network drive`)
* Utilized Group Policy Preferences (GPP) to map corporate file shares to client machines automatically upon login, removing the need for legacy batch login scripts.
* **Navigation Path:** `User Configuration -> Preferences -> Windows Settings -> Drive Maps`

![Drive Maps Console](./images/w6.png)
*Fig.6: Drive Maps Extension Console*

#### 3.4 Corporate Compliance (`banner logon`)
* Enabled interactive logon messages forcing an organization warning to display before a user can sign in.
* **Path 1 (Title):** `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> Security Options -> Interactive logon: Message title for users attempting to log on.`
* **Path 2 (Text):** `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> Security Options -> Interactive logon: Message text for users attempting to log on.`

---

## Step 4: Verification & Troubleshooting
* Logged into the Windows 10 client machine as `college\jeran rai`.
* Opened the command prompt and ran a forced policy update to pull the new changes from the Server 2025 Domain Controller immediately:
  ```cmd
  gpupdate /force
