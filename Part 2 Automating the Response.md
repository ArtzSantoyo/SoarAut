Now lets Change a bit the previous workflow so the alerts sended to the analyst can execute and automatic action

Create and Add an Ubuntu Agent to Wazuh:
Install the Wazuh agent on your Ubuntu machine.
Configure the agent to communicate with your Wazuh manager.
Change the Workflow for Testing:

![1](https://github.com/user-attachments/assets/913265ee-d281-40b9-a522-60eddf609898)

Add Wazuh SIEM functionality.
Set an action to run a command on the agent list (using the Ubuntu agent).
Create an Active Response:
Navigate to the Wazuh manager’s OSSEC file.
Go to the active response section.
Create a new command called “firewalldrop on local.”
This command will drop any connection with a suspicious URL on the localhost that activated the alarm.
Testing:
For testing purposes, use the Google IP address (8.8.8.8).
Check connectivity with a ping.
![ping google](https://github.com/user-attachments/assets/0a1216df-aaa9-4c24-8f31-4270d2ea751a)

Verify the Active Response:
Go to /var/ossec/bin and list the files.
Locate the agent control file that saves the configured responses.
Check the file to see the available options.
Use -L to check the current active responses.
Apply the Rule:
Use the following command:
/var/ossec/bin/agent_control -b 8.8.8.8 -f firewalldrop -u ubuntu
![2](https://github.com/user-attachments/assets/26dcfd35-c334-444c-a334-23856f73c99a)

![3](https://github.com/user-attachments/assets/69e49e0f-ce22-4a70-8e5b-43588aefbbc0)

![4](https://github.com/user-attachments/assets/dfb9d86a-5834-4919-a223-4fd5debf39b0)


This will block connections to the suspicious URL (Google IP) on the Ubuntu agent.
Check the Ping to Google:

Confirm that the ping to Google has stopped.
![5](https://github.com/user-attachments/assets/e8929c5c-a004-4db3-bdda-ea2d77da155c)

Double-Check in Wazuh Logs:
Look in /var/ossec/logs/active-responses.log to verify the active response.


![6](https://github.com/user-attachments/assets/9969f97b-3ea2-463a-912a-85fd148de1ce)
