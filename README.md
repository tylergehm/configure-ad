  <img src="https://raw.githubusercontent.com/tylergehm/configure-ad/main/az1.jpg" alt="GitHub banner" style="max-width:100%;height:auto;" />
</p>

<h1>Configuring On-Premises Active Directory within Azure Virtual Machines</h1>
Microsoft Active Directory (AD) is a directory service developed by Microsoft for Windows domain networks. It acts as a centralized database that stores and manages information about network resources, such as users, computers, printers, and groups. AD allows administrators to organize, secure, and control access to these resources through policies and authentication. Users log in once with their credentials to access permitted resources across the network, making it easier to manage large organizations efficiently.<br />
</p>
In this Azure lab, two virtual machines are deployed: one running Windows Server 2025 configured as the domain controller and another running Windows 11 configured as the client. The domain controller is the central server that manages security authentication and authorization within a Windows domain—it hosts Active Directory, controls user logins, enforces policies, and (in this setup) also runs DNS services. The client is a workstation that joins the domain to access centralized resources and authentication. By default, the Windows 11 VM’s network interface card (NIC) uses Azure’s managed DNS. To enable domain joining, the NIC’s DNS settings are manually changed to point to the private IP address of the Windows Server 2025 VM, allowing the client to resolve the domain name and communicate with the domain controller for authentication and policy application.

</p>
Active Directory Domain Services (AD DS) is installed on the Windows Server 2025 VM, promoting it to a domain controller for a new forest named mydomain.com—a forest is the top-level container in Active Directory that holds one or more domains, defining the security and replication boundary for the entire directory structure. Next, the Windows 11 client VM joins the mydomain.com domain by using its updated DNS settings to locate the domain controller, enabling centralized authentication and group policy management. Remote Desktop is then configured on the client VM to allow non-administrative domain users to log in remotely. Finally, a PowerShell script automates the creation of additional user accounts in Active Directory, streamlining user provisioning and ensuring consistent setup across the lab environment.


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 11 Pro (21H2)
- Windows 11 Home (Host Machine)

<h2>Configuration Steps</h2>

- Step 1 - Create a Domain Controller Virtual Machine
- Step 2 - Create Client Virtual Machine
- Step 3 - Set Domain Controller VM IP Address to Static
- Step 4 - Set Client VM's DNS settings to Domain Controller's Private IP Address
- Step 5 - Install Active Directory
- Step 6 - Set up a new forest
- Step 7 - Create domain admin user
- Step 8 - Join a client to the domain
- Step 9 - Set up remote desktop for non-administrative users on Client VM
- Step 10 - Create additional users with Powershell Script


<h2>Deployment and Configuration Process</h2>

