

### 🔍 Show all failed logins (KQL)
```kql
SecurityEvent
| where EventID == 4625

```
```
Explanation:

SecurityEvent → Table containing Windows security event logs.
where EventID == 4625 → Filters for Event ID 4625, which represents failed login attempts in Windows.
```


### ⏱️ Show recent failed attempts only (KQL)
```
SecurityEvent
| where EventID == 4625
| where TimeGenerated > ago(5m)
| project TimeGenerated, Account, IPAddress, Computer
```
```
Explanation:

where EventID == 4625 → Filters for failed login attempts.
where TimeGenerated > ago(5m) → Shows only events from the last 5 minutes.

project TimeGenerated, Account, IPAddress, Computer → Displays key details:

TimeGenerated → When the failed attempt occurred
Account → Username used in the attempt
IPAddress → Source IP of the login attempt
Computer → Target system that logged the event
```

