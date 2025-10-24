# SMB Cross-Platform Secure Share

A secure, cross-platform SMBv3 file sharing setup between macOS and Windows,  
featuring encrypted LAN transmission, fine-grained ACL permissions, and automated setup scripts.

---

##  Project Overview
This project explores how to build a **secure and high-performance file sharing channel** between macOS and Windows devices over a local network.

Instead of relying on third-party drives or cloud storage, this configuration establishes a **direct encrypted SMBv3 link** with:
- Static IP reservation (or IP Reserve if DHCP is enabled) + ARP binding
- Encrypted SMBv3 traffic over port 445
- Firewall whitelisting by device IP
- Independent NTFS permissions
- Cross-platform compatibility testing (Word for Mac)

---

##  Architecture

```
iMac (macOS)
    │  SMBv3 Encrypted Channel (Port 445)
Router (DHCP Reservation + ARP Binding)
    │
Windows 11 (SMB Server)
```

---

##  Key Features
-  Encrypted SMBv3 data channel
-  IP-level access control (DHCP + Firewall)
-  Separate user accounts (`smbshare`)
-  NTFS permission isolation (`User:(F)` / `smbshare:(M)`)
-  macOS Finder + Word compatibility
-  Troubleshooting log and automated scripts

---

##  Windows Setup
```powershell
net user smbshare StrongP@ssw0rd! /add
New-Item -Path "D:\Shared" -ItemType Directory -Force
icacls "D:\Shared" /inheritance:r /grant "User:(OI)(CI)F" "smbshare:(OI)(CI)M" /T
New-SmbShare -Name "Shared" -Path "D:\Shared" -FullAccess smbshare
Set-SmbServerConfiguration -EncryptData $true -Force
New-NetFirewallRule -DisplayName "Allow SMB from iMac" -Direction Inbound -LocalPort 445 -Protocol TCP -Action Allow -RemoteAddress 192.168.1.x
```

---

##  macOS Setup
```bash
sudo security delete-internet-password -s 192.168.1.10 /Users/$(whoami)/Library/Keychains/login.keychain-db
- # Optional: clear cached credentials in Keychain if reconnection fails

open "smb://192.168.1.10/Shared"
sudo xattr -d com.apple.quarantine /Volumes/Shared
```

---

##  Troubleshooting Highlights
- Word for Mac “no permission” issue → caused by sandbox restriction (may require full disk access)
- Fixed by using `/Volumes/Shared` path and granting full disk access
- Finder cache & Keychain cleanup required for reconnect (If path set error, reset is needed)

---

##  Security Notes
- Do **not** expose port 445 to the internet
- Use VPN for remote access
- SMBv3 encryption enabled (`Set-SmbServerConfiguration -EncryptData $true`)
- Isolated account (`smbshare`) for access control

---

##  Lessons Learned
- Permission inheritance can easily expand scope unintentionally
- Word for Mac uses temporary lock files that require delete permission
- macOS Finder caches SMB sessions, so credential cleanup is essential

---

##  Next Steps
- Add VPN tunnel (WireGuard)
- Automate scheduled SMB service monitoring
- Integrate centralized logging

---

## 📄 License
MIT License

- All information provided as is, without warranty.
