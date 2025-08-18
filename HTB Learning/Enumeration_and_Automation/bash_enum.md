## Bash Enumeration Script

Collect system, user, and file information quickly.

```bash
#!/bin/bash
echo "[+] Current user: $(whoami)"
echo "[+] Hostname: $(hostname)"
echo "[+] Kernel: $(uname -a)"
echo "[+] Network Information:"
ifconfig || ip a
echo "[+] Running processes:"
ps aux
echo "[+] SUID binaries:"
find / -perm -4000 2>/dev/null
```
