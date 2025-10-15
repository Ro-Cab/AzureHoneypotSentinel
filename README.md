# 💻 Azure Honeypot Monitoring w/ Sentinel

<h2>📘 Description</h2>

This project demonstrates how to build a cloud-based security monitoring environment using Microsoft Azure and Microsoft Sentinel. It guides you through deploying a single Windows honeypot virtual machine, exposing it to real-world Internet traffic, collecting attacker logs, and visualizing intrusion attempts in a live attack map dashboard. While not a full SOC, it simulates real security monitoring and log analysis workflows, making it ideal for learning SIEM fundamentals, Azure security, and Kusto Query Language (KQL).

<h2>🖼️ Overview</h2>
    
First, a Windows 10 virtual machine was deployed inside an Azure Resource Group and connected to a Virtual Network (VNet). Its Network Security Group (NSG) was left open to the public Internet, allowing real attackers and bots to attempt connections. <br>
The VM’s security logs were collected through a Data Collection Rule (DCR) using the Azure Monitor Agent (AMA) and sent to an Azure Log Analytics Workspace.  
This workspace was linked to Microsoft Sentinel, where I used Kusto Query Language (KQL) to analyze failed login events and enrich them with GeoIP data. <br>
Finally, the results were visualized in a custom Sentinel workbook displaying a global attack map, showing where intrusion attempts originated in real time.

<div align="center">
    
<img width="1813" height="864" alt="image" src="https://github.com/user-attachments/assets/dad9a899-b416-472d-8510-bb4a88b7b225" />
    
 _The diagram shows how the Azure Sentinel Honeypot Lab was structured._  
    
</div>
    
<h2>:toolbox: Languages and Utilities Used </h2>

- <b>Kusto Query Language (KQL) </b>         - Querying and filtering security logs in Sentinel
- <b>Microsoft Azure </b>                    - Primary cloud platform
- <b>Microsoft Sentinel</b>                  - SIEM for log collection and visualization
- <b>Azure Log Analytics Workspace</b>       - Centralized log repository
- <b>Azure Monitor Agent (AMA)</b>           - Forwards security logs from the VM to Sentinel
- <b>Windows Event Viewer</b>                - Inspecting local security events

<div align="center">
    
