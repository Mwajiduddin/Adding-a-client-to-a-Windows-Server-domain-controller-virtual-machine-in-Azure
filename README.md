<h1 align="center">Adding a client to a Windows Server domain controller virtual machine in Azure</h1>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/img_activedirectory-page.png" height="65%" width="40%" />
</p>

<h1>Introduction</h1>
In this tutorial we will launch a Windows Server virtual machine and change it into a domain controller by installing Active Directory. We will also set up a client virtual machine to connect it with the domain controller and configure admin and user accounts in VMs.<br />

<h2>Tutorial Guidelines</h2>

<h3>Step 1: Creating two virtuals machine in Azure</h3>

First we need two VMs: one running Windows Server 2022 (Domain Controller) and another running Windows 10 Pro (Client). I've detailed in the previous tutorials on how to create a virtual machine in Azure which you can find by clicking 
[here](https://github.com/Mwajiduddin/How-to-create-a-virtual-machine-in-Microsoft-Azure) and [here](https://github.com/Mwajiduddin/Observing-network-traffic-between-two-virtual-machines-in-Azure-using-Wireshark) but I'll to a quick rundown here too.

To start off let's first create our Windows Server VM, this is the VM that will have Active Directory installed and serve as our Domain Controller. So log into Microsoft Azure, go into "Virtual machines", name the VM, select Windows Server 2022 next to "Image" and select a region. You might be wondering why I didn't create a resource group first like I did in the previous tutorials, here you can make and name one by clicking on "Create new" next to "Resource group". What I showed in the previous tutorials was to help get a better understanding on the process of creating one. Choose a size with atleast two VCPUs otherwise it will be slow, create a username and password and keep note of it, check the Licensing box, hit "Review + create" and after validation click on "Create." 

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c1.png" />
</p>


While this VM is being creating, we can start making our second VM which will serve as our Client so the same process as last time only for "Image" select Windows Pro 10, remember to select the same resource group as the first VM you made and for ease of use you can choose the same username and password as the first VM you made.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c2.png" />
</p>

Next we need to switch the Windows Server VM private IP address to static and this can be done by selecting the Windows Server VM, clicking on "Networking" then the blue text next to Network Interface, selecting "IP configuration", clicking on the IP address underneath the "Private IP address" column, switching it to "Static" and saving it.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c3.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c4.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c5.png" />
</p>

<h3>Step 2: Enabling connectivity between the Domain Controller and Client virtual machines</h3>

Log into your Client VM by using Remote Desktop Connection, once there open up command prompt and ping the Windows Server VM by its private IP address. You'll notice that it will respond with a timed out request meaning that Windows Server's firewall is blocking any ICMP inbound traffic. 

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c6.png" />
</p>

To allow this kind of traffic and enable connectivity, log into the Windows Server VM by using Remote Desktop Connection and search Windows Defender Firewall in the Windows search bar and select "Windows Defender Firewall with Advanced Security." Select "Inbound Rules" and sort by protocol by clicking on the protocol column. Then enable "Core Networking Diagnostic - ICMP Echo Request (ICMPv4-In)" both the private and domain by right clicking and selecting "Enable Rule". 

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c7.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c8.png" />
</p>

After this you will notice the ping requests will start working again in the Client VM.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c9.png" />
</p>

<h3>Step 3: Installing Active Directory in Windows Server VM</h3>

Open Server Manager in your Windows Server VM, select "Add roles and features", click Next for each section, once you are in "Server Roles" select "Active Directory Domain Services" and "Add Features" on the pop-up window and finish installing it. 

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c10.png" />
</p>

Then go to the top right flag with the yellow exclamation point, click on the blue text "Promote this server to a domain controller", then select "Add a new forest" on the "Deployment Configuration" window and name your root domain (make it simple). Make a password under "Domain Controller Options" (choose the same password you made for Windows Server VM for simplicity) and finish installing it.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c11.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c12.png" />
</p>

Once done installing our Windows Server VM is now a domain controller and you'll notice that it will automatically log you off so just log back in but this time our username will be different. Your username will be the (name of your root domain + backslash + VM's username) such as below.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c13.png" />
</p>


