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












## Conclusion



## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>
