# Get All Admins on Client
```
Administrators
```

# Check WSUS Settings on clients
``` 
CcmLog('WUAHandler') | where LogText contains 'Group policy settings were overwritten by a higher authority'  
| project Device, LogText, DateTime  
``` 

# Installed Software by Publisher
``` 
InstalledSoftware | where Publisher like 'Microsoft%
``` 
# Client with no Boundary
```  
CcmLog('LocationServices') | where LogText contains 'Client is not in any boundary group.'
| project Device, LogText, DateTime
``` 
# Default Gateway
```
IPConfig
| where (Status=='UP')
|summarize count() by IPV4DefaultGateway |render barchart
```
# Applications crashing
```
AppCrash | summarize dcount( Device ) by FileName
```
# Service Stopped on a specified Device
```
Service
| where (State == 'Stopped')
| where (Device == 'Devicename')
``` 
# Recently used Apps
```
CCMRecentlyUsedApplications
| summarize dcount( device ) by ProductName
| render columnchart
```
# Missing software updates
```
SoftwareUpdate | where (Device == 'DeviceName')
```
# Find Clients Missing Updates
```
SoftwareUpdate    
| where KBArticleIDs == 'KB4565489'
```
# Boot Times 
```
SystemBootData
 | where Device == 'DeviceName'
 | project SystemStartTime, BootDuration, OSStart=EventLogStart, GPDuration, UpdateDuration
 | order by SystemStartTime desc
 | render barchart with (kind=stacked, title='Boot times for MyDevice', ytitle='Time (ms)')
```
# Average Boot Time
```
SystemBootData
 | summarize avg(BootDuration / 1000)
 by Device
 | render barchart with (kind=stacked, title='Average Boot Times', ytitle='Time (seconds)')
```
# Boot Time per OS
```
SystemBootData
| join OperatingSystem 
| summarize avg(BootDuration / 1000) by Caption
| render barchart with (kind=stacked, title='Average Boot Times', ytitle='Time (seconds)')
```
# Boot Times per Model
```
SystemBootData 
|join Device
| summarize avg(BootDuration / 1000) 
by Model
| render barchart with (kind=stacked, title='Average Boot Times', ytitle='Time (seconds)') 
```
# DNS Server Name
```
IPConfig | where (Status == 'Up') | summarize count() by DNSServerList | render barchart
```
# Operating System 
```
OperatingSystem | summarize count() by strcat(Caption, ' ', BuildNumber) | render piechart

Alternative:
OperatingSystem         
| where Caption like 'Microsoft Windows%'
| project Caption, Version=case(
BuildNumber == '10240', '(1507)'
, BuildNumber == '10586', '(1511)'
, BuildNumber == '14393', '(1607)'
, BuildNumber == '15063', '(1703)'
, BuildNumber == '16299', '(1709)'
, BuildNumber == '17134', '(1803)'
, BuildNumber == '17763', '(1809)'
, BuildNumber == '18362', '(1903)'
, BuildNumber == '18363', '(1909)'
, BuildNumber == '19041', '(2004)'
, ''
)
| order by Version
| summarize count() by substring( strcat(Caption, ' ', Version), 10)
| render barchart with (title='Windows Version Summarization', xtitle='Versions', ytitle='Number of Devices')
```
 
