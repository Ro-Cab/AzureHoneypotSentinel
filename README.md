# Azure Honeypot Monitoring w/ Sentinel


## 🧩 Step 1 - Create a Free Azure Account  
Started at the Azure Free Trial page and registered with a personal email. A credit card was required for verification, but received free credits upon setup. Signed into the Azure portal to begin configuration.

<img width="975" height="386" alt="image" src="https://github.com/user-attachments/assets/a96c569e-e785-4093-a376-57e70f438b7b" />

_Azure portal home page after signing up_

---

## 🏗️ Step 2 - Set Up Resource Group and Network  
In the Azure portal, searched for “Resource groups” and created one named RG-SOC-LAB2 in East US 2.  
Created a Virtual Network named VNET-SOC-LAB2 in the same resource group and region. Left subnet settings at their defaults to host the VM.

<img width="975" height="221" alt="image" src="https://github.com/user-attachments/assets/9ad1fe2a-95ff-48da-9bf8-3340ed2dc168" />

_Resource group and vnet created_

---

## 💻 Step 3 - Deploy the Honeypot VM  
Created a new Virtual Machine under RG-SOC-LAB2, named CORP-NET-EAST2.  
Chose Windows 10 as the image, set size to Standard B2s.  
Set the username to labuser and password to Password123!.  
For networking, chose VNET-SOC-LAB2, enabled Public IP, disabled boot diagnostics. Completed the deployment.

<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/055d7e4c-4845-4589-91a1-9017e4a72476" />

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

<img width="975" height="468" alt="image" src="https://github.com/user-attachments/assets/0477ab22-463d-4148-a28c-2dfb46423d9d" />

_NSG inbound rule settings with “Allow Any” rule._


---

## 🔑 Step 5 - Connect to the VM  
Opened Remote Desktop Connection (mstsc) with VM’s public IP, logged in using labuser and password.  
Inside the VM, ran wf.msc and disabled Windows Defender Firewall on Domain, Private, and Public profiles.

<img width="975" height="546" alt="image" src="https://github.com/user-attachments/assets/704008a0-e064-4585-ac8f-6614ce5fb3ac" />

_Windows Firewall settings turned off._

---

## 📡 Step 6 - Verify Internet Access  
From the local computer, opened command prompt and pinged the public IP of the VM (`ping <68.220.176.6>`).  
Received replies, confirming the VM is exposed and live.

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/f17d5216-9a3d-485b-b0d4-22327de33947" />

_Ping response confirming VM connectivity._

---

## 🚫 Step 7 - Create Failed Login Attempts  
Closed the VM session, attempted to log in with incorrect credentials to generate failed login attempt logs.

<img width="975" height="378" alt="image" src="https://github.com/user-attachments/assets/20d61f6b-12f3-4760-ac75-0d31c68761bb" />

_Attempting to RDP into VM with incorrect credentials_

---

## 🔍 Step 8 - Review Windows Security Logs  
Inside the VM, opened Event Viewer → Windows Logs → Security.  
Looked for Event ID 4625 (failed login attempts) for both test and future attacks.

<img width="975" height="639" alt="image" src="https://github.com/user-attachments/assets/77e63832-93f0-4749-a014-dec80963bbf2" />

_Event Viewer showing failed logins (Event ID 4625)._

---

## 📊 Step 9 - Create a Log Analytics Workspace  
Back in the Azure portal, created Log Analytics Workspace named law-soc-lab, used rg-soc-lab and East US 2 region.  
This workspace is used to store VM logs for analysis.

<img width="975" height="289" alt="image" src="https://github.com/user-attachments/assets/6afaf1fc-4381-484e-825d-18e76365b78b" />

_Log Analytics workspace summary page._

---

## 🧠 Step 10 - Deploy Microsoft Sentinel  
Searched for Microsoft Sentinel in Azure, created and linked it to the law-soc-lab workspace.  
Activated Sentinel to start collecting and analyzing lab data.

<img width="975" height="360" alt="image" src="https://github.com/user-attachments/assets/b4e89080-0678-4727-8a81-6fee7de1d348" />

_Sentinel setup linked to Log Analytics Workspace._

---

## 🔌 Step 11 - Connect the VM to Sentinel  
Inside Sentinel, opened Content Hub and installed “Windows Security Events via AMA.”  
Created a Data Collection Rule (DCR), named it DCR-Windows-Logs2, assigned it to rg-soc-lab, and selected the VM to collect all security events.

<img width="975" height="456" alt="image" src="https://github.com/user-attachments/assets/42347024-8777-48a2-8444-a0723f25526f" />

_Installed Windows Security Event_   <br>

<img width="975" height="864" alt="image" src="https://github.com/user-attachments/assets/b071926a-190f-43e1-981f-d8406f6e967b" />

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

<img width="975" height="456" alt="image" src="https://github.com/user-attachments/assets/63dd0bec-2852-46d9-8abd-467719d0ce27" />

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

<img width="975" height="420" alt="image" src="https://github.com/user-attachments/assets/6e9d76ed-f4a3-4a6e-b167-acef63c8e57e" />

_KQL results showing failed login attempts._

---

## 🌍 Step 14 - Import GeoIP Watchlist  
Downloaded the GeoIP summary CSV and created a watchlist named “geoip” in Sentinel. Configuration -> Watchlists  
Set the name and alias as geoip, uploaded the file, and set Network as the key column to enable IP-to-location mapping.

<img width="975" height="314" alt="image" src="https://github.com/user-attachments/assets/6ba7e906-b0a8-45ec-b0d0-6b8608416a10" />

_Watchlist upload complete with record count._

---

## 🗺️ Step 15 - Build the Global Attack Map  
Opened Sentinel Workbooks → Add Workbook.  
Removed default visuals, added a query, pasted the provided JSON code for the global attack map.  
Saved as “Windows VM Attack Map” to view live attack sources mapped geographically.
Live data was now being plotted by region as attackers attempted to connect to the honeypot. 

![Sentinel attack map showing live global attack data.](./media/image16.png)