For a detailed explanation of creating Virtual Machines, please visit the project [Creating Virtual Machines in Azure Portal](https://github.com/tylergehm/vm)
</p>
<h2>Step 1- Create a Domain Controller Virtual  Machine</h2>
This project will begin by setting up a domain controller virtual machine (VM) in the Azure Portal.
</p>
<img width="1909" height="724" alt="image" src="https://github.com/user-attachments/assets/de7e043f-3239-41bd-ab84-3dcd8e4e95b0" />
Accessing the Resource Group page in the Microsoft Azure web portal to create a new resource group. </p>
A resource group in Microsoft Azure is a logical container that holds related resources—such as virtual machines, virtual networks, and public IP addresses—for a specific project or solution. In this Active Directory lab, the resource group will be used to organize and manage all components (the domain controller VM, client VM, network interfaces, and virtual network) in one place, enabling easy deployment, monitoring, cost tracking, and cleanup with a single deletion if needed. Creating it first ensures all subsequent resources are deployed into the same group and region for consistent connectivity and management. </p>

<img width="954" height="808" alt="image" src="https://github.com/user-attachments/assets/f24b12fa-7b37-4110-b605-3282b5ddceb0" />

The resource group is named "Active-Directory" and it has been placed in the US East 2 region. This means that all resources created within that group will be physically hosted in Microsoft’s data centers located in the eastern United States. In this lab, choosing US East 2 ensures both the domain controller and client VMs are co-located in the same Azure region for reliable Active Directory replication and domain join functionality.

</p>
<img width="1918" height="884" alt="image" src="https://github.com/user-attachments/assets/6cae6b76-e5cb-44dd-a646-2c09abe31196" />
The Virtual Network blade is accessed to create a VNet for the VMs used in the project. A virtual network (VNet) in Azure creates an isolated private network where the domain controller and client VMs reside, enabling secure, low-latency communication via private IP addresses—essential for DNS resolution, domain join, and Active Directory authentication. It acts as the on-premises network equivalent, ensuring both VMs can “see” each other without relying on public internet routing. </p>
In the Microsoft Azure web portal, the individual sections you navigate to—such as Virtual Machines, Virtual Networks, Resource Groups, Storage accounts, etc.— are officially called "blades". </p>

<img width="1037" height="846" alt="image" src="https://github.com/user-attachments/assets/1d6011ea-a54f-42b9-9bc4-7291be761932" /> </p>
The VNet was named "Active-Directory" and placed in the US East 2 region, which is where the resource group was created. </p>

<img width="1067" height="826" alt="image" src="https://github.com/user-attachments/assets/d9274cfe-9b3f-4948-91dd-521a233c8814" />
Inside the Active-Directory VNet, a subnet was created with the name "Active-Directory-Vnet". The subnet 10.0.0.0/16 defines a private IP address range (from 10.0.0.0 to 10.0.255.255) within the Active-Directory-Vnet virtual network, providing up to 65,536 usable IPs for VMs and services. This large, non-overlapping range ensures both the domain controller and client VMs can communicate securely using private IPs, mimicking an on-premises network while avoiding conflicts with Azure’s default address spaces. </p>


<img width="1895" height="756" alt="image" src="https://github.com/user-attachments/assets/a39cf354-bdaf-4a0c-a06a-dd7518ce0b6d" /> </p>
The next step, the Virtual Machine blade is accessed to create the VM that will be designated as the Domain Controller for the Active Directory domain that will be created in this project. </p>

<img width="1219" height="1012" alt="image" src="https://github.com/user-attachments/assets/dce37295-19a1-4e95-a0df-7d6f9b9881b0" /> </p>
The VM was named "DC-1", as it will become the designated Domain Controller for the Active Directory domain. The image selected for the VM was <b> Microsoft Server 2025 Datacenter </b. Microsoft Windows Server is required to host a domain controller because only it includes Active Directory Domain Services (AD DS), the proprietary service that manages centralized authentication, authorization, and directory replication for a Windows domain. Client OS versions like Windows 11 lack AD DS and cannot promote to domain controllers, limiting them to domain member roles. </p>

<img width="1448" height="885" alt="image" src="https://github.com/user-attachments/assets/7ef7cfee-ba14-4045-83b3-de09e53508b9" /> </p>
The DC-1 VM was set up so the network is in the Active-Directory VNet and subnet. </p>

<h2>Step 2 - Create Client Virtual Machine</h2>

<img width="1424" height="695" alt="image" src="https://github.com/user-attachments/assets/c0d905f9-f648-4ad9-85cc-f38308c4c9fd" /> </p>
The client VM was set up as "Client-1" using Windows 11 Pro disk image. The client VM was set to the Active-Directory Vnet and subnet. </p>


<h2>Step 3 - Set Domain Controller VM IP Address to Static</h2>
The domain controller’s NIC IP address must be set to static. By default, Azure assigns dynamic private IP addresses to newly created VMs. In this project, the domain controller’s IP is made static to prevent changes, since the client VM will rely on it as the DNS server. Without a fixed IP, DNS resolution and domain communication would break if the address changed. </p>

<img width="1900" height="799" alt="image" src="https://github.com/user-attachments/assets/03823e70-279d-42fb-8220-5a6150167bf4" /> </p>
DC-1 is accessed via the Virtual Machines blade. The first step to set the IP Address to static, the DC's Virtual NIC card must be accessed. This can be found in the drop down menu under Network Settings. </p>

<img width="1910" height="862" alt="image" src="https://github.com/user-attachments/assets/6207974c-a94e-4e27-a35b-665e4709947c" /> </p>
Inside the DC VM's Virtual NIC card, scroll down to "ipconfig1" and click on it to access the IP configuration. Here is where the IP address can be switched from dynamic to static. </p>

<img width="1477" height="492" alt="image" src="https://github.com/user-attachments/assets/22b9a740-692f-4245-934b-0e753a4c3496" /> </p>
Next, the DC VM will be accessed via Remote Desktop Connection using the VM's Public IP Address. </p>

<img width="469" height="302" alt="image" src="https://github.com/user-attachments/assets/5fe8a876-ec2c-408d-822c-ad13e794abd4" /> </p>

Once inside the DC VM, run the program "wf.msc" to open up Windows Firewall. For this lab's purpose, the firewall will be disabled to ensure there are no connectivity issues between the client, domain controller, and domain. </p>

<img width="1297" height="968" alt="image" src="https://github.com/user-attachments/assets/d8c734b0-bea4-4bef-b6f1-69b143908dbe" /> </p>

Inside Windows Firewall, click on properties and turn the firewall OFF. </p>

<h2>Step 4 - Set Client VM's DNS settings to Domain Controller's Private IP Address</h2>

The DNS settings on the Client Virtual NIC will be set to DC-1's Private IP address. This will allow the DC VM to act as a DNS server for the client VM. </p>

<img width="1907" height="625" alt="image" src="https://github.com/user-attachments/assets/71702b28-8e53-419c-9418-d2fb5675b582" /> </p>

DC-1's Private IP Address can be found in the DC-1 profile of the Virtual Machines blade. Once inside the profile, scroll down and find "Private IP Address" under Networking information. </p>

<img width="1906" height="608" alt="image" src="https://github.com/user-attachments/assets/46f84614-232b-422e-8c8f-fd43c2387b89" /> </p>

Return to the Virtual Machines blade to access Client-1's profile. From there, go to Network Settings and locate Client-1's Virtual NIC. </p>

<img width="1160" height="764" alt="image" src="https://github.com/user-attachments/assets/fe240d58-ecba-40ff-a82c-ea458f95e32d" /> </p>

Inside Client-1's Virtual NIC, locate the section "DNS Servers". Once the DNS Servers is accessed, click the button to switch from "Inherit Virtual Network" to "Custom". In the DNS Server field, enter DC-1's Private IP Address. This will change the client VM's DNS server from one provided by Microsoft to the Domain Controller. </p>

<img width="1870" height="505" alt="image" src="https://github.com/user-attachments/assets/999d722f-944d-4a1b-9898-c25371d8bcae" /> </p>

Remote Desktop Connection is used to access Client-1 VM. Once inside the VM, the Domain Controller will be pinged to ensure connection. </p>

<img width="1783" height="973" alt="image" src="https://github.com/user-attachments/assets/7810796a-a03f-4ffb-9afb-c9736090bbe4" /> </p>

Using Powershell, the client VM successfully pinged the DC VM by using the command "ping 10.0.1.4"; the Private IP Address of the Domain Controller. </p>

<img width="1476" height="788" alt="image" src="https://github.com/user-attachments/assets/45d25fe0-5ba0-49ec-b69d-9496a2a67b64" />

Next, the command "ipconfig /all" is entered to display information for all network adapters on a Windows system, including IP address, subnet mask, default gateway, DNS servers, MAC address, and DHCP status. The DNS server is displayed as "10.0.1.4", which is the Private IP Address for the Domain Controller. This indicates the the Domain Controller has been successfully configured as the DNS server for the client VM. </p>

<h2>Step 5 - Install Active Directory</h2>

<img width="1920" height="1078" alt="image" src="https://github.com/user-attachments/assets/2bacc799-421e-4517-a0b7-c5ec49c05656" />

To begin the process, using the Domain Controller VM, click on "Add Roles and Features" in the Dashboard of the Server Manager application. </p>

Server Manager is a Microsoft management console (MMC) tool designed exclusively for Windows Server operating systems because it provides centralized administration of server-specific roles, features, and services—like Active Directory, DNS, DHCP, and Hyper-V—that are not present in client versions such as Windows 11. Installing or running Server Manager on a client OS would serve no purpose, as those systems lack the underlying server infrastructure that Server Manager is designed for. </p>

<img width="1647" height="701" alt="image" src="https://github.com/user-attachments/assets/c8d3f94b-7c86-4d30-b398-b8d4a1effe39" /> </p>

<img width="981" height="695" alt="image" src="https://github.com/user-attachments/assets/da8c964d-8079-4fe2-a01d-a3811ee8b9af" /> </p>

Click next through the installation process, making sure the Privare IP Address for the Domain Controller is correct. In this case it is 10.0.1.4. </p>

<img width="977" height="689" alt="image" src="https://github.com/user-attachments/assets/69661e83-b473-4e1d-ae4c-be48c3c1cc84" /> </p>

In the server roles, select "Active Directory Domain Services". Selecting Active Directory Domain Services (AD DS) in Server Manager installs the core directory service components on the Windows Server, enabling it to function as a domain controller that stores the AD database, handles authentication requests, enforces security policies, and replicates directory data across the domain or forest. </p>

<img width="1650" height="701" alt="image" src="https://github.com/user-attachments/assets/b142274b-ff72-479b-be94-0aaef4ed45ee" /> </p>

Continue to click next through the sections of the installation process. </p>

<img width="974" height="696" alt="image" src="https://github.com/user-attachments/assets/1e2472f5-f940-470c-8de6-1f2b1b830460" /> </p>

In the Confirmation section, check the box to allow "Restart the destination server if required". In the confirmation pop-up box, click "Yes". </p>

<img width="980" height="699" alt="image" src="https://github.com/user-attachments/assets/ed86eb25-c694-43c9-b113-e6fbc57ecec3" /> </p>

Now that the installation is configured, click "Install" to begin the installation process.








