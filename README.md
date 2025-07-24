# Active Directory Project - John Yang

## Objective

In this project, I built and deployed a security monitoring and response environment for Active Directory user account management. The project begins with network and infrastructure planning and is followed by deploying virtual machines in the cloud, configuring firewall rules, and establishing a Windows Active Directory domain with custom users and permissions. Logs from endpoints were forwarded to a Splunk instance, where they would go through an alert I configured to identify unauthorized login activity. To extend the workflow, I integrated Slack and Shuffle (SOAR) to build an automated response pipeline that alerts a team of analysts and allow them to disable compromised accounts directly through user prompts. This project reinforced my practical understanding of cloud infrastructure, identity management, log ingestion, SIEM tuning, and real-time security automation, essential domains for modern security operations.

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

![alt text](JaneDoeRDP.jpg)

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

![alt text](ScriptsLinux.jpg)

When you see this message, hold down space and press y to agree with the license.

![alt text](SplunkTerms.jpg)

Create the credentials for your Splunk administrator account.

Lets add a new firewall rule for the Ubuntu machine on Vultr.

![alt text](NewFirewallRule4Splunk.jpg)

Lastly, type in `ufw allow 8000` to allow inbound connections on port 8000.

Also do this for port 9997.

![alt text](AllowListeningSplunk.jpg)

Now we should be able to access the Splunk interface on a web browser.

Type in the IP address of your Ubuntu machine and then port 8000.

![alt text](SplunkOnBrowser.jpg)

On the Splunk interface, find the "Apps" tab on the top left corner, then press "Find More Apps".

Search for Windows and install "Splunk Add-on for Microsoft Windows".

![alt text](SplunkAddonForWin.jpg)

Now, head over to the settings tab and click on "Indexes".

Create a new index.

![alt text](NewIndexSplunk.jpg)

Hover back over to the settings tab and click on "Forwarding and receiving".

Click on "Configure receiving" then "New Receiving Port".

![alt text](ForwardingandReceiving.jpg)

Now we need to set up the Splunk Universal Forwarder on both our ADDC and test machine to forward telemetry to Splunk.

RDP into the test machine.

Going back to the Splunk website and press on "Products".

Press "Get My Free Download" under the Universal Forwarder.

![alt text](UniversalForwarder.jpg)

Download the 64 bit installation package for Windows.

We can copy the file to our RDP session.

![alt text](UniversalForwarderCopy.jpg)

Double click on the installation package. We will need to create a username and password.

Eventually we will reach the "Receiving Indexer" tab.

For this, type in the IP address for our Ubuntu machine that has Splunk installed on it.

![alt text](IndexSetup.jpg)

Then, head over to file explorer.

Navigate to C:\Program Files\SplunkUniversalForwarder\etc\system\default.

Copy "inputs.conf", navigate to \system\local, and paste the file in.

![alt text](inputsconf.jpg)

Open up a notepad with admin privilieges, and open the inputs.conf file within it.

At the end of the file, add these lines but with your index name instead on the second line.

![alt text](inputsconfedit.jpg)

Save the notepad.

Now, search up "Services", navigate to the "SplunkForwarder" service, and right click to open properties.

Head over to "Log On", and press "Local System Account".

![alt text](splunkservice.jpg)

After doing this, restart the service.

We are finished setting up the forwarder on our test machine.

Let's see if we are receiving information.

Head back over to the Splunk web interface.

Hover over "Apps" and click on "Search & Reporting".

Search for our index on the search bar.

![alt text](SplunkIndexSearch.jpg)

We can see that we are receiving information from our test machine.

Now, repeat the process we took to install the forwarder on the Domain Controller machine as well.

After installing the forwarder on the other machine, we can see that our search for the index is returning two host values.

This confirms that our two Windows machines are successfully forwarding data to Splunk.

![alt text](2hosts.jpg)

From here, lets modify our search to narrow down the events that we are concerned with.

Using the Windows event code for a successful login (4624) and the specific login types (7 for Unlock and 10 for RemoteInteractive - RDP)

![alt text](Search.jpg)

We can also further refine the search by looking for events where the source IP address is any IP address that is not a blank address or an address that starts with 40. (assuming this is our actual office network).

![alt text](ModifiedQuery.jpg)

We can also make the display look cleaner.

![alt text](SuccessfulDetection.jpg)

We can save this search as an alert now.

Make sure to also add "Add to Triggered Alerts" under the "When triggered" section.

![alt text](SaveAsAlert.jpg)

We can see that the alert has been caught and sent over to the "Triggered Alert" section.

![alt text](AlertCaught.jpg)

### Part 5: Connecting it all with Shuffle

Navigate to Shuffle's website, create an account, and create a workflow.

You should be greeted with this interface.

![alt text](Shuffle.jpg)

We are going to use a webhook to bring in data from Splunk to Shuffle.

It should be located under the "Triggers" section on the left bar of Shuffle.

![alt text](Webhook.jpg)

Copy the webhook URI, then navigate back to Splunk.

On the alert that we just created, add a webhook process under the "When Triggered" section.

Paste the Shuffle webhook URI we just copied.

![alt text](WebhookAlert.jpg)

Now, if our alert catches an event, our webhook should receive its information as well.

Lets see if this works. Some of the events are still being caught by the alert, because of the recent RDP logins.

![alt text](WebhookCaught.jpg)

Nice!

We see the full details of the event here.

![alt text](alertDetails.jpg)

Lets now implement the Slack part of our workflow.

On the search bar on the left side of Shuffle, search for Slack.

We will need to authenticate with our Slack account.

![alt text](AuthenticateSlack.jpg)

On the "Channel Value" section, copy and paste the specific channel ID that you want your message to be posted.

Now let's compose the message.

We can use specific values that have been emitted from our previous event that was caught by the webhook.

![alt text](SlackMessage.jpg)

We can reemit the event from the webhook and check if the message is being posted.

![alt text](SlackMessageSuccess.jpg)

Nice! We can now go on to the user input part of our workflow.

The User Action button should be located under the triggers section as well.

![alt text](UserInput.jpg)

Shuffle will send an email containing the user prompt, to which the SOC analyst can press on yes or no.

We can reemit the event again to see if this works.

![alt text](EmailSuccess.jpg)
































 
















































## Conclusion



## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>
