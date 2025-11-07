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
- Step 6 - Set up a new Forest
- Step 7 - Create domain admin user within the domain
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

Now that the installation is configured, click "Install" to begin the installation process. </p>

<img width="975" height="702" alt="image" src="https://github.com/user-attachments/assets/ee59682e-d1ab-4a55-be79-2d815289a35d" /> </p>

The installation of Active Directory Domain Services is now complete. </p>

<h2>Step 6 - Set up a new Forest</h2>

<img width="1884" height="959" alt="image" src="https://github.com/user-attachments/assets/a334f4b2-f8ec-4a89-8dbd-b9cb3889924a" /> </p>

After the installation is complete, return to the Server Manager dashboard and click the notifications button. In the drop down of notifications, click "Promote this server to a Domain Controller". </p>

<img width="939" height="699" alt="image" src="https://github.com/user-attachments/assets/66d29557-465c-448d-b87a-f9aaec47453a" />

Once inside the Configuration Wizard, click the button that says "Add a new forest". In Active Directory, a forest is the top-level container that defines the ultimate security and administrative boundary for one or more domains. It establishes a shared schema, configuration, and global catalog, enabling trust relationships and centralized management across all domains within it—making it the highest logical structure in an AD environment. </p>

For this project, the forest will be named <b>mydomain.com</b>. Once completed, click next to continue. </p>

<img width="941" height="695" alt="image" src="https://github.com/user-attachments/assets/9331b8c4-a0d9-410b-8b28-f555f5702118" /> </p>

In this next section, a password must be entered for Directory Services Restore Mode. The Directory Services Restore Mode (DSRM) password is a local administrator credential set during AD DS promotion that allows you to boot the domain controller into a special recovery mode to repair or restore the Active Directory database (ntds.dit) if it becomes corrupted or needs offline maintenance. It functions independently of domain accounts, ensuring access to the server even when the AD database is unavailable. </p>

Once a password has been entered, click Next. </p>

<img width="937" height="695" alt="image" src="https://github.com/user-attachments/assets/c07da5a2-9344-4521-a61a-c38420a423a9" />

In the DNS Options section, uncheck the box that says "Create DNS Delegation". Unchecking "Create DNS Delegation" during AD DS promotion prevents the new domain controller from attempting to register a delegation record in a parent DNS zone (which doesn’t exist in this isolated Azure lab). Since this is a standalone forest with no upstream DNS server managing a parent zone, creating a delegation would fail or cause unnecessary errors—leaving it unchecked keeps the installation clean and focused on internal DNS resolution within the new domain. </p>

<img width="951" height="698" alt="image" src="https://github.com/user-attachments/assets/35cc319f-48ed-4b3a-9445-dbf0ed1373a9" /> </p>

Next, the NetBIOS name is checked to ensure that it is correct. The NetBIOS name is checked during AD DS promotion to confirm the legacy-compatible identifier (typically the first part of the domain name, like MYDOMAIN) that older systems and applications use for name resolution and authentication. Verifying it ensures compatibility with pre-Windows 2000 clients, avoids conflicts in the network, and prevents errors during domain join or replication. </p>

<img width="1649" height="695" alt="image" src="https://github.com/user-attachments/assets/3b026d22-562e-4f55-b111-ae827c51a4a5" /> </p>

Click next through Paths and Review Options. </p>

<img width="946" height="701" alt="image" src="https://github.com/user-attachments/assets/6aad9579-8e8d-421c-a711-14afe0ae9dbf" /> </p>

Once all prerequisite checks have been passed, click "Install" to begin installation of the forest. </p>

<img width="557" height="672" alt="image" src="https://github.com/user-attachments/assets/d8a44fd7-72d3-416f-9eb4-06be7cc8b610" />
</p>

After the forest is installed, the DC VM will restart to finish the installation. When logging back in, change the credentials to include the domain now. In this project, the DC login in will now become "mydomain.com\DC-1" instaed of just "DC-1". </p>

<h2>- Step 7 - Create domain admin user within the domain</h2>

<img width="1644" height="906" alt="image" src="https://github.com/user-attachments/assets/9325d624-c315-476b-95e3-818a89c600d1" />

This step of the project begins by opening up "Active Directory Users and Computers" in the DC VM. </p>

