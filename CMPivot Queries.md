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
# Process
``` 
Process | where (CommandLine == '"C:\\Windows\\EXPLORER.exe"')
```
# DNS Server
```
OptionalFeature
| where Caption == 'DNS Server'    | where InstallState ==1
```
# Patches installed in last 90 days
```
QuickFixEngineering | where InstalledOn >= ago(90d)
```
# Windows 10 Sprache Lannguage
```
OperatingSystem | where OSLanguage == 1033
```
# Count of a software title by version
```
InstalledSoftware | summarize countif( (ProductName == 'title') ) by ProductVersion | where (countif_ > 0)|
render barchart
```
# M365 Apps Channels
```
OfficeProductInfo | summarize count() by Channel, ProductName | where channel != 'Unknown'
```
# Semi-Annual M365 Apps ProPlus Out of Support
```
Office365ProPlusConfigurations | where VersionToReport < '16.0.11328.20644' and cfgUpdateChannel == 'http://officecdn.microsoft.com/pr/7ffbc6bf-bc32-4f92-8982-f9dd17fd3114'
```
# Monthly  M365 Apps ProPlus Out of Support
```
Office365ProPlusConfigurations | where VersionToReport < '16.0.12827.20656' and cfgUpdateChannel == 'http://officecdn.microsoft.com/pr/55336b82-a18d-4dd6-b5f6-9e5095c314a6'
```
# All Devices with errors duringUpdate Scan
```
ccmlog ('UpdatesDeployment') | where (LogText like '%0x87d00215%') | distinct Device
```
# Java Version
```
InstalledSoftware
| where ProductName like 'Java%' and Publisher == 'Oracle Corporation'
```
# Last day of Boot
```
OperatingSystem  | summarize dcount(Device) by bin(LastBootUpTime,1d) | render barchart
```
# Local Policy Corruption
```
WinEvent('System', 1d)
| summarize dcount(Device) by ID, Device
| where ID == 1096
| join OS
| summarize count() by substring( strcat(Caption, ' ', Version), 10 )
| render barchart with (title='Local Policy Corrupt', xtitle='OS', ytitle='Number of Devices')
```
# Confilicting GPO
```
FileContent('C:\\Windows\CCM\Logs\PwrProvider.log') | where Content like '%Conflict with Group Policy%' | distinct Device
```
# Find all devices with old Intel WIFI driver
```
File('c:\windows\system32\drivers\netwtw06.sys')
| where LastWriteTime < ago(365d)
| project Device, FileName, LastWriteTime
```
# Clients with GUID 0000
```
FileContent('c:\Windows\smscfg.ini')
| where Content contains 'Previous SMSUID=GUID:00000000-0000-0000-0000-000000000000'
```
# Find Clients with GUID last change day today
``` 
FileContent('c:\Windows\smscfg.ini')
| where Content contains 'Last SMSUID Change Date=DD/MM/YYYY'
```
# Battery Status per User
```
Battery | project Device, EstimatedChargeRemaining, Status |join (Device | project UserName)
```
# Query IP Ranges Cisco Any Connect
```
IPConfig | where InterfaceDescription like 'Cisco AnyConnect Secure Mobility Client Virtual Miniport Adapter for Windows x64' | summarize count() by substring(IPV4Address, 0, 9) | render barchart with (title='AnyConnect Subnet Assignment', xtitle='Subnet', ytitle='Number of Devices')
```
# Query if IP is Assigned to Cisco Any Connect
```
NetworkAdapterConfiguration | where Description like 'Cisco AnyConnect Secure Mobility Client Virtual Miniport Adapter for Windows x64' | summarize count() by IPEnabled | render piechart with (title='AnyConnect Status')
```
# Check if Defender is running
```
Service | where (Name == 'windefend') | summarize State=countif(State == 'Running') by Device | summarize NumberOfDevices=count() by iif(State==1,'Running','Not Running') | render piechart with (title='Windows Defender Running')
```
# SMBv1 Status
```
SMBConfig
| summarize Enabled=countif(EnableSMB1Protocol == true) by Device
| summarize NumberOfDevices=count() by iif(Enabled==1,'Enabled','Disabled')
| render barchart with (title='SMBv1 Status', xtitle='Status', ytitle='Number of Devices')
```
# Flash Player running?
```
Process | where (Name == 'FlashUtil_ActiveX.exe')
```
# Is Powershell V2 installed?
```
OptionalFeature
| where (Name == 'MicrosoftWindowsPowerShellV2')
| where InstallState == 1
```
# Find Exchange Servers
```
InstalledSoftware
| where ProductName == 'Microsoft Exchange Server'
```
# Duplicate Execution request last 7 days
```
CcmLog('execmgr', 7d)
| where LogText like 'A duplicate execution request is found for program%'
| project Device, LogText, DateTime
```