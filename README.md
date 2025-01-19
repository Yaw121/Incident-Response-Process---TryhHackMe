### Incident-Response-Process---TryhHackMe

<details><summary><b>Preparation</b></summary>

Preparation in an incident response plan involves establishing policies, assembling a trained response team, and ensuring access to tools and resources. It includes creating playbooks, conducting training and simulations, and maintaining documentation like network diagrams and contact lists. This phase ensures the organization is ready to detect, respond to, and recover from incidents effectively.
</details>

<details>
  <summary><b>Scenario</b></summary>

  In our scenario, we are acting as members of our Incident Response Team. A member of the organisation's SOC Team has called us to investigate and remedy a potential incident impacting a Windows workstation.

This is how the SOC Team has engaged us:

The user contacted the IT Team, reporting that his laptop started acting up and became extremely slow, to the point that he was having trouble working. The user couldn't pinpoint exactly what he was doing when the computer suddenly slowed down. He was browsing the web and working on some documents, as usual. He tried rebooting the machine, but performance was still very low.

IT has checked the machine's resources and found that the CPU usage is unusually high, even after closing all running apps. Suspecting a potential incident, IT has escalated the ticket to the SOC Team.

The SOC Team has verified that no alert was raised on the SIEM or EDR platforms for the workstation. The only anomaly that we have identified is some outbound connections on the perimeter firewall originating from the workstation's IP. The connections occur every second, and all have the same destination IP. The connections are not blocked by the FW. We have gone back to the user, who doesn't acknowledge these connection attempts.

Escalating to the IR Team.
</details>

<details>
  <summary><b> Detection and Analysis</b></summary>


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

### Analysing the Macro

To investigate further, we can view the list of macros in the document by navigating to View > Macros. In the newly opened window, we confirm the presence of a macro. Selecting the macro and clicking the Edit button allows us to examine its contents.

![image](https://github.com/user-attachments/assets/6d1db1d0-7ef1-4c99-b4da-e80c448b8959)
</details>



<details>
  <summary><b>Containment, Eradication, and Recovery</b></summary>

### Containment

What we can do now on the machine is kill the process to stop it from further “stealing” its resources. In the Task Manager, we can right-click on the process > select End task.

![image](https://github.com/user-attachments/assets/1778dec9-2a98-421c-8e5a-742a2262cc2d)


Now is the time to create a list of the IoCs that were gathered during our study and take appropriate action by searching the entire organization using every tool available to us (SIEM, EDR, network devices, etc.) for any instances of IoCs. The following IoCs have been gathered in our scenario and need to be addressed:
The IP and port of the C2 server (as already mentioned in the previous task).
The URL from which the macro-enabled Word document was downloaded.
The URL embedded in the macro from which the malware was downloaded.
The hash of the malware’s executable.

We can find and fix any further infected hosts within the company by searching the network for these IoCs.
</details>


<details><summary><b> Eradication and Recovery</b></summary>

We will need to remove any artifacts that were dropped on the machine in order to completely remove the danger.

To begin, we can remove the malware from the temporary folder in which it was operating. Next, we have to remove the Word document from the download folder that contained the macro that downloaded the malware automatically. In order to stop the user from accidentally clicking on the downloading link again, we must lastly delete the browser's download history.

Above all, we need to make sure that no persistence mechanisms remain in operation. The Run registry key must therefore be set back to its initial value. We can accomplish this by typing regedit into the Windows search box and choosing the Registry Editor application.
To view the compromised Run key, we can paste the full path of the key in the bar at the top of the editor: Computer\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
We will see a list of all the applications and processes configured to run at logon automatically. To delete the persistence, we can select the value containing our miner and simply right-click > select Delete.

With these actions, assuming that the malware analysis of the executable didn't discover any other persistence mechanisms or artefacts dropped by the malware, the machine is restored to its clean state.
</details>


<details><summary><b>Lessons Learned</b></summary>

  In this final steps of the IRP, we will have to document any lessons learned to prepare for any future incident. Here are the lessons learned;

- Implementing an EDR solution able to detect the kind of threat that we just faced (crypto miners and malicious macros).
- Enforcing a web-browsing control system that would prevent users from navigating to unsafe websites.
- Raising awareness among employees on the potential threat of macro-enabled Office files and navigating suspicious links, for example, with mandatory training on the topic.
- Discussing the approach of implementing a policy to block the execution of macros as a countermeasure, ensuring that this wouldn't disrupt legitimate business operations
  
</details>



