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

Next we need to switch the Domain Controller VM private IP address to static and this can be done by selecting the Windows Server VM, clicking on "Networking" then the blue text next to Network Interface, selecting "IP configuration", clicking on the IP address underneath the "Private IP address" column, switching it to "Static" and save.

<h3>Step 2: Enabling connectivity between the Domain Controller and Client</h3>

Log into Client VM by using Remote Desktop Connection, once there open up command prompt and ping the Domain Controller VM by its private IP address. You'll notice that it will respond with a timed out request meaning that Domain Controller's firewall is blocking any ICMP inbound traffic. To allow this kind of traffic and enable connectivity, log into the Domain Controller VM by using Remote Desktop Connection and search Windows Defender Firewall in the Windows search bar and select "Windows Defender Firewall with Advanced Security." Select "Inbound Rules" and sort by protocol by clicking on the protocol column. Then enable "Core Networking Diagnostic - ICMP Echo Request (ICMPv4-In)" both the private and domain by right clicking and selecting "Enable Rule". After this you will notice the ping requests will start working again in the Client VM.

<h3>Step 3: Installing Active Directory in our Domain Controller VM</h3>




