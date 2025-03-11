<h1>Home Lab</h1>

<h2>Description</h2>

Project stimmed from wanting to test out Active Directory and how to create/manage accounts. This will include Oracle VirtualBox with Windows Server 2019 and some Windows Vms.

<h2>Prerequisites</h2>

  - Oracle VirtualBox
  - Windows 10 ISO
  - Windows Server 2019 ISO
    
Set Up the Domain Controller (Windows Server 2019)

  - Create a New VM
    - **Name** the VM (Server2019-DC) and select “Windows 2019 (64-bit)”
  - Install Windows Server 2019
    -  Boot the VM and follow the Setup wizard.
      - Complete the Installation and set an Admin password.
  - Configure Networking
    - One adapter (Internet-facing) gets an IP from your home router
    - The second adapter (Internal-only) for your VBox
      1. **Rename Adapters**
          - _Ethernet 1 → “Internet”_
          - _Ethernet 2 → “VBox_internet"_
      2. **Assign Static IP to Internal Network**
        - Example IP: 172.20.0.1
        - Subnet Mask: 255.255.255.0
        - Leave **Default Gateway** blank (none)
        - **Preferred DNS**: 172.20.0.1 (the DC’s own IP)
      
<h2>Install and Configure Active Directory</h2>

  - Add Roles and Features
    - **Server Manager → Dashboard → Add Roles and Features**
    - Click **Next** twice, select your server, and then **Active Directory Domain Services**.
    - Click **Next** and Finish.
  - Promote the Server to a Domain Controller
    - In **Server Manager**, a **yellow notification** (flag icon) will appear.
    - Select **Promote this server to a domain controller**.
    - **Deployment Configuration**: **Add a new forest** and set your **Root domain name** (e.g., mydomain.com).
    - **Complete the Wizard** and let reboot.

<h2>Create and Manage AD Users</h2>

- Organizational Unit (OU)
  - **Open Active Directory Users and Computers**
  - Right-click your domain (e.g., mydomain.com) → **New → Organizational Unit**.
  - Name it something like Admin or IT Users.
- New User Accounts
  - Right-click the OU → **New → User**.
  - Fill in **username, password, and other details**.
  - Adjust properties (group memberships, profiles, etc.) as needed.
  - Bulk User Creation with PowerShell
    - Create a .txt file named name_list.txt with a random amount of Fname Lname on ./Desktop for ease of access. (10 were created for this project)
   

   In PS use the following commands. ("Yes to all" this will allow scripts to be ran)
  
          Set-ExecutionPolicy Unrestricted
   (The Rest till the end at } can be copied and pasted into PS. Automating account creation tests batch user accounts. The above script will parse the user and make the username, set password and put them in correct OU.
   Remember, when running the script make sure your in the directory that has the name_list.txt ex. /user/---/Desktop/name_list.txt)
  
          $PASSWORD_FOR_USERS = "Password123
          
          $USER_FIRST_LAST_LIST = Get-Content .\name_list.txt
          
          $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
          
          New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false
          
          foreach ($n in `$USER_FIRST_LAST_LIST) {
          
          $first = $n.Split(" ")[0].ToLower()
          
          $last = $n.Split(" ")[1].ToLower()
          
          $username = "$($first.Substring(0,1)$(last)".ToLower()
          
          Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
          
          New-AdUser -AccointPassword $password
          
          Given Name $first
          
          Surname $last
          
          DisplayName $username
          
          Name $username
          
          EmployeeID $username
          
          PasswordNeverExpires $true
          
          Path "ou=_USERS,$(([ADSI]' "").distinguishedName)"
          
          Enabled $true
          
          }
      
         
<h2>Enable NAT and RAS (Routing and Remote Access)</h2>

  - Install Remote Access
    - **Server Manager → Add Roles and Features**
    - **Remote Access → Routing**
    - Complete and finish.
  - Configure NAT
    - **Server Manager → Tools → Routing and Remote Access**
    - Right-click your server → **Configure and Enable Routing and Remote Access**
    - Select **Network Address Translation (NAT)**
    - When prompted for the interface, choose the **“Internet”** adapter.
    
<h2>Setup DHCP</h2>

  - Install DHCP Server
    - **Server Manager → Add Roles and Features → DHCP Server**
  - Create Scope
    - **Server Manager → Tools → DHCP**
    - Expand your server and right-click **IPv4 → New Scope**.
    - **Define IP Range**:
        - Start IP: 172.20.0.100
        - End IP: 172.20.0.200
        - Subnet Mask: 255.255.255.0
        
<h2>Create a Windows 10 Client VM</h2>

   - New VM
     - Name it Client1, select **Windows 10 (64-bit)** as OS.
  - Configure Networking
    - **Adapter 1**: Attach to “Internal Network” (the same one as the DC).
    - Complete the setup
  - Join the Domain
    - Check **IP settings** to ensure it’s using the DHCP scope from the DC.
    - **System Properties → Change Settings → Change**
    - **Enter Computer Name** (e.g., Client1) and **Domain** (mydomain.com).
    - Provide **Domain Admin** credentials.
    - Reboot.
