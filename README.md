# Active Directory Project - John Yang

## Objective

This project involved building and deploying a security monitoring and response environment for Active Directory user account management. The project begins with network and infrastructure planning and is followed by deploying virtual machines in the cloud, configuring firewall rules, and establishing a Windows Active Directory domain with custom users and permissions. Logs from endpoints were forwarded to a Splunk instance, where they would go through an alert I configured to identify unauthorized login activity. To extend the workflow, I integrated Slack and Shuffle (SOAR) to build an automated response pipeline that alerts a team of analysts and allow them to disable compromised accounts directly through user prompts. This project reinforced my practical understanding of cloud infrastructure, identity management, log ingestion, SIEM tuning, and real-time security automation, essential domains for modern security operations.

### Skills Learned

- Developed cloud infrastructure skills by deploying secure virtual environments and configuring access controls.
- Experience with Active Directory configuration and domain-based identity management.
- Improved capabilities in log ingestion, parsing, and alert tuning within a SIEM.
- Integration of collaboration/communication platforms into incident response workflows through SOAR applications.
- Applied threat detection and response concepts in a simulated SOC environment.

### Tools Used

- Vultr (Cloud Hosting) for deploying virtual machines and simulating a production environment.
- Windows Server & Active Directory for domain services, identity management, and user authentication.
- Splunk (SIEM) for collecting, indexing, and analyzing security logs from endpoint systems.
- Shuffle (SOAR) for building automated incident response workflows.
- Slack for real-time notifications.
- Webhooks & APIs for integrating alerting and response actions across platforms.

## Steps

### Part 1: Making the Playbook

Before getting started with the project, it was important to get a sense of the workflow that I wanted to make. This was essential for displaying the relationship and the interactions between different parts of the project.

![alt text](Playbook.jpg)

We will first deploy three virtual machines through Vultr, two of which will be Windows, and the other one being Linux on Ubuntu. For the Windows machines, one will be the Active Directory Domain Controller, and the second one will be a test machine that we will be detecting unauthorized logins from. We will install Splunk on the Ubuntu machine. Telemetry from the Windows machines will be sent to Splunk, which then will forward that information to Shuffle. Shuffle will automatically message a Slack channel and send an email that contains a user prompt that asks if the receipient (SOC analyst team) wants to disable that user account or not. If yes is chosen, Shuffle will disable the user account.

### Part 2: Setting Up the Virtual Machines

Navigate to Vultr, or any cloud provider of your choice. 

Deploy three virtual machines, two Windows 10 machines and one Ubuntu.

Lets title them accordingly, one for the Active Directory Domain Controller, one for the test machine, and one for Splunk.

![alt text](3server.jpg)

After we have set up the machines on the cloud, we need to configure firewall rules so that we can access our machines from our physical device.

![alt text](FirewallRules.jpg)

Make sure to accept inbound traffic from your IP address on ports 22 and 3389 for SSH and RDP.

After this, we need to enable VPC in all of our machines so that we can set up a private network for our cloud machines.

This is how it looks for one of the machines, the ADDC.

![alt text](ADDCVPC.jpg)

Great! We are now finished settiung up our virtual environment. To check that this works, we can access our machines through SSH or RDP and check our IP addresses on the command line.

### Part 3: Configuring Active Directory

To get started here, lets RDP into our ADDC machine.

![alt text](RDPin.jpg)

On the server manager that we should see when we first log into the Windows machine, press "add roles and features".

On the "features" tab, check Active Directory Domain Services.

Finally, press "add features".

![alt text](InstallAD.jpg)

After pressing next a few times, we should see the install button. Click on the install button, and it should be completed.

![alt text](InstallADComplete.jpg)

On the top of the server manager, there should be a yellow warning sign. 

If we press on the warning sign, we should see "Promote this server to a domain controller".

Press "Add a new forest".

![alt text](DomainController.jpg)

Name the domain, then make a password.

When we get to the end of the process, press install.

![alt text](ActiveDirectoryDomainServices.jpg)

Now, our machine should have Active Directory installed and should be a Domain Controller.

To confirm this, lets search for Active Directory in the search bar.

![alt text](ConfirmADInstall.jpg)

If we see it here, nice! It has been installed.

Now, lets press on "Active Directory Users and Computers".

Open up our new forest and go to the users section.

![alt text](ADUsers.jpg)

From here, lets create a new user named Jane Doe. 

![alt text](TestUser.jpg)

We are finished with the ADDC, now let's RDP into the Test Machine to join the new domain and authenticate using our new user account.

When we get in the Test Machine, search up "This PC" and right click Properties.

![alt text](RenameThisPC.jpg)

Then, click on "change".

![alt text](Change.jpg)

Then, press on "Member of Domain" button and type in the domain name that we have created.

![alt text](ChangeDomain.jpg)

Then, we need to login with our domain administrator account.

If this fails, it means that there is a DNS issue.

To fix this problem, first navigate to change adapter options.

Then, press on "Ethernet Instance 0 2", "Internet Protocol Version 4", and finally, type in the VPC address we got for the ADDC in "Preferred DNS address".

![alt text](TestMachineIPConfiguration.jpg)

After doing this, the domain registration should work.

![alt text](WeareIntheAD.jpg)

To see that the test machine is on the domain, we can open up the console on Vultr.

![alt text](SuccessfulADRegister.jpg)

We see that when logging into "Other user", we are signing into our domain.

However, a problem arises when we try to RDP in the test machine with our user account.

![alt text](RDPFail.jpg)

Lets fix this.

Going back to our console for the test machine, search up "Remote" and press "Allow remote connections to this computer".

![alt text](AllowRDPTest.jpg)

After navigating to the page, press on "Show settings" for the first option under Remote Desktop.

![alt text](ChangeRDPSetting.jpg)

Log in with the administrator account.

![alt text](TypeinADPassword.jpg)

After logging in, we should see this page. Press "Select Users".

![alt text](ConfigureRDPAllow.jpg)

Then press "Add".

Now we can add our user account from our domain.

![alt text](JaneDoeAllowRDP.jpg)

Now, we should be able to RDP into the test machine with our user account.

### Part 4: Setting up Splunk

Lets open up a terminal on a device and SSH into our Ubuntu server.

![alt text](SSHinUbuntu.jpg)

Navigate to Splunk and create an account.

Hover over the "products" bar and press on "Free Trials & Downloads".

Click on "Get My Free Trial" on "Splunk Enterprise".

![alt text](SplunkEnterprise.jpg)

Click the "Copy wget link" on the .deb file.

![alt text](wgetlinkSplunk.jpg)

Go back to our SSH session, paste the link we just copied, and press enter.

Splunk should now be downloaded.

Then, run `dpkg -i splunk` and tab so that the Splunk autocompletes to the download.

![alt text](SplunkInstall.jpg)

Splunk should now be installed.

Run `cd /opt/splunk/bin`, then run `./splunk start`.

When you see this message, hold down space and press y to agree with the license.

![alt text](SplunkTerms.jpg)

Create the credentials for your Splunk administrator account.

Lastly, type in `ufw allow 8000` to allow inbound connections on port 8000.

![alt text](AllowListeningSplunk.jpg)

Now we should be able to access the Splunk interface on a web browser.

Type in the IP address of your Ubuntu machine and then port 8000.

![alt text](SplunkOnBrowser.jpg)


































## Conclusion



## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>
