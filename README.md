<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/img_activedirectory-page.png" height="65%" width="40%" />
</p>

<h1>Introduction</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />

<h2>Tutorial Guidelines</h2>

<h3>Step 1: Creating two virtuals machine in Azure</h3>

First we need two VMs: one running Windows Server 2022 (Domain Controller) and another running Windows 10 Pro (Client). I've detailed in the previous tutorials on how to create a virtual machine in Azure which you can find by clicking 
[here](https://github.com/Mwajiduddin/How-to-create-a-virtual-machine-in-Microsoft-Azure) and [here](https://github.com/Mwajiduddin/Observing-network-traffic-between-two-virtual-machines-in-Azure-using-Wireshark) but I'll to a quick rundown here too.

To start off let's first create our Windows Server VM, this is the VM that will have Active Directory installed and serve as our Domain Controller. So log into Microsoft Azure, go into "Virtual machines", name the VM, select Windows Server 2022 next to "Image" and select a region. You might be wondering why I didn't create a resource group first like I did in the previous tutorials, here you can make and name one by clicking on "Create new" next to "Resource group". What I showed in the previous tutorials was to help get a better understanding on the process of what goes into creating one. Choose a size with atleast two VCPUs otherwise it will be slow, create a username and password and keep note of it, check the Licensing box, hit "Review + create" and after validating click on "Create." 

While this VM is being creating, we can start making our second VM which will serve as our Client so same process as last time only for "Image" select Windows Pro 10, remember to select the same resource group as the first VM you made and for ease of use you can choose the same username and password as the first VM you made.

Next we need to switch the Windows Server VM private IP address to static and this can be done by selecting the Windows Server VM, clicking on "Networking" then the blue text next to Network Interface, selecting "IP configuration", clicking on the IP address underneath the "Private IP address" column, switching it to "Static" and save.

<h3>Step 2: Enabling connectivity between the Domain Controller and Client virtual machines</h3>

Log into your Client VM by using Remote Desktop Connection, once there open up command prompt and ping the Windows Server VM by its private IP address. You'll notice that it will respond with a timed out request meaning that Windows Server's firewall is blocking any ICMP inbound traffic. To allow this kind of traffic and enable connectivity, log into the Windows Server VM by using Remote Desktop Connection and search Windows Defender Firewall in the Windows search bar and select "Windows Defender Firewall with Advanced Security." Select "Inbound Rules" and sort by protocol by clicking on the protocol column. Then enable "Core Networking Diagnostic - ICMP Echo Request (ICMPv4-In)" both the private and domain by right clicking and selecting "Enable Rule". After this you will notice the ping requests will start working again in the Client VM.

<h3>Step 3: Installing Active Directory in Windows Server VM</h3>

Open Server Manager in your Windows Server VM, select "Add roles and features", click Next for each section, once you are in "Server Roles" select "Active Directory Domain Services" and "Add Features" on the pop-up window and finish installing it. Then click on the top right flag with the yellow exclamation point, click on the blue text "Promote this server to a domain controller", then select "Add a new forest" on the "Deployment Configuration" window and name your root domain. Make a password under "Domain Controller Options" (choose the same password you made for Windows Server VM for simplicity) and proceed through in installing. Once done installing our Windows Server VM is now a domain controller and you'll notice that it will automatically log you off so just log back in but this time our username will be different. Your username will be the name of your root domain + backslash + VM's username such as below.


<h3>Step 4: Creating Organizational Units (OU) and Admin accounts in Active Directory</h3>

In this step we will be creating a couple of OUs which are basically folders in Active Directory. So after logging into your Domain Controller VM, open up Server Manager, click on "Active Directory Users and Computers" then right click on your root domain and select "Organizational Unit" and name one as "EMPLOYEES" and another as "ADMINS." Now we are going to create our own Admin account so go to the ADMINS folder, right click, hover over to New, click on User then name your Admin, user logon account, your password and uncheck the box that says "User must change password at next logon" and hit Finish. But the user we just made isn't an Admin just yet so right click on the user account, select Properties, click on "Member Of" tab and click on Add. A window will appear, at the bottom where it states to enter object names type in "Domain Admin" in the box, click on "Check Names" and then press OK. Then log out of the VM and log back in this time as the Admin you created, the username will be the name of your root domain + backslash + admin username.


<h3>Step 5: Joining the Client VM to the Domain Controller</h3>

In this step we are going to use the Domain Controller as Client's DNS server instead of the virtual DNS server assigned to the Client VM and we start off in the Azure portal and copy the private IP address for Domain Controller VM, head over to Networking under Settings for Client VM, click on the blue text of the Network Interface, then go into DNS server under Settings, select Custom and type in private IP address for the Domain Controller VM and hit Save. Then go back to Virtual machines, select your Client VM and click Restart and log back in to the VM. Open command prompt and type in the command ipconfig /all and you can see that the DNS server IP address is the same as the Domain Controller VM's private IP address. To join this Client VM to the Domain Controller, right click the Windows start icon and click on System, go to "Rename this PC (advanced)" then click "Change", select Domain, type in your root domain and the a window will pop up asking for a username and password where you will enter the credentials of the Domain Admin account you created. After entering your credentials you will be prompted to restart the VM, now when you boot up Client VM it will be a member of the Domain Controller and you can log in the Client VM as the Domain Admin User you created in the previous step.


<h3>Step 6: Logging into Client VM as a non-administrator user</h3>

We've showcased from the previous step that we can now log into the Client VM by using an administrator account, in this step we are going to configure the Client virtual machine to let us log in as a normal domain user. We start off by right clicking the Windows start icon and going into System, select Remote Desktop, click on "Select users that can remotely access this PC" under User accounts, click on Add, type in "Domain Users", hit "Check Names" and press OK. Now all domain users are allow to log into the Client VM.

<h3>Step 7: Creating many additional users and logging in as one of the users</h3>

Go into the Domain Controller VM, search up "Powershell ISE" in the Windows search bar, run it as administrator and copy and paste the script below into Powershell. 
<details>  
  <summary> <h5>pre-configured Powershell Script</h5> </summary>
  
  
```powershell
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
```
  
</details>
























