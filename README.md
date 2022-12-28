<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory deployed in the Cloud (Azure)</h1>
Active Directory is a product by Microsoft that is for domain network services. It is native to Windows Server and is a common tool used in Information Technology today. There is an on premises version and an Azure version. In this tutorial, Active Directory was installed on a virtual machine within Azure.  <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>Configuration Steps</h2>

Step 1: Create resources

Step 2: Establish connectivity to between Domain Controller and Client

Step 3: Install Active Directory

Step 4: Create an administrator and regular account in Active Directory

Step 5: Join Client to the Domain Controller 

Step 6: Setup Remote Desktop for non-administrative users to Client

Step 7: Create users in Active Directory using Powershell script

<h3>Step 1: Create resources</h3>

First step is to create to two Azure virtual machines. If don't know how to create a virtual machine in Azure, please see my tutorial [here](https://github.com/klcarpio/Create-an-Azure-Account-and-Deploy-a-Virtual-Machine).

The first virtual machine is the domain controller (DC-1) which will have Windows Server 2022. A domain controller is the main server computer network that gives access to its domain resources and authenticates users that are connected to it. 

The second virtual machine is the client (Client-1) which will have Windows 10. The client is just any computer connected to the domain controller. 

Note: Ensure that Client-1 is on the same virutal network as DC-1. This is normally automatically done under the "Networking" tab but double check. 

<p>
<img src="https://i.imgur.com/XbuO2GD.png" height="80%" width="80%" alt="1. DC1 VM"/>
</p>

<p>
<img src="https://i.imgur.com/epKINcm.png" height="80%" width="80%" alt="2. DC1 VM"/>
</p>

Next is to ensure DC-1's private IP address is changed from dynamic to static. DC-1's private IP address needs to be static so it does not change during the course of this exercise. 

To do this, go to DC-1's "Networking" section and click on the virtual Network Interface Card (NIC).

<p>
<img src="https://i.imgur.com/EuDoPB6.png" height="80%" width="80%" alt="3. DC1 NIC"/>
</p>

From there, click on IP configurations. 

<p>
<img src="https://i.imgur.com/LMRzrup.png" height="80%" width="80%" alt="4. DC1 IP Config"/>
</p>

Click on the private IP address (10.0.0.4). 

<p>
<img src="https://i.imgur.com/UMIfZlx.png" height="80%" width="80%" alt="5. DC1 private IP"/>
</p>

Click the "Assignment" button to switch it from dynamic to static. 

<p>
<img src="https://i.imgur.com/awI6Rpz.png" height="80%" width="80%" alt="6. Change dynamic to static"/>
</p>

<h3>Step 2: Establish connectivity to between Domain Controller and Client</h3>
Second step is establish connectivity between Client-1 and DC-1. Use Microsoft Remote Desktop to connect to both virtual machines. 
<br>
Open Client-1 and open the Command line. Type in the command "ping -t" + the private IP address of DC-1. In this case, it is 10.0.0.4. The "ping -t" command will continously ping DC-1. 

<p>
<img src="https://i.imgur.com/hhelYIM.png" height="80%" width="80%" alt="7. Ping DC1 from Client1"/>
</p>

Due to the firewall blocking incoming Internet Control Message Protocol (ICMP) traffic, the request is timing out. To fix this, login into DC-1 and open the application Windows Defender Firewall with Advanced Security from the Start menu. 

<p>
<img src="https://i.imgur.com/axnzdcP.png" height="80%" width="80%" alt="8. DC1 Firewall 1"/>
</p>

<p>
<img src="https://i.imgur.com/gqQwYO8.png" height="80%" width="80%" alt="9. DC1 Firewall 2"/>
</p>

From here, sort the list by protocol so it is easier to see. Scroll down to ICMPv4 and enable these two inbound rules. 

<p>
<img src="https://i.imgur.com/biR5Bf1.png" height="80%" width="80%" alt="10. ICMP enabled"/>
</p>

Switch back to Client-1 and the Command line will start to ping DC-1 successfully. 

<p>
<img src="https://i.imgur.com/IeA89LA.png" height="80%" width="80%" alt="11. DC-1 Client1 connectivity"/>
</p>

<h3>Step 3: Install Active Directory</h3>

Third step is to install Active Directory Domain Services. Go back into DC-1 Open the Server Manager in DC-1. This should normally open whenever DC-1 is logged into for the first. It can also be found in the Start menu.

