# Changelog

## 2026-10-21

### Intel/Killer Wi-Fi and Bluetooth EFI update

- Updated `EFI/OC/config.plist` to the OCLP Stage 1 configuration.
- Enabled `IOSkywalkFamily.kext`, `IO80211FamilyLegacy.kext`, its `AirPortBrcmNIC.kext` plugin, and `AirportItlwm.kext`.
- Enabled the `IOSkywalkFamily` downgrade block required by the Sequoia legacy Wi-Fi stack.
- Disabled `itlwm.kext` so it cannot conflict with `AirportItlwm.kext`.
- Enabled the existing BCM4360 spoof at `PciRoot(0x0)/Pci(0x14,0x3)` for OCLP detection.
- Confirmed the existing Intel Bluetooth kexts remain enabled: `IntelBluetoothFirmware.kext`, `IntelBTPatcher.kext`, and `BlueToolFixup.kext`.
- Confirmed Bluetooth NVRAM keys and `csr-active-config` (`03080000`) are present.
- Copied the repository `EFI` tree to the mounted pendrive at `/run/media/parth/1401/EFI`.

### Required post-install step

1. Boot macOS with the Stage 1 `EFI/OC/config.plist`.
2. Run OpenCore Legacy Patcher's **Post-Install Root Patch**.
3. After patching and rebooting, use `EFI/OC/config-postpatch.plist` as the active `config.plist`.
4. Confirm the spoof key is commented in Stage 2, while `AirportItlwm.kext` remains enabled and `itlwm.kext` remains disabled.
5. Reset NVRAM from the OpenCore picker if Bluetooth does not appear.

The EFI changes cannot be tested from Linux; Wi-Fi and Bluetooth functionality must be confirmed after booting macOS and completing the OCLP root patch.
