# Azure Honeypot Monitoring w/ Sentinel

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
Opened Remote Desktop Connection (mstsc) with VM’s public IP, logged in using labuser and password.  
Inside the VM, ran wf.msc and disabled Windows Defender Firewall on Domain, Private, and Public profiles.

<img width="2726" height="1520" alt="image" src="https://github.com/user-attachments/assets/a8ab02db-0ffd-4cb2-a985-34b28b58f9bb" />

_Windows Firewall settings turned off._

---

## 📡 Step 6 - Verify Internet Access  
From the local computer, opened command prompt and pinged the public IP of the VM (`ping <68.220.176.6>`).  
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
Looked for Event ID 4625 (failed login attempts) for both test and future attacks.

<img width="2507" height="1632" alt="image" src="https://github.com/user-attachments/assets/25d057f6-166c-4f3c-8428-95cbc8b711c0" />

_Event Viewer showing failed logins (Event ID 4625)._

---

## 📊 Step 9 - Create a Log Analytics Workspace  
Back in the Azure portal, created Log Analytics Workspace named law-soc-lab, used rg-soc-lab and East US 2 region.  
This workspace is used to store VM logs for analysis.

<img width="2514" height="741" alt="image" src="https://github.com/user-attachments/assets/2bca84a3-bae9-485b-96cc-5f5e227ab79d" />

_Log Analytics workspace summary page._

---

## 🧠 Step 10 - Deploy Microsoft Sentinel  
Searched for Microsoft Sentinel in Azure, created and linked it to the law-soc-lab workspace.  
Activated Sentinel to start collecting and analyzing lab data.

<img width="2519" height="922" alt="image" src="https://github.com/user-attachments/assets/0007c1f3-6c9f-4713-80a4-ce7192f67d04" />

_Sentinel setup linked to Log Analytics Workspace._

---

## 🔌 Step 11 - Connect the VM to Sentinel  
Inside Sentinel, opened Content Hub and installed “Windows Security Events via AMA.”  
Created a Data Collection Rule (DCR), named it DCR-Windows-Logs2, assigned it to rg-soc-lab, and selected the VM to collect all security events.

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

<img width="2733" height="1160" alt="image" src="https://github.com/user-attachments/assets/79a5f900-418a-4edc-8c4b-84977f41f713" />

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

<img width="2844" height="907" alt="image" src="https://github.com/user-attachments/assets/fc6c894a-a3ec-4a2b-a384-12ad92151baa" />

_KQL results showing failed login attempts._

---

## 🌍 Step 14 - Import GeoIP Watchlist  
Downloaded the GeoIP summary CSV and created a watchlist named “geoip” in Sentinel. Configuration -> Watchlists  
Set the name and alias as geoip, uploaded the file, and set Network as the key column to enable IP-to-location mapping.

<img width="2846" height="1278" alt="image" src="https://github.com/user-attachments/assets/6b36eb40-cc55-4553-9fa1-aa6fc9ed7602" />

_Watchlist upload complete with record count._

---

## 🗺️ Step 15 - Build the Global Attack Map  
Opened Sentinel Workbooks → Add Workbook.  
Removed default visuals, added a query, pasted the provided JSON code for the global attack map.  
Saved as “Windows VM Attack Map” to view live attack sources mapped geographically.
Live data was now being plotted by region as attackers attempted to connect to the honeypot. 

![Uploading image.png…]()
<div/>