<p>
<img src="https://i.imgur.com/hJxsS51.png" height="80%" width="80%" alt="12"/>
</p>

In Server Manager, click "Add roles and features." Follow the prompts. At "Server Roles", check "Active Directory Domain Services." Then click "Add Features."

<p>
<img src="https://i.imgur.com/oF9MYOM.png" height="80%" width="80%" alt="14"/>
</p>

<p>
<img src="https://i.imgur.com/QYvvnoF.png" height="80%" width="80%" alt="15"/>
</p>


Continue to follow all the installation prompts until it is done. 

<p>
<img src="https://i.imgur.com/0LSc0Wi.png" height="80%" width="80%" alt="16"/>
</p>

Once the installation is done, back at the Server Manager Dashboard, click the flag with the yellow hazard sign underneath. Then click "Promote this server to a domain controller."

<p>
<img src="https://i.imgur.com/a9q1YeQ.png" height="80%" width="80%" alt="17"/>
</p>

<p>
<img src="https://i.imgur.com/lPegL0Q.png" height="80%" width="80%" alt="18"/>
</p>

Click "Add a new forest" and type in the root domain. In this example, it was mydomain.com.

<p>
<img src="https://i.imgur.com/0T7J6hZ.png" height="80%" width="80%" alt="19"/>
</p>

Follow all the prompts to finish the installation. DC-1 will restart to add all the updates. It may take some time to update and log back in. 

<p>
<img src="https://i.imgur.com/0qO8sIv.png" height="80%" width="80%" alt="20"/>
</p>

<p>
<img src="https://i.imgur.com/DYXJjRX.png" height="80%" width="80%" alt="21"/>
</p>

<h3>Step 4: Create an administrator and regular account in Active Directory</h3>
Fourth step is to create an administrator account and a regular account in Active Directory. 

Once DC-1 has restarted, open up Server Manager, click "Tools" in the upper right corner, and select "Active Directory Users and Computers."

<p>
<img src="https://i.imgur.com/8Byqn45.png" height="80%" width="80%" alt="22"/>
</p>

In Active Directory Users and Computers, right click the domain (mydomain.com), go to "New" and "Organizational Unit." Create two organizational units for administrators (_ ADMINS) and employees (_ EMPLOYEES). Once done, right click mydomain.com and click referesh to sort the new organizational units to the top. 

<p>
<img src="https://i.imgur.com/RtW3ahG.png" height="80%" width="80%" alt="23"/>
</p>

<p>
<img src="https://i.imgur.com/HhrKred.png" height="80%" width="80%" alt="24"/>
</p>

<p>
<img src="https://i.imgur.com/MiJ08vZ.png" height="80%" width="80%" alt="25"/>
</p>

<p>
<img src="https://i.imgur.com/CiknW69.png" height="80%" width="80%" alt="26"/>
</p>

Click on the _ ADMINS organizational unit and right click into the right window pane. Go to "New" and click "User." Fill in the boxes to create a new user. In this example, "Jane Doe" was used and the login name is "jane_admin."

<p>
<img src="https://i.imgur.com/7OonWQN.png" height="80%" width="80%" alt="27"/>
</p>

<p>
<img src="https://i.imgur.com/UeEhhOd.png" height="80%" width="80%" alt="28"/>
</p>

<p>
<img src="https://i.imgur.com/cpGgmF1.png" height="80%" width="80%" alt="29"/>
</p>

Once the account has been created, right click it and go to "Properties." From there, click the "Member Of" tab, click the "Add..." button, type in domain and click the "Check Names" button. From there, click "Domain Admins" and press "OK" and "Apply" to exit out. 

<p>
<img src="https://i.imgur.com/haUxS8o.png" height="80%" width="80%" alt="30"/>
</p>

<p>
<img src="https://i.imgur.com/Bh813Yi.png" height="80%" width="80%" alt="31"/>
</p>

Log out of DC-1 as "labuser" and log back in as the administrator account that was created (jane_admin). 

<p>
<img src="https://i.imgur.com/RgmSMJB.png" height="80%" width="80%" alt="32"/>
</p>

<p>
<img src="https://i.imgur.com/33X3zMJ.png" height="80%" width="80%" alt="33"/>
</p>

<h3>Step 5: Join Client to the Domain Controller </h3>
Fifth step is to join Client-1 to DC-1. 