<img width="934" height="655" alt="image" src="https://github.com/user-attachments/assets/2666325f-bd11-4f00-8038-131608c72244" />

Right click on the "mydomain.com" directory, go to New, and click on Organizational Unit. An Organizational Unit (OU) in Active Directory is a container object used to organize and group users, computers, groups, or other OUs within a domain for easier management. It allows administrators to apply Group Policy Objects (GPOs) and delegate administrative control at a granular level, rather than applying settings domain-wide. </p>

<img width="1084" height="465" alt="image" src="https://github.com/user-attachments/assets/183e565c-97c7-4fad-a2d5-14ea23381844" /> </p>

Create a new organizational unit for "EMPLOYEES" and for "ADMINS". These are the OUs where the Active Directory Users will be created. </p>

<img width="1050" height="649" alt="image" src="https://github.com/user-attachments/assets/3559573b-a15e-4d44-88aa-15bb53dfb1fc" /> </p>

A new admin user will be created. This will be done by going to the ADMINS OU folder that was just created. Right click inside the folder, go to New, then click on User. </p>

<img width="1081" height="463" alt="image" src="https://github.com/user-attachments/assets/11000b9a-63b0-41b5-b742-8c37c86b1a51" /> </p>

Fill out the forms for the new admin user. In the case of this lab, uncheck the box that says "User must change password at next login". </p>

<img width="942" height="624" alt="image" src="https://github.com/user-attachments/assets/d961a2bf-e5d6-4db0-a82e-da6da0d0cdb8" /> </p>

The new admin user will show up in the ADMINS folder. Right click on the user's name and click on "Properties". 

<img width="1263" height="661" alt="image" src="https://github.com/user-attachments/assets/a538fdea-5473-448e-aeb5-9b2499af0284" /> </p>

Inside of Properties, click on the tab called "Member Of" then click "Add...". Inside the pop up box, add the user to the Domain Admins group. </p>

<img width="506" height="662" alt="image" src="https://github.com/user-attachments/assets/d1a98b28-e360-42e9-928c-55156cd8703c" /> </p>

After filling out the pop-up box, verify that the user has been added to the group and select "Apply". </p>

Once this has been configured, log out of the DC VM remote connection. 

<img width="549" height="668" alt="image" src="https://github.com/user-attachments/assets/d04f7ddb-c9ae-4451-b716-3ec11dbcc088" /> </p>

Now use Remote Desktop Connection to log in to the Jane User Admin account. </p>

<img width="1473" height="531" alt="image" src="https://github.com/user-attachments/assets/730bbfa4-e191-4920-952f-304868686476" /> </p>

The new account was successfully remotely created and connected. This was verified in the Command Prompt with the command "whoami". </p>

<h2>Step 8 - Join a client to the domain</h2>

A computer is called a client in a domain because it joins the domain and relies on the domain controller for centralized authentication, policy enforcement, user profile management, and access to domain resources—acting as a dependent workstation rather than an authority. Unlike the domain controller (which hosts Active Directory and provides services), the client participates in the domain to receive security and configuration instructions, enabling users to log in with domain credentials and access shared resources consistently across the network. </p>

<img width="1867" height="1049" alt="image" src="https://github.com/user-attachments/assets/1c4a7b81-ad57-4a38-ba7a-552444ac34a8" /> </p>

Using the Client VM, log into the VM and right-click on the Start Menu icon. Select "System". </p>

<img width="1250" height="986" alt="image" src="https://github.com/user-attachments/assets/dbe7c105-148c-4227-9d6e-ac2f056ca0b4" />


Once inside System, click on the link that says "Domain or Workgroup". </p>

<img width="539" height="565" alt="image" src="https://github.com/user-attachments/assets/b8274e9f-977c-4f51-ae4b-bc7a9381b48f" />

Select the button that says "Change..." </p>

<img width="411" height="473" alt="image" src="https://github.com/user-attachments/assets/e1365503-ddcd-4931-b0f5-7377d5d6ce39" />

Inside the Computer Name/Domain Changes pop up box, select the button that says "Domain" and enter the Domain Address (mydomain.com). Then select "Okay". </p>

