# Server-and-Active-Directory-Homelab
A homelab created to emulate a server/Active Directory Setup.

In this lab, we will be creating 1 Windows virtual machine running Windows Server 2025. This server will act as a domain, DHCP, and DNS server. Aside from the server, there will be ten computers with Windows 11.

The first screen we are presented with after installation is the Server Manager.

<img width="981" height="640" alt="image" src="https://github.com/user-attachments/assets/4e7816a2-9fdb-4110-b85d-eb9840c39cdb" />

We do not currently have any roles configured, and would need to first configure the domain role. Before that, however, we'll rename our PC to something more recognizable. I chose to rename it to DanDomainPC. After renaming the server and restarting it, there is one more thing we must do. 
Since we will be configuring this PC as a DHCP server, we need to set the IP address of this device to be a static address. Since we are running this on VMWare, the default gateway will need to be .2 instead of .1. Our DNS server will be set as the IP of our PC, since that is what will be using as our DNS server. 

<img width="390" height="447" alt="image" src="https://github.com/user-attachments/assets/51437b35-7335-4177-bab9-fb697b3d064c" />

Now that our PC has been renamed and our static IP/DNS server has been set, we are now ready to configure our roles. 
Go to 'Manage' at the top toolbar of the server and select 'Add Roles and Features'. We want to select our DanDomainPC server. For the server roles we will be selecting 'Active Directory Domain Services', 'DHCP Server', and 'DNS Server.'

<img width="340" height="157" alt="image" src="https://github.com/user-attachments/assets/4e2a2b6f-56e7-43b4-b90d-16e601038a7f" />

<img width="1433" height="62" alt="image" src="https://github.com/user-attachments/assets/5455b0fb-5ead-4ed7-a446-118e91683a3a" />

<img width="784" height="552" alt="image" src="https://github.com/user-attachments/assets/89bdf49d-f4be-47fc-aa43-c8b31e191963" />

For the domain, we will be creating a new forest, since we do not have a previously setup forest to attach to. I named the domain danp.lab for the purposes of this exercise. 

<img width="757" height="554" alt="image" src="https://github.com/user-attachments/assets/717030cb-42fc-4de2-abe0-bb629a7f73e5" />

Next, it wants admin credentials to authorize the DHCP server. In this case, I used alternate credentials, since my original account did not have admin privileges.

<img width="754" height="550" alt="image" src="https://github.com/user-attachments/assets/34fd7708-f3c6-4d5e-8b47-07bf9a64e87c" />

Now it prompts us to add a scope. I made the scope 192.168.170.135 - 192.168.170.254. My DHCP server's current address is 192.168.170.128, so this would not be included in the scope. This is by design. You do not want servers to be grabbing DHCP addresses. They need to be statically set. 
Later on, if we wanted to reserve specific IP addresses for printers, we could create a new reservation in DHCP, and it would hold that IP address for the printer.

<img width="319" height="283" alt="image" src="https://github.com/user-attachments/assets/a186c6c6-47e2-420b-a522-55f1215d483e" />

Upon logging into the one Windows 11 workstation that is setup currently and doing an ipconfig /release and ipconfig /renew to grab a new lease, we have verified that we successfully obtained an IP address from the DHCP server. We can also perform an nslookup on the IP address of our server to verify it is seeing it properly as the DNS server.

<img width="662" height="488" alt="image" src="https://github.com/user-attachments/assets/123576e2-d70d-49a9-bb05-9687b5032691" />

<img width="445" height="226" alt="image" src="https://github.com/user-attachments/assets/277f1824-be1b-4c85-8116-a420d49c4b22" />

Currently, we only have one Windows 11 PC, but we need ten by the end of the lab. It took 30 minutes to install a Windows 11 PC from the .iso file. Are we going to manually install it ten times and have it takes at least five hours? Isn't there a faster way? Turns out, there is. VMWare will allow us to take a snapshot of our image.
We must, however, strip our VM down to a general image in order to replicate it. To do this, we must go to the command prompt and enter the following command 

**C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown**

Now we can take the snapshot and convert it to a template for cloning. Cloned templates take only a few minutes to install, drastically cutting down the time and improving productivity.

<img width="678" height="270" alt="image" src="https://github.com/user-attachments/assets/d41d390a-84b3-4b37-9dde-bdf2bde5b6ca" />

