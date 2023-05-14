# Azure-Cloud-Detection-using-MS-Sentinel
This lab is a quick display of detection malicious tasks with the use of MS Sentinel in Azure cloud environment.
<h2>Introduction</h2>

This practice project is my first-hand experience with deploying and utilizing Azure resources along with Microsoft Defender for Cloud and Microsoft Sentinel.

It simulates a cloud environment to analyse security events and detect anomalies with the help of Kusto Query Language, analytical rules and MITRE ATT&CK® Matrix for Enterprise.

This document demonstrates through images and brief descriptions, steps undertaken to accomplish the listed learning objectives. 
<br />


<h2>Learning Objectives</h2>

- Configure and deploy Azure Resources such as Log Analytics Workspace, Virtual Machines, and Microsoft Sentinel
- Implement Network and Virtual Machine security best practices
- Utilize Data Connectors to bring data into Microsoft Sentinel for analysis
- Understand Windows Security Event logs
- Configure Windows Security Policies
- Utilize KQL to query logs
- Write Custom Analytic Rules to detect Microsoft Security Events
- Utilize MITRE ATT&CK to map adversary tactics, techniques, detection and mitigation procedures


<h2>Lab Topology</h2>
<img src="Snapshots/Topology.PNG"/>

<h2>Walkthrough</h2>

<h3>Part 1: Azure sign-up and resources deployed</h3>

 - Signed Up for a free trial of Microsoft Azure Pay-As-You-Go Subscription
 - Renamed default subscription name to (Sub-AZ-CD-P1)
 - Created and deployed Resource Group (RG-AZ-CD-P1), Virtual Machine Windows 10 Pro (VM-AZ-CD-P1) - Allowed all inbound Remote Desktop Protocol (RDP) connection
 - Noticed default deployment of Network Security Group (NSG), Virtual Network(VNet), Public IP, Network Interface Card (NIC) assigned to the Virtual Machine (VM)

<img src="Snapshots/Window sign up and resources.png" height="80%" width="80%" alt="Disk Sanitization Steps" />
<h3>Part 2: Microsoft Defender for Cloud’s Just-In-Time (JIT) VM Access</h3>

 - In the VM > Network tab, besides the default inbound rules, the allowed RDP (default port 3389) inbound connection during VM configuration is dangerous as it is susceptible to brute-force or password spray attack
 - Thus, I locked down the RDP connection via JIT Access in Microsoft Defender for Cloud by enabling the enhanced security plan
 - Microsoft Defender for Cloud > Environment Settings > Enable All > Save

<img src="Snapshots/JIT not conf on VM.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Next, selected VM > Connect > Enable JIT Access, to enable JIT Access to the VM and block any inbound RDP traffic to it
 - Resulting in VM > Networking > JIT Priority (Any-VM = Deny)
 - To connect to the VM from only my PC, I requested access via VM > Connect > MyIP (auto-detected) > Request Access
 
<img src="Snapshots/JIT conf on VM.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/JIT enabled with MY IP option.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Resulting in approved access, VM > Networking > JIT Priority (MyIP-VM = Allow)
 - The access was granted as I am the default Global Admin for the Azure account
 
 
 <h3>Part 3: Log Analytics Workspace (LAW) and Microsoft Sentinel</h3>
 
 - Created LAW (LAW-AZ-CD-P1) to record all logs coming from the VM
 - Created Microsoft Sentinel and connect to LAW for real-time monitoring and analytics of security events

 - Microsoft Sentinel > Data Connectors > Windows Events via AMA > Data collection rule > RG-AZ-CD-P1 > All Security Events > Collect = Status Connected
 - The Data Connector helped collect all security events from the VM to stream it on Microsoft Sentinel
 
 <img src="Snapshots/Win data connector in sen.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 <h3>Part 4: RDP and Event Viewer</h3>
 
 - Here, I remoted into the VM through my PC and viewed the VM’s Event Viewer to inspect the security logs’ properties
 - VM > Connect > MyIP
 - VM > Copy Public VM IP
 - PC > Remote Desktop Connection > Paste VM Public IP > Connect > Azure Admin Credentials > Yes > Connected
 - VM home page > Event Viewer > Windows Logs > Security > Event ID
 
 
 <h3>Part 5: Microsoft Sentinel’s in-built analytical rule template</h3>
 
 - I realised that if there were many data connectors connected to Microsoft Sentinel, it would be easier to view the log data in one place, and quickly query them using KQL in a customizable format
 - Microsoft Sentinel > Logs > KQL (Event ID 4624 + Project)
 
<img src="Snapshots/KQL query built-in rule.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
 
 - In order to only be alerted of certain events that might be meaningful to the environment, I made use of the pre-packaged Analytical Rule Template to generate an alert for investigating potential incidents and determine it to be either a true positive or false positive 
 <img src="Snapshots/FP or TP.webp" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Microsoft Sentinel > Configure Analytics > Rule Templates > Excessive Windows Logon Failure > Create Rule
 - Also, chose the tactic based on MITRE ATT&CK framework (Credential Access) and Severity (Low)
 
