# Death Stranding 2: ON THE BEACH — CrossOver Guide

Run **Death Stranding 2** on Apple Silicon Mac via **CrossOver + CXPatcher**.

---

## 🚀 Setup Guide

### 1. Patch CrossOver with CXPatcher

Download [CXPatcher](https://github.com/italomandara/CXPatcher), point it at your CrossOver.app — it updates D3DMetal and
other components.

### 2. Create bottle

Create a **Windows 10 64-bit bottle** in CrossOver.  
Name it `Steam` (or reuse an existing Steam bottle).

### 3. Configure bottle

Set these in the bottle's `cxbottle.conf` (`[EnvironmentVariables]` section):

```ini
CX_GRAPHICS_BACKEND = d3dmetal
WINEMSYNC = 1
D3DM_SUPPORT_DXR = 1
```

You can also set them via CrossOver GUI → Bottle → Settings:

- Enable **D3DMetal**
- Enable **MSync**
- Add environment variable `D3DM_SUPPORT_DXR = 1`

### 4. Install Steam + DS2

Install Steam in the bottle, then download **DEATH STRANDING 2: ON THE BEACH**.

> **Bottle path:**  
> `~/CXPBottles/Steam/drive_c/Program Files (x86)/Steam/steamapps/common/DEATH STRANDING 2 - ON THE BEACH/DS2.exe`

### 5. Apply the compatibility patch

The game crashes on startup without this.  
Download the [DS2-Mac patcher](https://github.com/davidakh/DS2-Mac) (v1.3+), extract `DS2_Mac_Fix.zip` and run:

```bash
python3 ds2.py "/path/to/DS2.exe"
```

#### Patches included:

| Patch                                | Required       | Notes                         |
|--------------------------------------|----------------|-------------------------------|
| `DXGI_FEATURE_PRESENT_ALLOW_TEARING` | ✅ Required     | Prevents init crash           |
| `DepthBoundsTestSupported`           | ✅ M1–M4        | Skip for M5+ (native support) |
| `Network polling CPU fix`            | 🔶 Recommended | Saves ~40% CPU                |
| `Force HDR detection`                | 🔶 Optional    | Only if display supports HDR  |

#### Manual DXGI fix (if patcher reports "Pattern not found")

Some game builds have a short jump instead of a near jump:

```bash
python3 -c "
path = 'DS2.exe'
with open(path, 'rb') as f:
    data = bytearray(f.read())
off = data.find(bytes.fromhex('44397C243C74'))
if off != -1:
    data[off+5:off+7] = b'\x90\x90'
    with open(path, 'wb') as f:
        f.write(data)
    print(f'Patched DXGI at offset 0x{off+5:X}')
else:
    print('Pattern not found — try another method')
"
```

#### Restore original exe

```bash
python3 ds2.py --restore "/path/to/DS2.exe"
# Or copy from backup:
cp DS2.exe.backup DS2.exe
```

> **Important:** Re-apply the patch after every game update (Steam will overwrite DS2.exe).

---

## 🔧 Known Issues & Fixes

### ❌ "Write permission denied" for save folder

The `Documents` folder in the bottle is a symlink to `~/Documents`.  
Replace it with a real folder:

```bash
BOTTLE=~/CXPBottles/Steam
cd "$BOTTLE/drive_c/users/crossover"
rm Documents
mkdir Documents
mkdir "Documents/DEATH STRANDING 2 - ON THE BEACH"
cp -R ~/Documents/DEATH\ STRANDING\ 2\ -\ ON\ THE\ BEACH/* "Documents/DEATH STRANDING 2 - ON THE BEACH/" 2>/dev/null
chmod -R 777 Documents/
```

### 🎮 Game Mode (optional)

Improves CPU/GPU priority, reduces background activity and lowers Bluetooth latency.  
It's a system-level toggle — the macOS notification won't appear, but the effect is still applied.

Requires Xcode. Usage:

```bash
# Before launching the game:
xcrun gamepolicyctl game-mode set on

# After quitting the game:
xcrun gamepolicyctl game-mode set auto
```

### Automation (optional)

Instead of running the commands manually every time, you can use a **LaunchAgent** — a macOS autorun mechanism. A
`.plist` file in `~/Library/LaunchAgents/` is automatically started by `launchd` when you log in and kept alive in the
background.

The agent below monitors for `DS2.exe` every 5 seconds. When the game launches, it runs
`gamepolicyctl game-mode set on`. When the game exits, it runs `gamepolicyctl game-mode set auto`. No manual commands
needed.

Save as **`~/Library/LaunchAgents/com.ds2.gamemode.plist`**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ds2.gamemode</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>
POLICY="/Applications/Xcode.app/Contents/Developer/usr/bin/gamepolicyctl"
ON=0
while true; do
    PROC=$(pgrep -f "DS2\.exe" | head -1)
    [ -n "$PROC" ] && [ "$ON" = "0" ] && $POLICY game-mode set on 2>/dev/null && ON=1
    [ -z "$PROC" ] && [ "$ON" = "1" ] && $POLICY game-mode set auto 2>/dev/null && ON=0
    sleep 5
done
        </string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
```

Load it:

```bash
launchctl load ~/Library/LaunchAgents/com.ds2.gamemode.plist
```

To remove:

```bash
launchctl unload ~/Library/LaunchAgents/com.ds2.gamemode.plist
rm ~/Library/LaunchAgents/com.ds2.gamemode.plist
```

---

## 🖥️ Recommended In-Game Settings

| Setting              | Value                             |
|----------------------|-----------------------------------|
| **Upscaler**         | PICO or DLSS                      |
| **Low Latency**      | ❌ None                            |
| **Frame Generation** | ❌ None                            |
| **VSync**            | ✅ On                              |
| **HDR**              | Only if you applied the HDR patch |

---

## 📊 Tested configuration

| Component           | Version                              |
|---------------------|--------------------------------------|
| **CrossOver**       | 26.1 (base)                          |
| **CXPatcher**       | 26.1p0.7.1                           |
| **Wine**            | wine-11.0-8720-g4351038808c          |
| **DS2 version**     | v1.10.89.0 (Steam Build ID 23923251) |
| **DS2-Mac patcher** | v1.3                                 |

| Component        | Detail                           |
|------------------|----------------------------------|
| **Hardware**     | Apple M2 Pro, 16 GB RAM          |
| **macOS**        | 26.5.2 (Tahoe)                   |
| **CrossOver**    | CXPatched 26.1p0.7.1 (wine 11.0) |
| **DS2 Build**    | Steam, Build ID 23923251         |
| **WiNE Version** | wine-11.0-8720-g4351038808c      |

---

## 📚 Resources

- [DS2-Mac Patcher](https://github.com/davidakh/DS2-Mac) — Required compatibility patch
- [CXPatcher](https://github.com/italomandara/CXPatcher) — CrossOver patcher for latest D3DMetal
- [CrossOver](https://www.codeweavers.com/crossover) — Compatibility layer
- [AppleGamingWiki: DS2](https://www.applegamingwiki.com/wiki/DEATH_STRANDING_2:_ON_THE_BEACH)
