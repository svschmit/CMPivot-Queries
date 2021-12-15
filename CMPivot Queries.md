# Check WSUS Settings on clients
``` 
CcmLog('WUAHandler') | where LogText contains 'Group policy settings were overwritten by a higher authority'  
| project Device, LogText, DateTime  
``` 
