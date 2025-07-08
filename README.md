# Dell XPS to Lenovo Cloned OS Migration
This guide documents the **full migration process** of a Windows 11 system from an old Dell laptop to a new Lenovo laptop.
# 🧭 Windows 11 Migration: Dell to Lenovo (Macrium Reflect + Rescue Disk)

This guide documents the **full migration process** of a Windows 11 system from an old Dell laptop to a new Lenovo laptop using Macrium ReflectX and Rufus.
It includes BIOS steps, driver fixes, PIN recovery, and repair steps — ideal for techs, sysadmins, or power users doing a full image restoration **across hardware**.

## Why/My Motivation
The reason for doing this was mostly laziness. I made the terrible assumption that in the modern world migrating a full clone OS to another machine should be easy.
I ran into problems every step of the way. If I can save you some time, I'm glad to help. However be careful there are some big downsides to trying this:

1) This process is fraught with potential for failure, from corrupted Windows due to mismatched hardware, to requiring a new license key (again from the mismatched hardware), to corrupted registry and other settings.
2) The easier and safer course is backing up your data (cloud or file backup software), and manually restoring the apps you want. I attempted this solution just to do it. It won't save you time.

---

## 🗂️ Overview
| Step | Description |
|------|-------------|
| 1️⃣ | Image old Dell laptop using Macrium Reflect X |
| 2️⃣ | Create Macrium Rescue USB (WinPE) using Rufus |
| 3️⃣ | Boot Lenovo (F2), disable Secure Boot, enable USB Boot |
| 4️⃣ | Boot from rescue USB, restore image to Lenovo's drive |
| 5️⃣ | Troubleshoot mouse/driver issues in Rescue UI |
| 6️⃣ | Reboot into Windows, resolve PIN and networking issues |
| 7️⃣ | Install missing drivers, restore system access |
| 8️⃣ | Repair Settings app, verify system integrity |

---

## 🔧 BIOS + Rescue Setup
| Action | Instructions |
|--------|--------------|
| 🔐 Boot machine, F2 into BIOS | Disable Secure Boot | BIOS → Security → Secure Boot → Disabled |
| 💽 Enable USB Boot | BIOS → Boot Options → USB Boot → Enabled |
| ⏏️ Boot Manager | Press `F12` on startup → select UEFI USB Device |

---

## 🛠️ Macrium Rescue Creation (via Rufus)
Use Rufus to write the ISO:
```text
Device: [Your USB Drive]
Boot selection: MacriumRescue.iso
File system: FAT32
Mode: ISO Image Mode (Recommended)
```

---

## 🔁 Image Restore Process
1. Boot Lenovo from rescue USB
2. Attach external drive with `.mrimg` file
3. Open Macrium Reflect
4. Choose **Restore Image**
5. Select **all partitions**, match destination disk layout
6. Proceed with restore
7. Make sure the destination drive is AT LEAST as large as the source drive

---

## 🖱️ Rescue UI Issues Fix
If trackpad doesn't work:
- Plug in USB mouse
- Or inject drivers before rescue creation via: `Update Drivers → Devices and Drivers`

---

## 🔑 PIN Issue: Post-Restore Login Blocked
After restore, you may see:
> "Your PIN is no longer available due to a change to the security settings on this device"

Resolution:
1. Boot Macrium Rescue
2. Open **Command Prompt**

### 👤 Enable Admin User (Offline Registry Edit)
```powershell
regedit
```
Load hive:
- File → Load Hive
- Path: `C:\Windows\System32\config\SAM`
- Name it: `OfflineSAM`

Edit path:
```
HKEY_LOCAL_MACHINE\OfflineSAM\SAM\Domains\Account\Users\000001F4
```
Modify `F` binary value:
- Change 11 → 10 (enables admin account)

Unload hive and reboot.

### 👤 Create Emergency User
In Macrium Rescue → Command Prompt:
```cmd
copy C:\Windows\System32\utilman.exe C:\Windows\System32\utilman.bak
copy C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```
Reboot → At login screen, click **Ease of Access** icon → opens `cmd.exe`:
```cmd
net user recoveryuser P@ssword123 /add
net localgroup administrators recoveryuser /add
```
Reboot and log in as `recoveryuser`

To revert:
```cmd
copy /y C:\Windows\System32\utilman.bak C:\Windows\System32\utilman.exe
```

---

## 🌐 Network Driver Fix (Offline)
1. Download WLAN driver from Lenovo on another PC
2. Transfer via USB (e.g., `nys3050fjy0lv9f0.exe`)
3. Insert USB after booting into recovery user
4. Identify USB drive (e.g., `E:`)
5. Run installer via CMD:
```cmd
E:\nys3050fjy0lv9f0.exe
```
6. Reboot, verify Wi-Fi available, **set up PIN**

---

## 🛡️ Windows System Repair
### ✅ System File Check
```cmd
sfc /scannow
```

### 🧱 Component Store Repair
```cmd
DISM /Online /Cleanup-Image /RestoreHealth
```

### 🔁 Re-register Built-in Apps
```powershell
Get-AppxPackage -AllUsers | Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```

---

## 👨‍🔧 Final Steps
- ✅ Run Lenovo Vantage to finish all driver updates
- ✅ Confirm `Device Manager` has no missing drivers
- ✅ Re-enable Secure Boot in BIOS if desired
- ⚠️ Windows Hello/Sign-in Options panel may require profile reset if still spinning

---

## 🧪 Troubleshooting Login Panel
### Create a New Local Admin
```powershell
net user tempfix P@ssw0rd! /add
net localgroup administrators tempfix /add
```
Login as `tempfix` → Check `Sign-in Options` screen. If it works, migrate to new profile.

---

## ✅ Mission Complete!
You've completed a full cross-hardware Windows 11 migration from Dell to Lenovo. 🎉

This doc is a reference for others attempting similar low-level system recoveries.

---

> 🧠 **Author:** Michael Anthony  
> 💻 **System:** Dell XPS → Lenovo V14 G4 IRU  
> 🔁 **Method:** Macrium Reflect Image Restore + Manual Repairs
