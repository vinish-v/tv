# Sony BRAVIA VU3 - Debloat & Custom Launcher Guide

This document records the modifications made to your **Sony BRAVIA VU3** (`BRAVIA_VU3_2K`, Realtek `RTK2.220814.001`, Android 11), how to customize the layout, and how to safely restore or change anything at any time.

---

## 1. Summary of Changes

- **Device**: Sony BRAVIA VU3 (`BRAVIA_VU3_2K`)
- **Firmware**: Realtek Android 11 (API 30), Build `RTK2.220814.001`
- **Home Screen**: Clean, responsive Apple TV 4K-style native launcher (`com.gathin.tvhome`), pitch-black background, 4-app dock, frosted pill buttons (Inputs, Wi-Fi name, Settings).
- **Dock Apps**:
  1. YouTube (`com.google.android.youtube.tv`)
  2. Netflix (`com.netflix.ninja`)
  3. Prime Video (`com.amazon.amazonvideo.livingroom`)
  4. JioHotstar (`in.startv.hotstar`)
- **Remote Buttons**: Remote app buttons (YouTube 201/202, Netflix 191, Prime Video 189) mapped to their respective applications via `InputKeyService`.
- **Tuner & Inputs**: Native Realtek/Sony inputs (HDMI 1, HDMI 2, HDMI 3 / ARC, Video, Live TV) seamlessly integrated in the top-right Inputs pill dropdown menu.
- **Debloated Packages**: 27 packages disabled (Sony promo rows, store demo, help guides, telemetry, analytics, and unselected preloaded services).
- **Kept Intact**: YouTube & YouTube Music, Netflix, Prime Video, JioHotstar, JioCinema / Voot / JioSaavn, Airtel Xstream, NetMirror TV, VLC, MX Player, Send Files to TV, Google Assistant, Chromecast built-in, Realtek firmware updater, Gboard on-screen keyboard, Bluetooth remote pairing, TV inputs framework, and all core Android system components.

---

## 2. How to Re-enable Any App or System Service

Nothing was permanently uninstalled; all disabled packages can be re-enabled instantly with a single command over ADB:

```powershell
# In PowerShell:
$env:TV_HOST="192.168.1.2"
tools\venv\Scripts\python.exe tools\tv.py "pm enable --user 0 <package_name>"
```

For example, to re-enable Sony AirPlay:
```powershell
tools\venv\Scripts\python.exe tools\tv.py "pm enable --user 0 com.sony.dtv.airplayapp"
```

To re-enable SonyLIV:
```powershell
tools\venv\Scripts\python.exe tools\tv.py "pm enable --user 0 com.sonyliv"
```

---

## 3. How to Restore Everything to Factory Default

To revert all changes, re-enable every disabled package, and restore the stock Google TV launcher:

### Windows (PowerShell):
```powershell
powershell -ExecutionPolicy Bypass -File backup\restore_my_tv.ps1 -TvHost 192.168.1.2
```

### Linux / macOS (Bash):
```bash
./backup/restore_my_tv.sh 192.168.1.2
```

---

## 4. How to Change Dock Apps, Columns, or Wallpaper

### Option A: Using Themes (Instant, no rebuild required)
You can push themes with wallpapers or layout tweaks directly:
```powershell
# Push Aurora theme (wallpaper + matching teal/violet theme):
powershell -ExecutionPolicy Bypass -File tools\theme.ps1 -TvHost 192.168.1.2 themes\aurora

# Push Glass theme:
powershell -ExecutionPolicy Bypass -File tools\theme.ps1 -TvHost 192.168.1.2 themes\glass

# Reset back to built-in clean black look:
powershell -ExecutionPolicy Bypass -File tools\theme.ps1 -TvHost 192.168.1.2 --reset
```

### Option B: Editing `launcher/app/src/main/res/values/config.xml`
1. Open `launcher/app/src/main/res/values/config.xml`.
2. Edit `dock_apps` (order of packages pinned in the bottom row).
3. Edit `columns` (4, 5, 6, or 7).
4. Run the build-and-install script:
   ```powershell
   $env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot"
   $env:ANDROID_HOME = "C:\Users\mugen\AppData\Local\Android\Sdk"
   powershell -ExecutionPolicy Bypass -File launcher\build-and-install.ps1 -TvHost 192.168.1.2 -StockLauncher com.google.android.apps.tv.launcherx -InputKeyService
   ```

---

## 5. IMPORTANT: Closing ADB for Security

Port 5555 communicates unencrypted over your Wi-Fi network. Now that the setup is complete, close network ADB:

1. On the TV, go to:
   **Settings → Device Preferences → Developer options**
2. Turn **OFF** "USB debugging" (and "Network/ADB debugging" if shown).
3. Select **Revoke USB debugging authorisations** and click OK.
4. Restart your TV using the remote control.
5. Verify from your computer that port 5555 is closed:
   ```powershell
   tools\venv\Scripts\python.exe tools\adb-closed.py 192.168.1.2
   ```
   (It will return exit code 0 when connection is refused/closed).
