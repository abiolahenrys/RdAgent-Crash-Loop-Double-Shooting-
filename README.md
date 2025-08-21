# RdAgent Crash Loop Incident Report

## Author
Henry-Abiola Samuel Oludare  

## Date
21 August 2025  

## Server
SurveyServer – Windows Server

---

## 1. Background
The server experienced repeated crashes caused by the **Azure Guest Agent service (RdAgent)**.  
Each crash occurred when the RdAgent service attempted to start but terminated unexpectedly, creating a **continuous crash-restart loop** that destabilized the server.

---

## 2. Problem

### Symptoms
- Server instability and occasional full crash.  
- Event Log entries indicated repeated service failures:

The RdAgent service terminated unexpectedly. It has done this N time(s). The following corrective action will be taken in 15000 milliseconds: Restart the service.

- Attempts to stop the service (`net stop RdAgent`) or check running tasks (`tasklist`) showed **no active process**.

### Root Cause
- The RdAgent installation was **corrupted or outdated**, preventing successful startup.  
- Automatic restart attempts caused a **crash loop**, which destabilized the server.  
- Initial attempts in PowerShell to stop/disable the service failed due to alias conflicts (e.g., `sc` interpreted as `Set-Content`).

---

## 3. Analysis

- **Event Logs:** Multiple "terminated unexpectedly" entries were recorded in System logs.  
- **Service Control Manager:** Restart attempts occurred every 15 seconds.  
- **Service Configuration Check:**
```powershell
sc qc RdAgent
START_TYPE = AUTO
Binary path pointed to a corrupted installation.
Attempts to stop the service via PowerShell initially failed due to command alias conflicts.
```
## 4. Solution
The resolution involved disabling the crash loop, backing up the old agent, removing the corrupted agent, and reinstalling the latest stable version.
1- Disable the RdAgent crash loop
cmd /c "sc failure RdAgent reset= 0 actions= """ 
cmd /c "sc config RdAgent start= disabled"
net stop RdAgent

2-Backup existing agent
xcopy "C:\WindowsAzure\GuestAgent_*" "C:\Backup\RdAgent\" /s /e /h

3-Remove Corrupted Agent
msiexec /x "C:\WindowsAzure\GuestAgent_*\WaAppAgent.msi" /quiet

4-Install Latest Agent
msiexec /i "C:\Temp\WindowsAzureGuestAgent.msi" /quiet /norestart

5 Enable Start RdAgent
sc.exe config RdAgent start= auto
sc.exe start RdAgent
Get-Service RdAgent


