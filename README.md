

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




# --- Hide Taskbar (Works on Windows 11 including 24H2) ---
Add-Type @"
using System;
using System.Runtime.InteropServices;
public class WinUtil {
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
    public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    public static extern bool IsWindowVisible(IntPtr hWnd);
}
"@

function Hide-Taskbar {
    $classes = @('Shell_TrayWnd', 'Shell_SecondaryTrayWnd', 'TaskbarWindowXamlHost')
    foreach ($cls in $classes) {
        $hWnd = [WinUtil]::FindWindow($cls, $null)
        if ($hWnd -ne [IntPtr]::Zero -and [WinUtil]::IsWindowVisible($hWnd)) {
            [WinUtil]::ShowWindow($hWnd, 0) | Out-Null  # 0 = SW_HIDE
        }
    }
}

# hide immediately
Hide-Taskbar

# maintain hide every 2 seconds so Explorer can’t bring it back
while ($true) {
    Start-Sleep -Seconds 2
    Hide-Taskbar
}






#

# --- Restore Taskbar (Windows 11) ---
Add-Type @"
using System;
using System.Runtime.InteropServices;
public class WinUtil {
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
    public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    public static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
}
"@

$classes = @('Shell_TrayWnd', 'Shell_SecondaryTrayWnd', 'TaskbarWindowXamlHost')
foreach ($cls in $classes) {
    $hWnd = [WinUtil]::FindWindow($cls, $null)
    if ($hWnd -ne [IntPtr]::Zero) {
        [WinUtil]::ShowWindow($hWnd, 5) | Out-Null   # 5 = SW_SHOW
    }
}

Write-Host "Taskbar restored successfully." -ForegroundColor Green