<br>

To do this, go back to the Azure Portal. Go to the Client-1 virtual machine and click on the "Networking" section in underneath "Settings" on the left hand side. 

<p>
<img src="https://i.imgur.com/jwMFN40.png" height="80%" width="80%" alt="34"/>
</p>

From there, click on "DNS Servers." Then click, "Custom." Type in DC-1's private IP address (10.0.0.4). 

<p>
<img src="https://i.imgur.com/2dPPj3C.png" height="80%" width="80%" alt="35"/>
</p>

<p>
<img src="https://i.imgur.com/1xed7dc.png" height="80%" width="80%" alt="36"/>
</p>

Restart Client-1 and log back in. Once logged in, right click the Start menu and click on "System."

<p>
<img src="https://i.imgur.com/xkU5A1U.png" height="80%" width="80%" alt="37"/>
</p>

On the right hand side, click "Rename this PC (advanced)." Then click the "Change..." button and type in the domain name (mydomain.com). 

<p>
<img src="https://i.imgur.com/Icy1PH1.png" height="80%" width="80%" alt="38"/>
</p>

Open up the Command line and type the command "ipconfig /displaydns." This will show all the Fully Qualified Domain Names associated to Client-1. It will show that the DNS Servers are associated to DC-1's private IP address. 

<p>
<img src="https://i.imgur.com/ozaaPMB.png" height="80%" width="80%" alt="39"/>
</p>

<h3>Step 6: Setup Remote Desktop for non-administrative users to Client</h3>
Sixth step is to setup Remote Desktop for non-administrative users to Client-1. To do this, log into Client-1. This time, log in using the domain name and the admin account (mydomain.com\jane_admin).

<p>
<img src="https://i.imgur.com/33LPGMy.png" height="80%" width="80%" alt="40"/>
</p>

Once logged in, right click the Start menu and click "System." Then click "Remote desktop" on the right side. From there, click "Add..." and type in "Domain Users." Click "Check Names" and click "OK" to exit out. 

<p>
<img src="https://i.imgur.com/f2nqqY3.png" height="80%" width="80%" alt="41"/>
</p>

<p>
<img src="https://i.imgur.com/GNZgZQw.png" height="80%" width="80%" alt="42"/>
</p>

<p>
<img src="https://i.imgur.com/250dtjg.png" height="80%" width="80%" alt="43"/>
</p>

<h3>Step 7: Create users in Active Directory using Powershell script</h3>
Seventh and final step is to use Powershell to create users. A script was created by Josh Madakor that creates 10,000 users with the password "Password1."

The script can be found [here](https://github.com/joshmadakor1/AD_PS/blob/master/1_CREATE_USERS.ps1)

Go back into DC-1 and open Windows Powershell from the start menu. Right click it and "Run as administrator."

<p>
<img src="https://i.imgur.com/GM8VLuY.png" height="80%" width="80%" alt="44"/>
</p>

<p>
<img src="https://i.imgur.com/oLTbGLc.png" height="80%" width="80%" alt="45"/>
</p>

Copy and paste the script into a new Powershell console. I modified the script to only create 10 users so it is easier to manage and play around with. 

<p>
<img src="https://i.imgur.com/2Yot0nU.png" height="80%" width="80%" alt="46"/>
</p>

Once the users have been created, go back to Active Directory Users and Computers. All the users that were created were placed into the _ EMPLOYEES organizational unit. Choose one user (fihapi.nile) and log into Client-1 with the user. Remember the password is Password1. 

<p>
<img src="https://i.imgur.com/QzRho2A.png" height="80%" width="80%" alt="47"/>
</p>

<p>
<img src="https://i.imgur.com/DgfxJtj.png" height="80%" width="80%" alt="48"/>
</p>

<h3>Bonus Step: How to unlock users' accounts and reset passwords</h3>
In order to unlock a user's account, right click the user account and click "Properties." 
Click on "Unlock Account." You can also right click the user account and "Reset Password..."

<p>
<img src="https://i.imgur.com/HTcYBBU.png" height="80%" width="80%" alt="49"/>
</p>

<p>
<img src="https://i.imgur.com/lNfDusu.png" height="80%" width="80%" alt="50"/>
</p>

<p>
<img src="https://i.imgur.com/HrMlyi7.png" height="80%" width="80%" alt="51"/>
</p>

Thank you for checking out my Active Directory tutorial!