<img width="662" height="506" alt="image" src="https://github.com/user-attachments/assets/215e46fd-a6dd-422a-87e3-63a84318fdb8" />

Now that the PCs are imaged, we need to join them to the domain. It was here that I investigated to see if there was a way to join them to the domain in bulk. As it turns out, there is, but it was not a net productivity benefit to do it this way. It would have been faster to just go to each individual device and join them manually.
To bulk join them, I need to enable the PSRemoting on the PCS, set the network profile to private, enable the local admin, and in my powershell script, I had to put the IP addresses of every device in a foreach statement.

<img width="938" height="541" alt="image" src="https://github.com/user-attachments/assets/1d95523e-f5e3-404a-948f-b78a4f310278" />

<img width="650" height="439" alt="image" src="https://github.com/user-attachments/assets/7f2e501a-6670-49ed-b65b-b3be208e5859" />


Now that the PCs were created, the next goal on the agenda was to create 60 different users in AD. For this I created another bulk script that imported this information from a CSV file.

<img width="722" height="415" alt="image" src="https://github.com/user-attachments/assets/48fe647d-38e7-4e2f-a39f-3ed283592e99" />

What this script did was to read through each entry in the CSV and create users in Active Directory with the information that corresponds to the properties in the foreach statement. This script also moved them into the OUs that were specified in the CSV, which were Sales, HR, and Research.
For security purposes, I used Read-Host and -AsSecureString so that the password had to be typed in encrypted characters instead of using ConvertTo-SecureString and -AsPlainText, which would have revealed the password in plain text.

<img width="530" height="283" alt="image" src="https://github.com/user-attachments/assets/c15f0a43-74a8-419e-a026-2bfb5510f4bb" />

<img width="231" height="530" alt="image" src="https://github.com/user-attachments/assets/12532a6f-9610-4a05-814f-34758a59e373" />


To verify that the users were created, we can simply check the respective OUs in Active Directory. In a production environment, these users would be tied to several groups that have GPOs tied to that. In this way, user's access can be controlled to only have access to what they need to do their jobs. This is called 'The Principle of Least Privilege' in the security world.

<img width="434" height="438" alt="image" src="https://github.com/user-attachments/assets/aa8754e3-c8ae-4778-9fa7-f528af5faa40" />

<img width="401" height="446" alt="image" src="https://github.com/user-attachments/assets/5561b94a-2fef-4bcd-bdc1-884bbd3d508b" />

<img width="421" height="427" alt="image" src="https://github.com/user-attachments/assets/e1ab3502-640a-4f14-96a3-c267f8bbd0d1" />

Earlier, we had created endpoints through a combination of VM snapshots and cloning, combined with a powershell script. However, we could deploy a more efficient environment for mass deployment and endpoint management through the tool called Microsoft Endpoint Configuration Manager, or MECM for short. To setup MECM, we will need a domain server, a database server, and of course the MECM server.
On the domain server, we will need to extend the Active Directory schema so that it can communicate with the MECM Server. We can do this by navigating to the …\SMSS\Bin pathway in the command prompt through the cd (current directory) command and then running extadsch.exe

<img width="894" height="116" alt="image" src="https://github.com/user-attachments/assets/c50648e3-34c6-4291-9fc4-92c7b9d175c6" />

We will also want to create an account for the administration of the database server and the MECM server. In this lab, I created an account called SQLAdmin for both and ensured it had admin rights to the MECM Server and SQL Server. However, in a real-world environment, for security purposes, you would want two separate accounts. I also created an account called sql_service for certain services on the database to use to run.

<img width="813" height="30" alt="image" src="https://github.com/user-attachments/assets/21175d24-ead2-4c33-bd7b-41cb9dd7625b" />
<img width="575" height="69" alt="image" src="https://github.com/user-attachments/assets/c606b07d-da27-4a3f-ab3f-de75087cbf5f" />

To setup the database, we first need to download SQL2022 ISO. Once downloaded, we need to mount the ISO file and run the setup. In the setup, we select New SQL Server standalone installation. Let the program run a check and if everything comes back okay, then we can move to the next step, which is selecting the instance features.

<img width="779" height="590" alt="Database1" src="https://github.com/user-attachments/assets/b36faebf-b52d-4b3d-b8a9-117da398b481" />
<img width="801" height="702" alt="Database2" src="https://github.com/user-attachments/assets/a5ef9991-1771-4df3-b52a-22407264401e" />