<img src="Snapshots/KQL query built-in rule analytics.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="Snapshots/Rule for excessive login enabled.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 <h3>Part 6: Local Security Policy, Scheduled Task and Event Viewer</h3>
 
 - First, I configured the Other Object Access Event in the Local Security Policy of the VM as it is not logged by default in the Event Viewer
 - Audit Other Object Access Events - This policy setting allows you to audit events generated by the management of task scheduler jobs
 - VM Windows Start > Local Security Policy > Advanced Audit Policy Configuration > Object Access > Audit Other Object Access Events > Configure both Success+Failure > OK
 
 <img src="Snapshots/Local security policy audit policy.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Next, I created a new task to trigger on schedule which launched the Internet Explorer browser in action every time the task runs
 - VM home page > Task Scheduler > Create Task (Trigger+Action) = Open Internet Explorer browser
 
<img src="Snapshots/Task scheduler fr 1 st time.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/Task scheduler fr 1 st time with action.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
 
 - The Event Viewer immediately lists the new scheduled task with Event ID 4698 and the update to the same with Event ID 4702 
 - VM home page > Event Viewer > Event ID 4698 (Other Object Access Events)
 
 <img src="Snapshots/after task sch event ID changed to 4702.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<h3>Part 7: Microsoft Sentinel’s custom analytical rule template</h3>

 - The same events can be found in Microsoft Sentinel’s log tab, be queried by KQL to analyse event data for investigation, and project only the most relevent columns
 - Microsoft Sentinel > Logs > KQL (Event ID 4698 + event data + project)
 
 <img src="Snapshots/P7 1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - In addition, organized the event data with KQL to yield relevant information 
 - This KQL query was later fed into the upcoming custom analytical rule for investigation

 <img src="Snapshots/P7 2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - I wanted Microsoft Sentinel to alert me of such a potential incident directly, so I configured a custom analytical rule template to do the needful
 - The rule states that an alert will fire when a scheduled task is created in a VM
 - With MITRE ATT&CK framework, I mapped the tactic to be Persistence and technique to be T1053-Scheduled Task/Job

<img src="Snapshots/Mitre Attack Schedule Task.webp" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/P7 3.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Microsoft Sentinel > Configure Analytics > Custom Rule Template > Sen-analytics-custom-rule-AZ-CD-P1 > Create Rule
 - The Entity Mapping options were selected based on the essential information that will be helpful to jumpstart the investigation process. Also, the query schedule was set to run every 5 minutes

<img src="Snapshots/P7 4.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

 - Again, I created a new task scheduled to run and launch the Internet Explorer browser
 - VM home page > Task Scheduler > Create Task (Trigger+Action) = Open Internet Explorer browser
 - As expected, the event was reported in the Event Viewer
 
 <img src="Snapshots/P7 5 task scheduler 2nd time.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
 <img src="Snapshots/P7 6 task scheduler 2nd time in event log.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<h3>Part 8: Active Incident, Investigation and Remediation</h3>

 - More interestingly, the custom analytical rule worked as an Active Incident was recorded in Microsoft Sentinel
 - Microsoft Sentinel > Incidents > Investigate Details > Resolve+Close

<img src="Snapshots/P7 6 task scheduler 2nd time in sen.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/Sen incident detected.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/P8 1 Sen incident detected.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/P8 2 sen more detailed abotu incdent.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/P8 3 inevstigate sen incident.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<img src="Snapshots/P8 4 close+review incident in sen.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<h3>Part 9: Conclusions</h3>

 - The scheduled task, in this case, was non-malicious as it only opens a benign Internet Explorer. Thus, the incident is deemed false positive and the case is closed
 - Alternatively if it was malicious, an automated Playbook that uses Azure Logic Apps could be added to the custom rule in Microsoft Sentinel and help with an appropriate response to such incidents
 
<h3>Part 10: MITRE ATT&CK’s Detection and Mitigation</h3>

 - Referencing the MITRE ATT&CK framework Detection indicators, the Event ID 4698 should be monitored for scheduled tasks that facilitate initial or recurring potentially malicious code execution
 - Also, the suggested Mitigation recommended, that User Account Privileges should be limited to only authorise admins to create scheduled tasks on remote systems
 
 <img src="Snapshots/Mitre Attack.webp" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<h2>References</h2>

 - Incident Response process from identification to remediation
 - Log Analytics Workspace (LAW)
 - Azure Sentinel 
 - Just-in-Time (JIT) Access to secure VM management ports
 - Local Security Policy
 - Schedule Task/Job to trigger malicious applications
 - MITRE ATT&CK® Matrix for Enterprise
 
 <h1>-THE END-</h1>
