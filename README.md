# Hackintosh EFI for Predator Helios 16 (macOS 15 Sequoia)
### Intel / Killer Wi-Fi (AirportItlwm + OCLP) & Intel AX201 Bluetooth Setup

This repository contains the OpenCore EFI created via **NullMoth 1401** and modified to enable **Native Intel / Killer Wi-Fi** and **Intel AX201 Bluetooth** on macOS 15 Sequoia.

---

## Hardware Overview
- **System**: Acer Predator Helios 16 (PH16-71)
- **CPU**: 13th Gen Intel Core i5-13500HX (Raptor Lake, 14 Cores / 20 Threads)
- **Chipset**: Intel 700 Series Chipset Family (HM770 / Alder Lake PCH)
- **Wi-Fi Card**: Intel / Killer Wi-Fi 6 AX201 / CNVi (`8086:7a70` at `0000:00:14.3`)
- **Bluetooth**: Intel AX201 Bluetooth (`8087:0026` connected to XHCI USB Port 14)
- **Ethernet**: Realtek / Killer E2600 Gigabit Ethernet (`10ec:2600`)

---

## Summary of Changes & File Documentation

### 1. `EFI/OC/config.plist` (Current Mode: macOS Installer / Recovery)
- **USB Ports (Restored)**:
  - Enabled `UTBDefault.kext` alongside `USBToolBox.kext` to ensure **every physical USB port is active** during the installation. Your 2.4G mouse/keyboard receiver and Android phone will work on any port.
- **Android USB Tethering (`HoRNDIS.kext`)**:
  - Injected `HoRNDIS.kext` (v9.3 64-bit for OpenCore) so your Android phone provides immediate internet connectivity to the macOS Recovery installer via USB cable.
- **Kernel > Quirks**:
  - Enabled `DisableIoMapperMapping: True` to prevent AppleVTD DMA stalls on network/USB devices.
- **Wi-Fi Spoof & Block (Temporarily Disabled for Installer)**:
  - Disabled the Skywalk downgrade block and commented out the Broadcom BCM4360 spoof (`#PciRoot(0x0)/Pci(0x14,0x3)`). Spoofing must not be active in Recovery because the installer ramdisk cannot run OCLP root patches.

### 2. `EFI-PostInstall/` (For After macOS is Installed onto SSD)
- Contains **`config-oclp-stage1.plist`** and **`config-oclp-stage2.plist`** ready to copy over to your EFI once you complete the macOS installation and boot into the desktop.
- Stage 1 enables the BCM4360 spoof and Skywalk downgrade for OpenCore Legacy Patcher (OCLP) to detect your card.
- Stage 2 disables the spoof after OCLP root patches are applied so `AirportItlwm.kext` provides native Wi-Fi.

### 3. `EFI/OC/Kexts/UTBMap.kext`
- Custom USB injector map generated specifically for this machine's Intel 700 Series XHCI controller (`0000:00:14.0`).
- **Bluetooth Fix**: Designates Port 14 (`HS14`) as `UsbConnector = 255` (Internal).
  - *Why*: macOS requires internal Bluetooth controllers to have type 255. Without this, `bluetoothd` drops power, firmware fails to upload, or the adapter disconnects upon sleep/wake.
- Designates Port 6 (`HS06`) as `UsbConnector = 255` (Internal ACER HD Webcam).
- Maps physical USB 3 Type-A ports (`HS01`, `HS02`, `HS03`, `SS01`, `SS02`, `SS03`) with `UsbConnector = 3`, keeping total ports strictly under Apple's 15-port limit.

### 4. `EFI/OC/Kexts/IOSkywalkFamily.kext`
- Downgrades `com.apple.iokit.IOSkywalkFamily` to allow `IO80211FamilyLegacy` to bind in macOS 14 & 15.

### 5. `EFI/OC/Kexts/IO80211FamilyLegacy.kext`
- Restores the legacy Wi-Fi framework (includes `AirPortBrcmNIC.kext` plugin).

