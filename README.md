# Soar Automation
Lets Create a SOAR With Wazuh and TheHive
I will be using an Ubuntu VM in the clouds (DigitalOcean) and a Windows 10 VM with Sysmon installed
The logical WorkFlow will be this:
![SOAR Diagram](https://github.com/user-attachments/assets/9b9b642f-1367-4626-b844-23687093e4a8)

First lets configure the components of this SOC environment

Cassandra Configuration:
Edit the Cassandra configuration file (/etc/cassandra/cassandra.yaml).
Customize the cluster name: I set it to “MyHive”).
Update the Listen_address to the IP address of your TheHive VM.
Modify the SeedProvider to use the same IP address.
Stop and restart the Cassandra service to apply the changes.
Elasticsearch Configuration:
Edit the Elasticsearch configuration file (/etc/elasticsearch/elasticsearch.yml).
Change the cluster name to “MyHive.”
Set the network_host to the VM’s IP address.
Uncomment the cluster initial master node configuration (remove node 2 if not needed, unless I need to upscale).


Start and enable the Elasticsearch service.
Verify its status using systemctl status elasticsearch.
TheHive Configuration:
Edit TheHive’s application configuration (/etc/thehive/application.conf).
Update the hostname to the public IP of your TheHive VM.
Set the cluster name to “MyHive.”
Scroll down to the Service configuration section and adjust application.BaseUrl to the VM’s public IP.
Save the changes.
Start and enable TheHive.
Access TheHive dashboard by navigating to the VM IP on port 9000.
Adding Wazuh Agents:
Add a new agent in Wazuh for your Windows machine with Sysmon.
Use the Wazuh server’s public IP as the server address.
Assign an agent name (e.g., “WinVM”).
Follow the instructions to install Wazuh on the Windows machine using PowerShell as an administrator.
Confirm that the agent is initiated.
Configuring Sysmon Data Ingestion:
Open the Windows Event Viewer and navigate to Applications and Services Logs > Microsoft > Windows > Sysmon > Operational.
Copy the full name of the Sysmon log.
Locate the Wazuh installation folder (usually C:\Program Files (x86)\ossec-agent) and open the ossec.conf file with Notepad.
In the log analysis section for applications, add a copy of the existing configuration and include the Sysmon operational log name.
Restart the Wazuh service.
Generating Data for Testing:
Visit the Wazuh dashboard and explore the Security Events section.
To generate data, Im gonna use Mimikatz. Before downloading it, ensure that your browser settings allow downloads (go to Windows security/Virus and Threat protection/exclusions and add the downloads folder the in my browser: chrome in this case, I go to settings, privacy and security and then select no protection, so now we can download mimikatz in the downloads folder without problems).
To download Mimikatz: (https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919)
