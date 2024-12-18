# Incident-Response-Process---TryhHackMe

Let's open the Windows Task Manager and look at the Processes tab. There are many ways to open the Task Manager, the most straightforward being to right-click on the Windows toolbar > select Task Manager from the menu.

![image](https://github.com/user-attachments/assets/bc06f45a-35a3-4b67-8716-767882c6cc22)


We can see no application is currently running but we know from the ticket that, the user's system is running slow so we can click on "more details" in Task Manager to see the CPU usage.


![image](https://github.com/user-attachments/assets/2e09fa07-a8f2-4c6c-a617-bb07a8fd832a)

From the above screenshot, we can identify the first anomaly which seems like there's an application using most of the processing power so we need to find out what application it is. It looks like a Cyrpto miner but we need to make sure and tackle it.


To verify our suspicion, we continue our analysis by examining the properties of the suspicious process. This can be done by right-clicking on the process and selecting Properties from the context menu.

![image](https://github.com/user-attachments/assets/8a84c383-9207-4df9-a77c-afae6f739c19)


In the General tab, we find another red flag: the executable's location is in a temporary folder, a common indicator of malware. To confirm our suspicion, we can check for outbound connections by locating the process's PID. Right-click on the process, select Go to details, and note the highlighted row in the Details tab for the process information.


![image](https://github.com/user-attachments/assets/436da90f-9cd8-4454-879c-d32b3dbdb586)


To proceed, we open a Command Prompt by searching for cmd in the Windows search bar and launching the Command Prompt application. Using the PID obtained from Task Manager's Details tab, we run the following command:
netstat -aofn | find "{PID}".

![image](https://github.com/user-attachments/assets/25265b22-466b-4507-acd4-a26b1debce4f)


Based on the above screenshot from the command prompt, we can tell there's an outbound connection attempt towards a suspicious combination of IP and random destination port. This could mean the malware is trying to contact the Command and Control (C2) server to deliver 
mining data if its indeed a crypto miner.
The Ip and ports indicate a compromise in the system so we should perform various actions to Immediately block the IP with a rule on the organisation's front-end firewall to prevent communication with the C2, Search for the IP in the organisation's network traffic logs to hunt for other occurrences of the malware within the organisation's infrastructure, Feed the IoC to a monitoring rule on the SIEM to proactively detect any later infection from the same malware.

We need to find out if the user had downloaded any malicious attachments from a phishing email. So we go to  Downloads in the user's browser.

![image](https://github.com/user-attachments/assets/8e2ef3cb-e41d-4aa7-95ab-c2d0fa8beb2e)


Here, we can see the user had downloaded some weirdly named files, and the link looks suspicious because it's unusual for a legitimate link to point to an IP address instead of a domain name. Moreover, the file has a very suspicious extension: DOCM indicates that the file is a Macro-enabled Word Document, which means that it most likely contains macros
We can open the link and investigate more about the downloaded file. 

![image](https://github.com/user-attachments/assets/c5e1e5da-76e1-442f-890f-806ccec7b5cc)


The document raises suspicions as it contains a link to a non-existent webpage and, despite its extension suggesting the presence of macros, Microsoft Word does not issue any warnings. This likely indicates that any embedded macro is being executed automatically without user intervention.

To investigate further, we can view the list of macros in the document by navigating to View > Macros. In the newly opened window, we confirm the presence of a macro. Selecting the macro and clicking the Edit button allows us to examine its contents.

![image](https://github.com/user-attachments/assets/6d1db1d0-7ef1-4c99-b4da-e80c448b8959)



