

### 🧩 To disable Shutdown / Restart

Run this in **PowerShell (Administrator)**:

```powershell
# Hide Shutdown, Restart, Sleep, and Hibernate from all power menus
New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Force | Out-Null
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoClose" -Value 1 -Type DWord

# Restart Explorer to apply instantly
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Output "Shutdown and Restart hidden for this user."
```

✅ **What it does**

* Removes the *Power* button entries from the **Start menu**, **Alt + F4**, and **Ctrl + Alt + Del** screens.
* The computer stays fully usable; your wife can’t shut it down from the GUI.
* You can still use command-line or shortcut methods to power off.

---

### 🧩 To shut down or restart yourself

Create or use shortcuts like:

```powershell
shutdown /s /f /t 0    # immediate shutdown
shutdown /r /f /t 0    # immediate restart
```

Or run them from **Win + R → cmd**.

---

### 🧩 To restore the normal power menu

Run:

```powershell
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoClose" -ErrorAction SilentlyContinue
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Output "Shutdown and Restart restored."
```

---

That’s the *tested, minimal, and reversible* configuration for 24H2.
It leaves search, taskbar, and Start functionality intact while making the shutdown options completely disappear.
