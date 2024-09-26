# Soar Automation
Lets Create a SOAR With Wazuh and TheHive
I will be using an Ubuntu VM in the clouds (DigitalOcean) and a Windows 10 VM with Sysmon installed
The logical WorkFlow will be this:
![SOAR Diagram](https://github.com/user-attachments/assets/9b9b642f-1367-4626-b844-23687093e4a8)

On the DigitalOcean homepage, I will create 2 VMs for my servers: one will be Wazuh and the other will be TheHive, both with Ubuntu 20.04 LTS,  I'm gonna Create a firewall so only my local IP can have Access to the VMs and therefore I can work in a more secure lab, the rules will be to only  for inbound traffic: allow TCP, UDP and SSH connections from my local IP address, and leave the outbound rules as they are by default: and then add my VMs to the firewall and get them both protected.
![add firewall](https://github.com/user-attachments/assets/143c0fae-8c51-4288-836c-be5e7e082a2d)
![firwall](https://github.com/user-attachments/assets/46886af3-0d82-43d3-bb5a-3328819c940e)


First lets configure the components of this SOC environment

Cassandra Configuration:

Edit the Cassandra configuration file (/etc/cassandra/cassandra.yaml).
Customize the cluster name: I set it to “MyHive”).
![1](https://github.com/user-attachments/assets/b667fed7-93b4-432f-b91a-b0f0f508b2d7)

Update the Listen_address to the IP address of your TheHive VM.
Modify the SeedProvider to use the same IP address
Change RCP Address to the VM IP and change rcp_keepalive: true
![2](https://github.com/user-attachments/assets/a05bcb75-3d84-467d-9e3a-147c723e115a)
![4](https://github.com/user-attachments/assets/32d9a290-776e-4905-8f3a-b0419ff0084b)

Stop and restart the Cassandra service to apply the changes.
![5](https://github.com/user-attachments/assets/16ddbdf4-6a1f-43b5-8700-1a5c855640c2)

Elasticsearch Configuration:

Edit the Elasticsearch configuration file (/etc/elasticsearch/elasticsearch.yml).
Change the cluster name to “MyHive.” and uncomment it, same with node name
![6](https://github.com/user-attachments/assets/0e37be44-73b1-42ec-9657-045735ace911)

Set the network_host to the VM’s IP address.
Uncomment the cluster initial master node configuration (remove node 2 if not needed, unless I need to upscale).

![7](https://github.com/user-attachments/assets/af4f8f0c-e752-49de-b65f-8b723dcfe61a)


Start and enable the Elasticsearch service.
Verify its status using systemctl status elasticsearch.
![8](https://github.com/user-attachments/assets/811e4802-5420-4741-b812-eaf01139130b)

TheHive Configuration:

Edit TheHive’s application configuration (/etc/thehive/application.conf).
Update the hostname to the public IP of your TheHive VM.
Set the cluster name to “MyHive.”
![10](https://github.com/user-attachments/assets/521c53b1-2472-4398-93a9-898b82ecf89f)

Scroll down to the Service configuration section and adjust application.BaseUrl to the VM’s public IP.
Save the changes.
Start and enable TheHive.
![11](https://github.com/user-attachments/assets/88f6470f-df37-4040-8225-286d59fefac6)

Now some of the files needed for the hive to work properly need to be available to users and the group but by default they are ROOT permission only so now I need to change it.
![9](https://github.com/user-attachments/assets/c41b806a-4802-434c-a6b5-4dd5e1ff21ec)

Access TheHive dashboard by navigating to the VM IP on port 9000(AKA IP:9000)
![entering thehive](https://github.com/user-attachments/assets/f3e6600b-6b29-4004-a04d-c1fee040fcbe)
The Default credentials are: username "admin@thehive.local" password :"secret"
![inside the hive](https://github.com/user-attachments/assets/387a719a-a07e-4a87-8051-103fc1ec97ef)


Adding Wazuh Agents:

Add a new agent in Wazuh for your Windows machine with Sysmon.
Use the Wazuh server’s public IP as the server address.
![addagent](https://github.com/user-attachments/assets/a32d4003-6d3f-4eaf-90e5-f7a543e746a2)

Assign an agent name (e.g., “WinVM”).
![12](https://github.com/user-attachments/assets/3f6e3b29-2ad4-4c85-9a47-b755bc5869ce)

Follow the instructions to install Wazuh on the Windows machine using PowerShell as an administrator.
Confirm that the agent is initiated by going to windows services and look for Wazuh
![13](https://github.com/user-attachments/assets/b23cbe86-49b6-47f7-af6a-cb4d852c1e73)
![13primo](https://github.com/user-attachments/assets/40b1f615-b74e-4f7b-a8d5-7d4f17a349cf)
Now I go to the Wazuh Dashboard and I can see that my agent is being shown

![14](https://github.com/user-attachments/assets/ccd00a1d-5408-42ec-a552-d5ebda2c3123)



Configuring Sysmon Data Ingestion:

Open the Windows Event Viewer and navigate to Applications and Services Logs > Microsoft > Windows > Sysmon > Operational.
Copy the full name of the Sysmon log.
![sysmonoperational](https://github.com/user-attachments/assets/7507c235-53cd-4cde-ab61-6a6c999e13a0)

Locate the Wazuh installation folder (usually C:\Program Files (x86)\ossec-agent) and open the ossec.conf file with Notepad.
![15](https://github.com/user-attachments/assets/f2993a77-a4dc-47eb-89a3-22de88d7d3eb)

In the log analysis section for applications, add a copy of the existing configuration and include the Sysmon operational log name.
![16](https://github.com/user-attachments/assets/438430f4-1669-4be4-8a2a-b5aa755014fe)

Restart the Wazuh service.

Generating Data for Testing:

Visit the Wazuh dashboard and explore the Security Events section.
To generate data, Im gonna use Mimikatz. Before downloading it, ensure that your browser settings allow downloads (go to Windows security/Virus and Threat protection/exclusions and add the downloads folder the in my browser: chrome in this case, I go to settings, privacy and security and then select no protection, so now we can download mimikatz in the downloads folder without problems).

To download Mimikatz: (https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919)

Now with mimikazts downloaded, extract the zip file, lets go to the folder where it is contanined, copy the address and open a Powershell As admin, and change the direction with the "cd" command to where mimikatz is, run mimikatzs (command: .\mimikatz.exe)
![19](https://github.com/user-attachments/assets/bb36dc8a-dead-45ba-9f9f-bd5ff938d7c1)

Then I check to Wazuh Dashboard/Events and search for mimikatz, but no events yet (possibly a rule or alert was not trigger) so i need to configure wazuh to log more or create a rule
![20](https://github.com/user-attachments/assets/43327af7-1140-4b99-ae7c-0ec3591daed8)

So now i need to modify the ossec configuration to log everythings, but before that is a good practice to create a backup and then i can modify without worries
![21](https://github.com/user-attachments/assets/a596a8f4-620c-42c6-a3f3-24cdf4c23e69)

Now I want Wazuh to start to log everything, so get inside the file with nano
And in "logall" and "logall_json" change it from "no" to "yes" and after that restart the wazuh manager and now wazuh will archive all logs in a file called "archive"
![22](https://github.com/user-attachments/assets/7cda74ec-8681-4443-a15e-91c9097e9f1b)

We can check that by going to the /var/ossec/logs/archives 
![24](https://github.com/user-attachments/assets/027a9b27-c5af-47b0-abd1-aaca6251ec51)

Now wazuh in order to start digesting this logs is neccesary to change the configuration in filebeats: nano /etc/filebeat/filebeat.yml and change "archive enable" to true, and as always that a a configuration is changed, the service need to restart
![25](https://github.com/user-attachments/assets/a2203d5f-2603-4ef8-9005-e58727a18e95)
![26](https://github.com/user-attachments/assets/cd681eb2-f837-421e-886e-78e3d6a4ba43)


Now with the new configuration lets check the wazuh dashboard and create a need index
In wazuh/menu/stack management/index patterns and create a new index pattern that i will name "wazuh-archives-*" (the asterick is to include everything) then next in time field "timestamp" and create index pattern
![27](https://github.com/user-attachments/assets/7570ef63-a092-4f68-b077-b27fb92832c5)


Is possible to check is there is data ingested in the archives so im gonna test with mimikatz with: nano archives.json | grep -i mimikatz and I can see there is data, so I know data is being generated and collected by wazuh, sometimes it takes a bit of time to show off in the dashboard
![28](https://github.com/user-attachments/assets/006ad63b-3209-442d-bca7-c97603adfca1)

next to go wazuh/menu/discover and select 

