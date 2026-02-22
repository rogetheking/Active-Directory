<p align="center">
  <a href="https://github.com/drewmarsh/on-premises-active-directory-configuration">
    <img src="/images/active-directory.png" alt="AD Logo">
  </a>

  <a href="https://github.com/drewmarsh/on-premises-active-directory-configuration">
    <img src="/images/network-diagram.png" alt="Network Diagram">
  </a>
</p>

# 🧠 Technologies Used
- Active Directory Domain Services
- Microsoft Azure (Cloud computing)
- Microsoft Remote Desktop
- PowerShell
- Windows Server 2022
- Windows 10 Pro, version 22H2

# ⚙️ Deployment & Configuration

### 🖥️ Create both Azure Virtual Machines  [(example guide)](https://github.com/drewmarsh/azure-creating-VM)
- **Client-1** running Windows 10 Pro, version 22H2 - x64 Gen2
- **DC-1** running Windows Server 2022 Datacenter Azure Edition - x64 Gen2

> [!NOTE]
> Put both of these Azure Virtual Machines into a Resource Group called `DC-1_group`

### 🛜 Configure static IP address on Domain Controller 1 (DC-1)
1. Open the DC-1 virtual machine
2. Navigate to `Networking` > `Network settings` > The name of the network interface (ex. `dc-1410`) > `IP configurations`
3. Select `ipconfig1`
4. Change `Dynamic` to `Static`
5. Click `Save`

<img src="/images/static-ip.png" alt="Static IP">

### 📋 RDP into DC-1 and enable ICMP rules

After the domain controller fully initializes, minimize the Server Manager window for now.

1. Use windows search to open 'Windows Defender Firewall'
2. Select `🛡️ Advanced settings`
3. Navigate to the 'Inbound Rules' tab
4. Find the two rules labeled 'Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)'
5. Right-click and select `Enable` on both rules

<img src="/images/inbound-rules.png" alt="Inbound Rules">

### 📥 Install Active Directory on DC-1
1. Open Server Manager
2. Click `Manage` > `Add Roles and Features`
3. Click `Next` 3 times to get to the Server Roles tab
4. Enable ☑️ ```Active Directory Domain Services```
5. Click `Add Features`
6. Click `Next` 3 more times until reaching the `Install` button
7. When installation finishes, click `Close`

<img src="/images/tick-ad-domain-services.png" alt="Tick AD Domain Services">

<img src="/images/install-ad.png" alt="Install AD">

In the top-right of the Server Manager window:
1. Click the flag icon with a warning notification
2. Select 'Promote this server to a domain controller'

<img src="/images/promote-server.png" alt="Promote Server to a DC">

In the Active Directory Domain Services Configuration Wizard:
1. On 'Deployment Configuration' tab:
   - Tick ☑️ ```Add a new forest```
   - Enter "mydomain.com" in **Root domain name:**

<img src="/images/deployment-config.png" alt="Deployment Config">

2. On 'Domain Controller Options' tab:
   - Set secure password in both fields

<img src="/images/set-password.png" alt="Set Password">

3. On 'DNS Options' tab:
   - Ensure ◻️ ```Create DNS delegation``` is unchecked

4. Click `Next` until reaching 'Prerequisites Check' tab
5. Click `Install`

<img src="/images/click-install.png" alt="Click Install">

<img src="/images/installing.png" alt="Installing">

> [!NOTE]
> Your connection to DC-1 will be lost and you'll need to RDP back using "mydomain\" before your username

<img src="/images/computer-is-restarting.png" alt="Computer Is Restarting">

<img src="/images/new-credentials.png" alt="New Credentials">

When the credentials are accepted, it will take a moment to load back to the desktop while the system waits for the Group Policy Client

<img src="/images/wait-for-group-policy.png" alt="Wait for Group Policy">

### 📝 Add Organizational Units for Employees & Admins

1. In Server Manager, navigate to `Tools` > `Active Directory Users and Computers`

<img src="/images/users-and-computers.png" alt="Users and Computers">

2. Create Organizational Units:
   - Right-click `mydomain.com` > `New` > `Organizational Unit`
   - Create "_EMPLOYEES"
   - Repeat process to create "_ADMINS"

<img src="/images/employees.png" alt="_EMPLOYEES">

<img src="/images/admins.png" alt="_ADMINS">

### 👩‍💻 Add a new admin named Jane Doe

1. Right-click `_ADMINS` > `New` > `User`
2. Enter information:
   - **First name:** `Jane`
   - **Last name:** `Doe`
   - **User logon name:** `Jane_admin`

<img src="/images/new-user.png" alt="New User">

3. Set password:
   - Enter secure password
   - Uncheck ◻️ ```User must change password at next logon```
   - Check ☑️ ```Password never expires```

<img src="/images/jane-doe-password.png" alt="Jane Doe Password">

4. Add to Domain Admins:
   - Right-click 👤 Jane Doe > `Properties`
   - Navigate to 'Member of' tab
   - Click `Add...`
   - Enter "domain admins"
   - Click `Check Names` > `OK`
   - Click `Apply` > `OK`

<img src="/images/domain-admins.png" alt="Domain Admins">

5. Log out and log back in as Jane_admin

<img src="/images/jane-logon.png" alt="Jane Logon">

<img src="/images/jane-welcome.png" alt="Jane Welcome">

### 🌐 Add DC-1's private IP as Client-1's DNS server
1. On Client-1, navigate to:
   - `Networking` > `Network settings` > Network interface name > `DNS servers`
2. Change from `Inherit from virtual network` to `Custom`
3. Enter DC-1's private IP address in the **Add DNS server** text field
4. Click `Save`
5. Restart Client-1

<img src="/images/add-dns.png" alt="Add DNS">

### 🤝 Join Client-1 to the domain

1. RDP into Client-1
2. Test connection:
   - Open Command Prompt
   - Ping DC-1's IP (e.g., `ping 10.0.0.4`)
   - Verify response

<img src="/images/ping-DC.png" alt="Ping DC">

3. Join domain:
   - Navigate to `System` > `About` > `Rename this PC (advanced)` > `Change...`
   - Enter "mydomain.com" in **Domain:** field
   - Enter credentials: "mydomain.com\Jane_admin"
   - Restart Client-1

<img src="/images/change-pc-name.png" alt="Change PC Name">

<img src="/images/join-domain-with-admin-credentials.png" alt="Join Domain with Admin Credentials">

4. Verify on DC-1:
   - Open `Active Directory Users and Computers`
   - Check `mydomain.com` > `Computers`
   - Verify Client-1 is listed

<img src="/images/client1-computer.png" alt="Client-1 Computer">

### 🌐 Set-up Remote Desktop for Non-administrative Users on Client-1

1. On Client-1 (as Jane_admin):
   - Open `Settings`
   - Navigate to `System` > `Remote Desktop`
   - Click `Select users that can remotely access this PC`
   - Click `Add...`
   - Enter "domain users"
   - Click `Check Names` > `OK`

<img src="/images/domain-users.png" alt="Domain Users">

### 👥 Bulk create Active Directory with script and test

1. On DC-1 (as Jane_admin):
   - Open PowerShell ISE
   - Click `File` > `New`
   - In the **Untitled1.ps1** text box, write your own or paste [this bulk user creation script](https://github.com/drewmarsh/active-directory-bulk-user-creation)
   - Click green Run Script button
   - Click red Stop Operation when desired users created

<img src="/images/bulk-create-ad-users.png" alt="Bulk Create AD Users">

2. Verify users:
   - Check `Active Directory Users and Computers`
   - Look in `_EMPLOYEES` folder