In this screen, we want to select to enable Database Engine Services. We can name the database instance with the default configuration. In the next screen, we will setup the server configuration. We can keep the defaults here. We will change them later under the SQL Server Configuration 2022. 

<img width="802" height="697" alt="Database3" src="https://github.com/user-attachments/assets/03388211-de5c-45b7-abc9-3e19b9b43af2" />
<img width="800" height="702" alt="Database4" src="https://github.com/user-attachments/assets/609661bb-6967-4b05-b43a-9508ea85e641" />
<img width="803" height="704" alt="Database5" src="https://github.com/user-attachments/assets/995b187f-2993-4276-b6b5-f98c353e4f29" />

From there, we can continue to click next and it will install the database. Once it has installed the database, we should have SQL Server Management Studio 22 and SQL Server 2022 Configuration Manager. First, let’s open SQL Server 2022 Configuration Manager and edit a couple of the settings. First, under SQL Services, we want to change the ‘Log On As’ under SQL Server and SQL Server Agent to the sql_service account we created in AD for this very purpose. We would also want to enable TCP/IP.

<img width="840" height="89" alt="Database6" src="https://github.com/user-attachments/assets/39525c4e-836d-40bc-b53a-dadfe4986269" />
<img width="534" height="189" alt="Database7" src="https://github.com/user-attachments/assets/b97f219c-67e5-47a7-9c24-35207e61d497" />

For SQL Server Management Studio 22, first we need to login under the account that has admin privileges to the database. In this case, that is the SQLAdmin account. We will opt to trust the certificate. Once logged in, we will need to do a few things. First, let’s expand the database and then the security option, then logins. We want to ensure that sql_service, SQLAdmin, and our MECMServer is added to the logins with administrator privileges to the database. This is to ensure the database and the MECMServer can communicate with each other properly. To add the computer DanMECMServer, we will need to add a $ before it, so $DanMECMServer. The database won’t add the computer correctly without doing this.

<img width="508" height="712" alt="Database11" src="https://github.com/user-attachments/assets/0a480249-79bf-49fc-8054-566cfe876a5b" />
<img width="722" height="656" alt="Database8" src="https://github.com/user-attachments/assets/10db0955-cab6-4757-9e15-649b81c6724d" />
<img width="723" height="655" alt="Database9" src="https://github.com/user-attachments/assets/1aa3c0a5-79f4-4a7b-ae41-c17530e4b47d" />
<img width="267" height="274" alt="Database10" src="https://github.com/user-attachments/assets/464131ab-f3ed-4a67-9eea-0e308f452f34" />

Now we can configure our MECM Server. First, let’s ensure that our SQLAdmin account has admin privileges to the MECMServer as well. In real world environments, we would also have a separate account to have admin rights to the MECM server only. Now, we will download and install the Microsoft Endpoint Configuration Manager. We will need to fulfill several prerequisites before we can begin to install MECM. In Server Manager, the following Roles and Features will first need to be installed:
•	Web Server (IIS): Required for management points and site communication. Include these role services under IIS:
o	Common HTTP Features (Default Document, Static Content)
o	Health and Diagnostics (HTTP Logging, Logging Tools)
o	Performance (Static Content Compression)
o	Security (Request Filtering, Windows Authentication)
o	Application Development (.NET Extensibility 3.5 and 4.6/4.8, ASP.NET 3.5 and 4.5/4.8, ISAPI Extensions, ISAPI Filters)
o	Management Tools (IIS Management Console) [1]
•	BITS (Background Intelligent Transfer Service): Enables reliable downloading of files.
•	Remote Differential Compression (RDC): Required for signature creation and package updates.
•	.NET Framework 4.8: Required for site servers, site systems, and the console (newer versions of Configuration Manager mandate .NET 4.8). [1]
•	Microsoft ODBC Driver for SQL Server: A required driver installed on the site server.
•	SQL Server (Separate or Local)
•	Windows Server Update Services (WSUS): To roll out Windows update across the environment.

Since that is now out of the way, we can run the setup. Upon running the installation setup wizard, we enter the FQDN of our MECM server, the folder paths pointing to the database, the site code we want to use, etc. Typically, MECM takes around 15-45 minutes to install.