<img width="563" height="436" alt="image" src="https://github.com/user-attachments/assets/cfb27afd-0665-4743-99d8-077b31ccccd6" /> </p>

Enter in the Domain Controller authentication credentials. </p>

<img width="367" height="207" alt="image" src="https://github.com/user-attachments/assets/8da01694-150f-41d5-ac05-e64a087af517" /> </p>

The Client VM has successfully joined the Domain; the VM must be restarted to complete the process. </p>

<img width="935" height="651" alt="image" src="https://github.com/user-attachments/assets/0bfb9ba8-8db6-4444-a1d1-8ac4965cd53f" /> </p>

As the Client VM is restarting, return to the DC VM and open up the Active Directory Users and Computers. Once the application has opened, a new OU called "Clients" is created. </p>

<img width="935" height="657" alt="image" src="https://github.com/user-attachments/assets/cd32e68e-5d34-44a6-b89b-62c058443dda" /> </p>

Clicking on the "Computers" folder, Client-1 is now officially joined into the Domain. </p>

<img width="932" height="653" alt="image" src="https://github.com/user-attachments/assets/504058be-a06f-42cd-9d20-6fe6c2d47620" /> </p>

Drag and drop Client-1 into the CLIENTS folder in the domain. </p>

<img width="556" height="681" alt="image" src="https://github.com/user-attachments/assets/123225d1-a663-474b-8960-93f2723349a4" /> </p>

Now, using Remote Desktop Connection, the Client-1 Public IP Address will be used to log in to the Domain as Jane (Admin). This is to verify that the Client-1 VM has joined the Domain.

<img width="1143" height="918" alt="image" src="https://github.com/user-attachments/assets/e8272d58-43a5-4f60-94e4-93610894169b" />

Successfully logged in as Jane using the Client-1 VM.

<h2Step 9 - Set up remote desktop for non-administrative users on Client VM></h2>

In this step, all domain users will be given access to remotely connect into the Client-1 VM. The Client-1 VM currently has Jane logged in, which is appropriate because this account has Administrative privileges to set up RDC access. </p>

<img width="1918" height="1079" alt="image" src="https://github.com/user-attachments/assets/93861306-9311-4a49-b50e-84b5af762993" /> </p>

This process will begin by right-clicking on the Start menu and selecting "System". </p>

<img width="1249" height="990" alt="image" src="https://github.com/user-attachments/assets/a16772ba-9b92-477f-9102-151e7391f378" /> </p>

Scroll down in the About section of System and locate "Remote Desktop", then click on it. </p>

<img width="1261" height="978" alt="image" src="https://github.com/user-attachments/assets/d516b412-659b-4a31-b2a2-784686e2e29f" /> </p>

Next, select "Remote Desktop Users". </p>

<img width="494" height="406" alt="image" src="https://github.com/user-attachments/assets/97be5041-7f70-47d2-b028-6b45a6370ad9" /> </p>

In the Remote Desktop Users pop-up, click on "Add..." </p>

<img width="608" height="309" alt="image" src="https://github.com/user-attachments/assets/69a42bb6-c130-4e56-ba0d-4c97d73447c7" />

Add "Domain Users" in the text box and click "Okay". </p>

<img width="502" height="409" alt="image" src="https://github.com/user-attachments/assets/07826b9b-4787-4ea8-aff5-3bd38d342d83" /> </p>

All Domain Users have now been given access to remotely connect into the computer, the Client-1 VM. </p>

<h2>Step 10 - Create additional users with Powershell Script</h2>

This step will involve using a Powershell script to generate users into the domain. </p>
PowerShell is a powerful scripting language and command-line shell developed by Microsoft, built specifically for system administration and automation on Windows — it allows complex tasks like creating thousands of Active Directory users to be executed efficiently with logic, loops, and integration with AD modules. </p>
This script automates the creation of up to 1,000 realistic-looking user accounts in Active Directory by generating random first and last names using alternating consonants and vowels (e.g., jad.kim), combining them into a username (e.g., jad.kim), and assigning each a fixed password (Password1). It uses the New-ADUser cmdlet to create enabled accounts with full names, display names, and places them in an OU called _EMPLOYEES at the domain root.  </p>

[Note: This script was created by Josh Madakor for the Information Technology course on the Course Careers learning platform.] </p>


