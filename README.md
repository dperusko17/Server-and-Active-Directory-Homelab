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