<h3>Step 4: Creating Organizational Units (OU) and logging in as an Admin in Active Directory</h3>

In this step we will be creating a couple of OUs which are basically folders in Active Directory. So after logging into your Domain Controller VM, open up Server Manager, click on "Active Directory Users and Computers" then right click on your root domain and select "Organizational Unit" and name one as "EMPLOYEES" and another as "ADMINS."

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c14.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c15.png" />
</p>

Now we are going to create our own Admin account so go to the ADMINS folder, right click, hover over to New, click on User then name your Admin, user logon account, your password and uncheck the box that says "User must change password at next logon" and hit Finish. 

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c16.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c17.png" />
</p>

But the user we just made isn't an Admin just yet so right click on the user account, select Properties, click on the "Member Of" tab and click on Add. A window will appear, at the bottom where it states to enter object names type in "Domain Admin" in the box, click on "Check Names" and then press OK. Then log out of the VM and log back in this time as the Admin you created, the username will be the (name of your root domain + backslash + admin username).

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c18.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c19.png" />
</p>


<h3>Step 5: Joining the Client VM to the Domain Controller</h3>

In this step we are going to use the Domain Controller as Client's DNS server instead of the virtual DNS server assigned to the Client VM and we start off in the Azure portal and copy the private IP address for Domain Controller VM, head over to Networking under Settings for Client VM, click on the blue text next to "Network Interface", then go into DNS server under Settings, select Custom and type in private IP address for the Domain Controller VM and hit Save.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c20.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c21.png" />
</p>


Then go back to Virtual machines, select your Client VM and click Restart and log back in.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c22.png" />
</p>

To double check, open command prompt and type in the command ipconfig /all and you can see that the DNS server IP address is the same as the Domain Controller VM's private IP address.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c23.png" />
</p>

To join this Client VM to the Domain Controller, right click the Windows start icon and click on System, go to "Rename this PC (advanced)" then click "Change", select Domain, type in your root domain and then a window will pop up asking for a username and password where you will enter the credentials of the Domain Admin account you created.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c24.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c25.png" />
</p>

After entering your credentials you will be prompted to restart the VM, now when you boot up Client VM it will be a member of the Domain Controller and you can log in the Client VM as the Domain Admin User you created in the previous step.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c26.png" />
</p>


<h3>Step 6: Logging into Client VM as a non-administrator user</h3>

We've showcased from the previous step that we can now log into the Client VM from an administrator account, in this step we are going to configure the Client virtual machine to let us log in as a normal domain user. We start off by right clicking the Windows start icon and going into System, select Remote Desktop, click on "Select users that can remotely access this PC" under User accounts, click on Add, type in "Domain Users", select "Check Names" and press OK. Now all domain users are allowed to log into the Client VM.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c027.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c28.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c29.png" />
</p>

<h3>Step 7: Generating many additional users in Powershell and logging in as one of the users</h3>

Go into the Domain Controller VM, search up "Powershell ISE" in the Windows search bar, run it as administrator, copy the script below then click on the file icon on the top left corner and paste the script's content and hit the green play icon. Essentially we are creating 1,000 different users each with the password of "Password1" and you can see this populated in the "EMPLOYEE" OU.


<details>  
  <summary> <h5>Non-administrator users generation Powershell script</h5> </summary>
  
  
```powershell
 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 1000
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
               -Path "ou=EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}
```
  
</details>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c30.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c31.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c32.png" />
</p>

As this script is running, go to "Active Directory Users and Computers", refresh the "EMPLOYEES" folder and you can see it being populated by numerous different users. Log off of the Client VM and then choose one of the generated users that you would like to log back into the Client VM, remember the password is "Password1." You can double check by going into command prompt and typing in the "whoami" and "hostname" commands.

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c33.png" />
</p>

<p align="center">
<img src="https://github.com/Mwajiduddin/Mwajiduddin/blob/main/images/c34.png" />
</p>


Don't exit out of your virtuals machines and delete your resource groups in Azure just yet because if you want to learn how to configure file shares and its permissions in an Active Directory and client environment you can segue into the next tutorial [here](https://github.com/Mwajiduddin/Network-File-Shares-and-Permissions) where you can learn to do so. 