<img width="741" height="566" alt="Domain35" src="https://github.com/user-attachments/assets/98e9eda6-0397-4643-bd59-df177b26a598" />
<img width="846" height="751" alt="Domain41" src="https://github.com/user-attachments/assets/3702296b-b26c-4203-8c75-e256561212d2" />
<img width="733" height="559" alt="Domain42" src="https://github.com/user-attachments/assets/0828737e-ba63-4eb9-9f0a-33b19a9c3394" />

Now that MECM is installed, we need to set up the environment. Let’s first get the current PCs in our environment communicating with the MECM server. To do this, we want to run a discovery. To run the discovery, we will select Administration  Hierarchy Configuration  Discovery Methods from the left pane and then select ‘Run Full Discovery Now.’ You can then specify what domains or OUs you want to discover devices from. We will select our danp.lab domain. This can take a varying amount of time to complete, depending on your environment. Once it is complete, your existing assets should be populated in the ‘Assets and Compliance’  ‘Devices’

<img width="1714" height="872" alt="Domain43" src="https://github.com/user-attachments/assets/bca78397-970b-4c76-b323-b020443fc7d5" />
<img width="509" height="526" alt="Domain44" src="https://github.com/user-attachments/assets/5c38f899-67bc-4218-be06-250bed27ed04" />
<img width="1036" height="656" alt="Domain45" src="https://github.com/user-attachments/assets/50798d65-c537-44f5-97dc-07071f4bf915" />
<img width="1002" height="474" alt="Domain46" src="https://github.com/user-attachments/assets/12fc7313-1fa8-47d0-aea7-6f77c39b54fe" />

Next, a boundary group will be created. This will allow us to install software from the MECM Server to the existing endpoints as well as imaging new endpoints. Navigate to Administration  Hierarchy Configuration  Boundary Groups to create a new boundary group. You will need to enter the name and the IP(s) of the subnet(s) in the domain that you want to use for this. 

<img width="632" height="580" alt="Domain50" src="https://github.com/user-attachments/assets/0718b1c9-0ff7-4ef9-bf96-28830bf5d59a" />

A boot image will also need to be created for new endpoints to boot to the image. We’ll go to Software Library  Operating Systems  Boot Images. The winpe.wim file will be copied into the MECM source content directory. To navigate to the boot image, the path is Software Library  Operating Systems  Boot Images  Add Boot Image. On the Data Source window, we will enter the pathway manually to the file we copied earlier. This will create the boot image. 

<img width="695" height="592" alt="Domain57" src="https://github.com/user-attachments/assets/0446ace2-4b0b-487b-a738-6c9214546099" />

Next an Operating System image must be declared. An ISO file will be needed and mounted. Within that ISO and inside of the Operating System Image Wizard, browse to the install.wim file. 

<img width="1113" height="588" alt="Domain59" src="https://github.com/user-attachments/assets/c499872d-c8d5-47b6-a78b-ff8aa2a8aa8e" />
<img width="689" height="603" alt="Domain60" src="https://github.com/user-attachments/assets/4720f534-79df-4a85-a607-4c71ef0784ba" />


Now that we have the boundary group, boot image, and operating system image created, it is time to create a task sequence. Here we are choosing to install an existing image package and referencing the Operating System Image we created earlier. Under Configure Network, we’ll select to join the domain of danp.lab for the task sequence. 

<img width="757" height="679" alt="Domain61" src="https://github.com/user-attachments/assets/4df48f77-6108-4edb-a398-e1e93e6ec1d2" />
<img width="753" height="678" alt="Domain62" src="https://github.com/user-attachments/assets/482e044b-13dd-4b58-bd02-69e066f937c9" />
<img width="756" height="677" alt="Domain63" src="https://github.com/user-attachments/assets/66c81ad0-d78f-4d48-a319-ce262c1f65a7" />
<img width="758" height="676" alt="Domain64" src="https://github.com/user-attachments/assets/2a785e6a-d376-45f2-a6b3-2d48d5fb5632" />


Once the Task Sequence is created, there is one more step left before we begin the imaging process. First, we need to create general virtual machines in VMWare. MECM needs a hardware MAC address and computer name to know what to image. We can locate the MAC address of the virtual computer by going to VMWare Workstation --> Settings --? Network Adapter --> Advanced. Once we have the MAC address of the VM, we can import the VM into MECM by going to Assets and Compliance --> Import Computer Information. It is here where we can put what we want the name of the computer to be and what the MAC address of the VM is. We can also import a file, which would allow us to import VMs into MECM in bulk. We will get to that later. For now, let's import the three VMs we created.