### 6. `EFI/OC/Kexts/AirportItlwm.kext`
- Stable v2.3.0 Ventura release of AirportItlwm. Operates seamlessly under the restored legacy stack in Sequoia.

### 7. `EFI/OC/config.plist.orig`
- Clean backup of the original unedited NullMoth 1401 configuration.

---

## Step-by-Step Instructions to Complete Setup

### Step 1: Boot with Stage 1 Config
1. Your USB drive (`/run/media/parth/1401/EFI`) and this local EFI are already configured with the Stage 1 `config.plist`.
2. Boot into macOS 15 Sequoia using this EFI.
3. Open Terminal and verify SIP status with `csrutil status`.

### Step 2: Run OpenCore Legacy Patcher (OCLP)
1. Download and install [OpenCore-Legacy-Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Launch OpenCore Legacy Patcher.
3. Click on **Post-Install Root Patch**.
4. OCLP will detect the spoofed Broadcom card (`BCM4360`) and display **Networking: Modern Wireless**.
5. Click **Start Root Patching** and enter your administrator password.
6. Once the patch finishes, restart your system.

### Step 3: Switch to Stage 2 (Disable Wi-Fi Spoof)
1. Mount your EFI partition.
2. In `EFI/OC/`:
   - Either replace `config.plist` with `config-postpatch.plist` (rename `config-postpatch.plist` to `config.plist`), OR
   - Edit `config.plist` and add a `#` in front of `PciRoot(0x0)/Pci(0x14,0x3)` under `DeviceProperties > Add`.
3. Reboot your system.
4. **Done!** Native Apple Wi-Fi menu bar, Control Center, Wi-Fi networks, AirPlay, and Bluetooth will now be fully operational!

---

## Bluetooth Troubleshooting Tips
- If Bluetooth does not immediately appear after root patching:
  1. Restart and press Space at the OpenCore boot picker.
  2. Select **Reset NVRAM** (ResetNvramEntry is enabled in the EFI).
  3. Boot macOS.

---

## USB Tethering Guide (Android & iPhone)

### 1. USB Port Mapping (Hardware Layer)
- **What was fixed**: The previous setup used an unmapped dummy configuration (`UTBDefault.kext`), which caused macOS to exceed its 15-port limit on the 700-series chipset (26 ports total). Physical USB ports dropped or power-cycled when high-bandwidth devices (like tethered phones) were plugged in.
- **Current status**: Physical USB 3 ports are now explicitly mapped via `UTBMap.kext`.

### 2. iPhone USB Tethering
- iPhone USB tethering is supported natively in macOS. Once plugged into any mapped USB port, enable **Personal Hotspot > Allow Others to Join** on your iPhone and tap "Trust This Computer".

### 3. Android USB Tethering on macOS 15 Sequoia
macOS **does not natively support Android's default RNDIS protocol**. On macOS 15 Sequoia, legacy `HoRNDIS.kext` is obsolete and causes kernel panics or boot freezes. Use either of the following modern methods:

- **Method A: Native CDC-NCM / Ethernet Tethering (No software needed)**:
  1. Connect your Android phone via USB cable.
  2. On Android, go to **Settings > Network & Internet > Hotspot & Tethering**.
  3. If available on your Android version, toggle **Ethernet Tethering** (or in Developer Options > **Default USB Configuration**, select **USB Ethernet / NCM**).
  4. macOS recognizes this natively as a standard USB Ethernet interface.

- **Method B: TetherKit (Modern user-space RNDIS for Sequoia)**:
  1. Open Terminal in macOS and install [TetherKit](https://github.com/XiaoMiku01/TetherKit):
     ```bash
     brew install XiaoMiku01/tap/tetherkit
     ```
  2. Launch TetherKit and enable USB tethering on your phone.
  3. It creates a virtual network interface without needing unstable kernel extensions or touching SIP!

