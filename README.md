  <img src="https://raw.githubusercontent.com/tylergehm/configure-ad/main/az1.jpg" alt="GitHub banner" style="max-width:100%;height:auto;" />
</p>

<h1>Configuring Active Directory within Azure Virtual Machines</h1>
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

- Step 1 - Create a Domain Controller Virtual Macine
- Step 2 - Set Domain Controller VM IP Address to Static
- Step 3 - Create Client Virtual Machine
- Step 4 - Set Client VM's DNS settings to Domain Controller's Private IP Address
- Step 5 - Set up Remote Desktop for non-administrative users on Client VM
- Step 6 - Install Active Directory
- Step 7 - Set up a new forest
- Step 8 - Create domain admin user
- Step 9 - Join a client to the domain
- Step 10 - Set up remote desktop for non-administrative users
- Step 11 - Create additional users with Powershell Script


<h2>Deployment and Configuration Process</h2>

For a detailed explanation of creating Virtual Machines, please visit the project [Creating Virtual Machines in Azure Portal](https://github.com/tylergehm/vm)