<img width="427" height="425" alt="Domain86" src="https://github.com/user-attachments/assets/72ff3c54-4cdb-4e56-aeb8-39c4b03a4c1d" />
<img width="663" height="138" alt="image" src="https://github.com/user-attachments/assets/0f65e2a9-f4b3-4d4f-9183-0f7830ac3cd7" />
<img width="693" height="600" alt="image" src="https://github.com/user-attachments/assets/e009f7a6-3394-4881-a244-1b917e83d8bc" />
<img width="746" height="570" alt="image" src="https://github.com/user-attachments/assets/c1adf30e-11b1-4260-a5ca-cf1e3b1fd343" />
<img width="692" height="596" alt="image" src="https://github.com/user-attachments/assets/57585db1-3888-4804-bd33-20200d8f664c" />

We should be all set! Now, we simply need to boot up one of our newly created VMs to firmware and choose to boot from PXE. Note that we want to make sure we allow PXE booting from our MECM.

<img width="770" height="655" alt="image" src="https://github.com/user-attachments/assets/920fbe83-fabc-48bb-a4f8-8a9224a6db92" />
<img width="673" height="628" alt="Domain71" src="https://github.com/user-attachments/assets/5a63ce76-319e-4d9a-acbc-b2778e07d697" />

From here, MECM should do the rest, and we should have three newly created and imaged VMs. 

<img width="1022" height="763" alt="Domain77" src="https://github.com/user-attachments/assets/77fcae46-3ec1-460e-b005-99da17763eea" />
<img width="1027" height="847" alt="Domain79" src="https://github.com/user-attachments/assets/dda5c344-928d-42af-bc99-d0a07ae2447a" />

Being that we are imaging a kiosk, we want it to behave as such. Hence, there are certain settings we need to configure. For one, we need to configure an autologin account. Since this is emulating what would be a publicly accessible device in an environment, we want to limit the privileges and lock this down as much as possible. We will configure group policy on the computer account and the user account to lock this kiosk down. First, let's go to the registry and navigate to the Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon hive. We will want to ensure the strings AutoAdminLogon and ForceAutoLogon are present and set to 1. If they are not present, we will need to create them. Two additional strings of DefaultUsername and DefaultPassword will need created as well. Their values will be set to the username and password of the autologin account. Note that this autologin account will also need to be created in Active Directory with the same username and password you've set in the registry, otherwise it won't work.

<img width="258" height="655" alt="image" src="https://github.com/user-attachments/assets/d613b72c-36a3-47d3-a677-f8cf975bc60e" />
<img width="401" height="546" alt="image" src="https://github.com/user-attachments/assets/698e100c-3cbe-4928-9aeb-fb7515ba6712" />

Now that we have our autologin account created, we can configure group policy settings on the kiosk. First thing is configuring the user account to automatically navigate to a site upon bootup. For test purposes, we'll just have it go to Google Chrome's home page. For this, we'll navigate to User Configuration --> Policies --> Administrative Templates --> System in the Group Policy Editor. For the interface file name, we will enter "C:\Program Files\Google\Chrome\Application\chrome.exe" --kiosk https://www.google.com. The first part navigates to and launches the browser. The --kiosk tells the Chrome browser to launch in full-screen kiosk mode. Finally, the last part is the URL of Chrome's home page. Naturally, we'll want to make sure this policy is enabled.

<img width="1468" height="528" alt="image" src="https://github.com/user-attachments/assets/56f66f28-94fe-41de-a55f-be237c59749e" />

Other policies for the kiosk include preventing users from using hidden shortcuts to break out of the kiosk environment or accessing task manager.
• Disable Task Manager:
	• Path: User Configuration --> Policies --> Administrative Templates --> System --> Ctrl+Alt+Del Options
	• Setting: Remove Task Manager --> Set to Enabled.
• Block Windows Registry Tools:
	• Path: User Configuration --> Policies --> Administrative Templates --> System
	• Setting: Prevent access to registry editing tools (regedit) --> Set to Enabled.
• Disable Command Prompt / PowerShell:
	• Path: User Configuration --> Policies > Administrative Templates --> System
	• Setting: Prevent access to the command prompt --> Set to Enabled.

3. File System & Storage Lockdown

Stop users from plugging in malicious USB drives or browsing the local C: drive folders to look at your MECM client configuration directory.
• Hide Local Drives:
	• Path: User Configuration --> Policies --> Administrative Templates --> Windows Components --> File Explorer
	• Setting: Hide these specified drives in My Computer -> Set to Enabled and choose Restrict A, B, C and D drives.
