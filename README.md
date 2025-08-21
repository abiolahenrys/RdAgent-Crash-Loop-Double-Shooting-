# Azure Guest Agent (RdAgent) Crash Loop Troubleshooting

This document outlines the troubleshooting steps and resolution for a recurring **Azure Guest Agent (RdAgent)** crash loop that caused server instability.  

---

## Background  
The server experienced repeated crashes caused by the **Azure Guest Agent service (RdAgent)**.  
Each crash was triggered when the RdAgent service attempted to start but failed, leading to a continuous **crash–restart loop**.  

---

##  Problem  

### Symptoms  
- Server instability and occasional full crash.  
- Event Log entries showing repeated failures:  

## Log output
"The RdAgent service terminated unexpectedly. It has done this N time(s).
The following corrective action will be taken in 15000 milliseconds: Restart the service."

- Attempts to stop the service using `net stop RdAgent` or `tasklist` showed no active process.  

### Root Cause  
- The **RdAgent installation was corrupted or outdated**, causing the service to fail at startup.  
- Windows attempted to restart the service every 15 seconds, leading to instability.  
- Initial PowerShell attempts to stop/disable the service failed due to alias conflicts (e.g., `sc` interpreted as `Set-Content`).  

---

##  Analysis  

1. **Checked Event Logs**  
 - Multiple `terminated unexpectedly` entries under **System logs**.  
 - Service Control Manager showed restart attempts every 15 seconds.  

2. **Verified Service Configuration**  
 ```powershell
 sc qc RdAgent

START_TYPE was set to AUTO.
Binary path pointed to a corrupted installation.
--
