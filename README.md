# SOC Automation
Resources used: One Ubuntu server with Wazuh, one Ubuntu server with TheHive, a Windows 10 VM (virtualbox) and Shuffle


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
And in "logall" and "logall_json" change it from "no" to "yes" and after that restart the wazuh manager and now wazuh will archive all logs in a file called "archive" (This way regardless of a rule being triggered or not it will be archived and therefore allows me to search for it)
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

Next, go to wazuh/menu/discover and select Wazuh-archives-* as the filter. Now we can see it on the dashboard. I will scroll down until I see OriginalFileName to craft the alert, because regardless of whether an attacker changes the name, it will still be detected.
![aparece mimi](https://github.com/user-attachments/assets/d25e994f-358c-46d8-8503-12592ebe9ace)
![29](https://github.com/user-attachments/assets/1092e509-fc7d-473b-9ed7-d538922d7730)
Now, let's create a rule using the dashboard: home/management/administration/rules. Then, manage the rule files
![30](https://github.com/user-attachments/assets/49c68721-0cf7-4538-8134-66f58b259d3d)

Search for Sysmon, and below, I can see sysmon_id_01 (which means a process is being generated). Then, click the I icon; these are Sysmon rules built into Wazuh (specifically for targeting event ID 1). I copy one to use as a reference for creating a custom rule to detect Mimikatz. After that, I go to Custom Rules, click on the local rules icon to start creating a rule, paste the previous Sysmon rule, and let's start editing
![31](https://github.com/user-attachments/assets/fdb518e4-6e58-4377-8751-2dd512c24981)

First, the Rule ID should always start with 10.000. The one above is 10.001, so I cannot use that. Instead, I’ll use 10.002. For the level, I’m going to use 15 (just for fun). The field name, as mentioned before, will be OriginalFileName, and I’ll add Mimikatz. In the options, I will erase it because I want the full log. The description will be 'Alert: Mimikatz Detected', and in the ID, I’ll set it to 03 because it indicates credential dumping (which is what Mimikatz does)

![32](https://github.com/user-attachments/assets/6b5a0a89-6b7a-441f-bba5-092ad352341a)

Save it and wazuh is gonna need to restart

Now, to test it, I’m going to change the name of mimikatz.exe to HarmlessFile and execute it. Then, I’ll check if it appears in my security events.

![33](https://github.com/user-attachments/assets/7f0a00a9-f833-4820-b596-e9639de3cb7a)

And there it is, with the Rule ID, description, and everything else configured as before. It’s notable that even if the image of the file changes to the original filename, it is still detected as expected

![34](https://github.com/user-attachments/assets/7aea83ba-6af0-4868-be4d-4b7b2324112a)

![35](https://github.com/user-attachments/assets/0df823a5-6ad0-4fc0-8c3a-e37436450929)

<b> Now Integrating With Shuffle</b>

Heading to the Shuffler homepage, I click on Workflows and select New Workflow, which I will call 'SOC Automation', and then create it

![36](https://github.com/user-attachments/assets/6f8d0556-b1bf-4c36-b7ae-1f8563462719)

Now, click on Triggers, drag a Webhook icon, and change the name to wazuh-alerts. Then, copy the webhook URI, which will be used shortly
Next, click on the 'Change me' icon and set it to 'Repeat back to me.' Then, in the call section, click the plus icon and select 'Execute Argument

Now I need to ingrate Shuffle to my Wazuh Manager by adding and integration tag in the ossec file, therefore I need to to go the ossec.conf file
and add an integration tag(source:https://wazuh.com/blog/integrating-wazuh-with-shuffle/)this way wazuh knows that i wanna connect to shuffle, that will be the next:

Name the Shuffler, and for the hook URL, use the URL from the webhook URI. Set the Rule ID to match the same ID for Mimikatz in Wazuh. Then, save it and restart the Wazuh manager. After that, check the manager status

![37](https://github.com/user-attachments/assets/2f23797a-0e05-4e5a-855f-d2190ba98ccf)
![38](https://github.com/user-attachments/assets/5d17fd64-6c1a-4e79-871e-871fed2ab6ed)

Click on Run inside the Shuffler webhook, and then run Mimikatz. The argument will be executed And the Results of the run are:
![39](https://github.com/user-attachments/assets/9aeab9b8-2a90-443d-98a5-b583a1b1c36f)

<b>Add more Functions</b>

As implemented, Shuffler will receive the alerts. The next step will be to create a more complete workflow:

Mimikatz alert sent to Shuffler.
Shuffler receives the Mimikatz alert by extracting the SHA256 hash from the file.
Check that hash with VirusTotal.
Send details to TheHive to create an alert.
Send an email to the SOC analyst to begin the investigation.
Therefore, I go to the ChangeMe icon and modify its action to 'Regex.' I will set the action to 'Regex Capture Group,' and the input data will be 'Hashes.' Now, at the bottom, it needs a regex pattern. To make things easy, I’ll copy the hashes from the previous execution and ask ChatGPT to parse a SHA256 value for those hashes.
![40](https://github.com/user-attachments/assets/f3bd0404-00ea-4fbf-bd07-bfbe552d8d6d)
Now I rerun the workflow in Shuffler, and I can see that it’s correctly providing the SHA256 hash
![41](https://github.com/user-attachments/assets/9eb291a1-9d81-4c5d-9abe-e03306cd137f)

Next, I modify 'Change Me' to 'SHA256_Regex.' The next step is to use VirusTotal with its API, so I go to VirusTotal and copy my account API key. In Shuffler, I search for the VirusTotal app for my workflow, add it to SHA256_Regex, and then set it up. I find the action to Get a Hash Report and set the ID to SHA256_Regex to group the list, then rerun the workflow.

In the results from VirusTotal, scrolling down to Last_analysis_stats, I can see that 67 scanners detected this file as malicious
![42](https://github.com/user-attachments/assets/d1c0cf1e-24cb-44d0-bf3a-993380b4ca50)
![43](https://github.com/user-attachments/assets/cbbad875-705d-4c06-a9c2-93f6515b4621)
![44](https://github.com/user-attachments/assets/ac7c7ac2-d2ef-49f6-8377-fe46a0ff38df)

Lets add TheHive to this workflow, look for in shuffle apps and drag it to the workflow
![45](https://github.com/user-attachments/assets/557d7000-da67-4649-8d8e-3ccd342d1bde)

Now is time to go back to TheHive Dashboard, Log in and create a new organization, the new organization has no users to lets create them
![46](https://github.com/user-attachments/assets/9acd81e5-8b91-48d1-a003-4b883d6ea0ce)

"The first account is a normal analyst profile, and the second one will be a service type, also with an analyst profile. After creating both accounts, I set a password, and for the SOAR or service account, I generate an API key to authenticate with Shuffler. Then, I log out and log in with one of my analyst accounts, and now the dashboard looks different. Next, I go back to Shuffler and authenticate TheHive using the API and the IP address of TheHive VM

![46](https://github.com/user-attachments/assets/c06e031f-69ee-491d-97f5-898626c371f5)
![47](https://github.com/user-attachments/assets/d9d573c9-2f9a-4456-a3f9-cb88e02f7f1f)
![48](https://github.com/user-attachments/assets/120dd8e2-4c3c-45c6-862e-9be4f7272130)

After that, I select my Hive app in the workflow and edit it. find the action to create an alert and customize the body of the alert to my liking. For the URL, I use the IP address of my TheHive VM on port 9000. Therefore I need to modify my firewall to allow inbound traffic on port 9000, as this is where the TheHive instance will run, and I need to test this automation. So, I head to the firewall and create a custom rule to allow all TCP IPv4 traffic on port 9000

![49](https://github.com/user-attachments/assets/8f2a8dca-ef8d-4b7a-9e65-6acb4af6730c)

![50](https://github.com/user-attachments/assets/22f4fce5-2a83-445e-890b-bff86971c9ec)
Then Re-run the workflow in shuffle and I can see TheHive executing properly and generating alerts on the dashboard

![51](https://github.com/user-attachments/assets/c1d1c7d0-7101-4f32-8dd6-6970625bf21a)

![52](https://github.com/user-attachments/assets/6239ad02-c78a-4835-80a2-27fc677bd812)
![53](https://github.com/user-attachments/assets/e7284114-160e-48d4-a92b-19053fe51a9f)

<b> Sending Reports to the Analyst</b>

Now, I drag the email app into the workflow and connect it to VirusTotal. To customize the email, I set the recipients' addresses, the subject, and the body of the email, then run the workflow. I receive the notification at the provided email

![54](https://github.com/user-attachments/assets/c0235ef8-7d01-49a9-90e4-b0f1316ea0f5)

![55](https://github.com/user-attachments/assets/f9f96187-3984-4d26-8aee-77ed7dcd5b56)

Now with everything set up lets review the workflow on shuffler

![first part workflow](https://github.com/user-attachments/assets/4ce57e75-c7ed-45ad-a935-862ac489283f)

In the next Part, I will Set up this configuration to Apply Responses from the Analyst.