• Prevent USB / Removable Media Insertion:
	• Path: Computer Configuration --> Policies --> Administrative Templates --> System --> Removable Storage Access
	• Setting: All Removable Storage classes: Deny all access --> Set to Enabled.

4. Automated Maintenance & Power Properties

Ensure the kiosks never go to sleep, lock themselves automatically, or prompt users to install random Windows updates that interfere with MECM schedules.
• Disable Screen Locking:
	• Path: Computer Configuration > Policies --> Administrative Templates > Control Panel > Personalization
	• Setting: Do not display the lock screen --> Set to Enabled.
• Disable Power Sleep Timeouts:
	• Path: Computer Configuration > Policies --> Administrative Templates --> System --> Power Management --> Sleep Settings
	• Setting: Specify the system sleep timeout (plugged in) --> Set to Enabled and type 0 (Never sleep).

	Once this is all set, the computer should automatically log in to this account upon bootup, browse to Google Chrome's home page automatically, and display only the webpage so that the user cannot access the Start Menu or Task Manager.

	<img width="1021" height="762" alt="image" src="https://github.com/user-attachments/assets/c7a291ff-7e11-436a-b817-8023cdb48a74" />


Now we set all this at a local PC level, but in an enterprise environment, it would be much more efficient to set the group policy via the MECM server and have it automatically applied to devices in bulk. In this way, we could have several different group policies that are tied to specific groups and OUs in Active Directory and our MECM Task Sequences. In this manner, we would have full control over the security and privileges of our PCs and users, even in a large enterprise environment. To set this, we can create the group policy the same way we did on our local PC, but just on our domain server instead. We will declare the OUs, groups, and computers affected through the scope. 

<img width="870" height="490" alt="image" src="https://github.com/user-attachments/assets/7c66bcca-4b48-4279-b286-388d7afe0966" />

Through this, anything that is in the Kiosk OU or is a member of the Kiosk Lockdown Group will have this group policy applied to them. To ensure this is applied to a PC upon image, we can edit the task sequence in the MECM Server to include a gpupdate /force and a restart. So long as we add it to the OU that has the group policy we want, it will be applied upon imaging.

<img width="929" height="549" alt="image" src="https://github.com/user-attachments/assets/5a3f08f3-c285-474d-8f8e-4dcadcb3e943" />
<img width="933" height="543" alt="image" src="https://github.com/user-attachments/assets/2fec2849-5043-4f7d-8ee2-64e3140ca763" />

If we import computers using a file, we could have a file with thousands of PCs imported into MECM and then image them all with our various group policies declared per OU in Group Policy Editor. This would allow us to image and declare policy for our devices at scale.

<img width="929" height="421" alt="image" src="https://github.com/user-attachments/assets/f1aaeeac-cfd2-4b82-bf5d-a0ba288c4ac4" />

To do this, we must first create the file as a .csv. This can be through Excel or notepad. It is imperative to ensure it is in the correct format, or it won't be accepted by MECM.

<img width="233" height="114" alt="image" src="https://github.com/user-attachments/assets/b033d222-6d4d-4664-bbda-cbfbaa636cc4" />

Now that the file is created, we can go back to 'Import Computer Information' and this time select 'Import computers using a file.'

<img width="682" height="200" alt="image" src="https://github.com/user-attachments/assets/90479bbb-d2d5-456e-8363-6d8584d8e8a5" />

We want to ensure that we have ComputerName and MAC Address selected from the columns. From here, we simply import the file and viola, the computers are now added to MECM and we can image them through the MECM server with the group policy applied, provided we select the correct task sequence to place it into the correct OU upon imaging.

<img width="689" height="358" alt="image" src="https://github.com/user-attachments/assets/842ecf9a-b545-4921-b89d-0a90a29556e5" />
<img width="688" height="604" alt="image" src="https://github.com/user-attachments/assets/331f9010-ef5f-4bd1-9b40-65f5a932bbd7" />
<img width="694" height="605" alt="image" src="https://github.com/user-attachments/assets/75c13163-ccbc-48f5-a91c-0b19fce45cb2" />




<img width="238" height="63" alt="image" src="https://github.com/user-attachments/assets/0e42fb75-ccce-4b24-a440-793ee192d1bd" />