[GeoIP csv](https://raw.githubusercontent.com/joshmadakor1/lognpacific-public/refs/heads/main/misc/geoip-summarized.csv) <br>
[MAP.JSON](https://drive.google.com/file/d/1ErlVEK5cQjpGyOcu4T02xYy7F31dWuir/view)

</div>

<h2>:globe_with_meridians: Environments Used </h2>

- <b>Windows 10 (Azure VM)</b>          - Honeypot host
- <b>Windows (Local)</b>                - For management and remote access
- <b>Azure Portal</b>                   - Central management interface

<h2>🧭 Program walk-through:</h2>

<div align="center">
   
## 🧩 Step 1 - Create a Free Azure Account  
Started at the Azure Free Trial page and registered with a personal email. A credit card was required for verification, but received free credits upon setup. Signed into the Azure portal to begin configuration.

<img width="2949" height="1123" alt="image" src="https://github.com/user-attachments/assets/0a8eee80-f4e8-4142-8597-65d571785cd2" />

_Azure portal home page after signing up_

---

## 🏗️ Step 2 - Set Up Resource Group and Network  
In the Azure portal, searched for “Resource groups” and created one named RG-SOC-LAB2 in East US 2.  
Created a Virtual Network named VNET-SOC-LAB2 in the same resource group and region. Left subnet settings at their defaults to host the VM.

<img width="2950" height="652" alt="image" src="https://github.com/user-attachments/assets/7b77f47f-3972-440e-8102-040f2ecde12d" />

_Resource group and vnet created_

---

## 💻 Step 3 - Deploy the Honeypot VM  
Created a new Virtual Machine under RG-SOC-LAB2, named CORP-NET-EAST2.  
Chose Windows 10 as the image, set size to Standard B2s.  
Set the username to _labuser_ and password to _Password123!_.  
For networking, chose VNET-SOC-LAB2, enabled Public IP, disabled boot diagnostics. Completed the deployment.

<img width="2549" height="1136" alt="image" src="https://github.com/user-attachments/assets/f7a46dce-13a3-46ff-a84b-6fb92d3c8693" />

_Newly created VM overview_

---

## 🔥 Step 4 - Open the Firewall (Expose VM)  
Navigated to Network Security Group settings for the VM, deleted default inbound rules. Added a new inbound rule with:  
- Source: Any  
- Destination: Any  
- Port: *  
- Protocol: Any  
- Action: Allow  
- Priority: 100  
- Name: DangerAllowAll
   
_This exposed the VM as a honeypot to the public internet._  <br>

<img width="2827" height="1353" alt="image" src="https://github.com/user-attachments/assets/0b790e7c-d0dc-459c-ac1d-d37b9eb6449d" />

_NSG inbound rule settings with “Allow Any” rule._


---

## 🔑 Step 5 - Connect to the VM  
Opened Remote Desktop Connection with VM’s public IP, logged in using labuser and password.  
Inside the VM, ran wf.msc and disabled Windows Defender Firewall on Domain, Private, and Public profiles.

<img width="2726" height="1520" alt="image" src="https://github.com/user-attachments/assets/a8ab02db-0ffd-4cb2-a985-34b28b58f9bb" />

_Windows Firewall settings turned off._

---

## 📡 Step 6 - Verify Internet Access  
From the local computer, opened command prompt and pinged the public IP of the VM (_ping <68.220.176.6>_).  
Received replies, confirming the VM is exposed and live.

<img width="2503" height="1197" alt="image" src="https://github.com/user-attachments/assets/825bf75c-5113-4645-83ba-cad94926ac8e" />

_Ping response confirming VM connectivity._

---

## 🚫 Step 7 - Create Failed Login Attempts  
Closed the VM session, attempted to log in with incorrect credentials to generate failed login attempt logs.

<img width="2498" height="1009" alt="image" src="https://github.com/user-attachments/assets/ed672090-2ea0-427c-bb75-7075a3a44d94" />

_Attempting to RDP into VM with incorrect credentials_

---

## 🔍 Step 8 - Review Windows Security Logs  
Inside the VM, opened Event Viewer → Windows Logs → Security.  
Looked for Event ID 4625 (failed login attempts) to observe failed attempts.

<img width="2507" height="1632" alt="image" src="https://github.com/user-attachments/assets/25d057f6-166c-4f3c-8428-95cbc8b711c0" />

_Event Viewer showing failed logins (Event ID 4625)._

---

## 📊 Step 9 - Create a Log Analytics Workspace  
Back in the Azure portal, created Log Analytics Workspace named LAW-SOC-LAB2, used RG-SOC-LAB2 and East US 2 region.  
This workspace is used to store VM logs for analysis.

<img width="2514" height="741" alt="image" src="https://github.com/user-attachments/assets/2bca84a3-bae9-485b-96cc-5f5e227ab79d" />

_Log Analytics workspace summary page._

---

## 🧠 Step 10 - Deploy Microsoft Sentinel  
Searched for Microsoft Sentinel in Azure, created and linked it to the LAW-SOC-LAB2 workspace.  
Activated Sentinel (SIEM) to start collecting and analyzing lab data.

<img width="2519" height="922" alt="image" src="https://github.com/user-attachments/assets/0007c1f3-6c9f-4713-80a4-ce7192f67d04" />

_Sentinel setup linked to Log Analytics Workspace._

---

## 🔌 Step 11 - Connect the VM to Sentinel  
Inside Sentinel, opened Content Hub and installed “Windows Security Events via AMA.”  
Created a Data Collection Rule (DCR), named it DCR-Windows-Logs2, assigned it to RG-SOC-LAB2, and selected the VM to collect all security events.

<img width="2621" height="1234" alt="image" src="https://github.com/user-attachments/assets/30733e2f-f2fd-43dd-987f-24526a2b909b" />

_Installed Windows Security Event_   <br>

<img width="2139" height="1569" alt="image" src="https://github.com/user-attachments/assets/1cf0d6ba-8c09-4d68-8db8-0a6cad1ec630" />

_DCR creation screen with selected VM._

---

## 🕒 Step 12 - Wait for Logs  
Waited 20–40 minutes for logs to ingest.  
In Log Analytics → Logs, ran the query: <br>

SecurityEvent
<br>

_(This query retrieves Windows Security Log data from the Log Analytics workspace)_
<br>
Results appeared, which confimred my VM’s logs were being forwarded successfully.

<img width="2824" height="1288" alt="image" src="https://github.com/user-attachments/assets/14265f83-55a9-4d00-95c9-aac421d550c3" />

_SecurityEvent query showing incoming data._


--- 


## 🔍 Step 13 - Analyze Logs with KQL  
Used following queries in the Log Analytics workspace:

**Show all failed logins:**  <br>
SecurityEvent <br>
| where EventID == 4625


**Show recent failed attempts only**:
<br>
SecurityEvent
<br>| where EventID == 4625
<br>| where TimeGenerated > ago(5m)
<br>| project TimeGenerated, Account, IPAddress, Computer


_These help you track attacker activity in real time._

<img width="2617" height="1123" alt="image" src="https://github.com/user-attachments/assets/d15f02c1-fa3d-4bde-b50f-a27b791e5556" />

_KQL results showing failed login attempts._

---

## 🌍 Step 14 - Import GeoIP Watchlist  
Downloaded the GeoIP summary CSV and created a watchlist named “geoip” in Sentinel. Configuration -> Watchlists  
Set the name and alias as geoip, uploaded the file, and set Network as the key column to enable IP-to-location mapping.

<img width="2513" height="804" alt="image" src="https://github.com/user-attachments/assets/d03ef110-b463-43b1-9693-691f0966de48" />

_Watchlist upload complete with record count._

---

## 🗺️ Step 15 - Build the Global Attack Map  
Opened Sentinel Workbooks → Add Workbook.  
Removed default visuals, added a query, pasted the provided JSON code for the global attack map.  
Saved as “Windows VM Attack Map” to view live attack sources mapped geographically.
Live data was now being plotted by region as attackers attempted to connect to the honeypot. 

<img width="2846" height="1278" alt="image" src="https://github.com/user-attachments/assets/6b36eb40-cc55-4553-9fa1-aa6fc9ed7602" />

_Sentinel attack map showing live global attack data._

<div/>
