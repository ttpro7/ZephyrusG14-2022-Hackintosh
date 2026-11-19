# Zephyrus G14 2022 Hackintosh EFI

OpenCore EFI for the **ASUS ROG Zephyrus G14 (2022, GA402RJ)**  
**AMD Ryzen 7 6800HS** + **Radeon RX 6700S**

The `main` branch is the EFI that boots **macOS Tahoe 26.7.1 (25G241)** on this machine. It replaces the May 2025 tree and the [`rog2026`](https://github.com/ttpro7/ZephyrusG14-2022-Hackintosh/releases/tag/rog2026) release (OpenCore 1.0.6).

---

## SMBIOS

Serial, MLB, system UUID, and ROM are **blank**. Generate your own before using iCloud or iMessage.

- Product: `MacBookPro16,2`
- `UpdateSMBIOSMode`: `Custom`
- Use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) or the [Dortania SMBIOS guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html)
- Do not reuse a serial from a screenshot or from an older copy of this EFI

---

## Tested on 19 November 2026

One GA402RJ, 16 GB RAM, 1 TB NVMe. This is what that install was doing, not a promise for every board revision.

### Working

- Boot into macOS Tahoe 26.7.1 (25G241)
- CPU reported as AMD Ryzen 7 6800HS, 8 cores / 16 threads
- Radeon RX 6700S with Metal; internal panel 2560×1600 at 120 Hz
- Software brightness (`AppleBacklightDisplay` + `AMDBacklight.kext`)
- Built-in speakers and microphone (`SimpleHDA` + `AppleALC`, layout-id 33)
- Wi-Fi (`SimpleBCM.kext`, `en0` active)
- Bluetooth (BCM20702 controller on)
- Precision trackpad (`ASUE120A` through VoodooI2C, including the Apple multitouch driver)
- Internal keyboard (VoodooPS2)
- USB webcam
- Battery status
- Internal NVMe

### Not working

- Radeon 680M iGPU is disabled. Only the RX 6700S is used.
- `Hypervisor.framework` is unavailable (`kern.hv_support = 0`) even though `RyzenHV.kext` is loaded.
- Intel Wi-Fi is not in this EFI. This is the Broadcom build. An AX210 needs a different kext set.

### Not retested

- Sleep. The tested install has `pmset sleep 0`. Do not assume S3 works. Older notes used BIOS 312 and [Smokeless UMAF](https://github.com/DavidS95/Smokeless_UMAF) to enable S3. Flashing is at your own risk, and **GA402RJ is not GA402RK**.
- HDMI, an inserted SD card, and Windows boot from the picker. A Windows partition is present and `ScanPolicy` is 0, but Windows was not booted for this update.
- Hardware Fn brightness keys. The brightness slider is active; Fn routing depends on ASUS Control, which is not in this repository.

---

## OpenCore

| File | Build |
| --- | --- |
| `EFI/OC/OpenCore.efi` | Acidanthera **1.0.8** RELEASE |
| `EFI/BOOT/BOOTx64.efi` | Acidanthera **1.0.8** RELEASE |
| `OpenRuntime.efi`, `OpenCanopy.efi`, `CleanNvram.efi` | Acidanthera **1.0.4** RELEASE |

`OpenCore.efi` and `BOOTx64.efi` match the official 1.0.8 RELEASE hashes. The three 1.0.4 files are the ones this install actually boots with. They were not swapped for 1.0.8, because that change was not tested. Try any driver alignment from a USB stick first.

Boot args:

```text
amdbl=0 rhv_enable_svm=1 rhv_allow_users=1 rhv_hv=1 shda-lang=en
```

`SecureBootModel` is `Disabled`. `csr-active-config` is SIP enabled (`00000000`).

---

## Kexts

| Kext | Version | Role |
| --- | --- | --- |
| Lilu | 1.7.2 | plugin host |
| VirtualSMC | 1.3.8 | SMC |
| SMCAMDProcessor | 1.0.1 | AMD sensors |
| SMCBatteryManager | 1.3.7 | battery |
| SMCRadeonGPU | 0.3.3 | dGPU sensors |
| AppleALC | 1.9.8 | audio layout |
| SimpleHDA | 1.0.1 | built-in speaker path |
| AMDBacklight | 1.0.0 | panel brightness |
| GVAForce | 1.1.1 | media path assist |
| ForgedInvariant | 1.5.0 | AMD kernel-patch companion |
| AMDRyzenCPUPowerManagement | 0.7.2 | CPU power |
| RyzenHV | 0.7.8 | SVM helper; hypervisor still unavailable |
| AMFIPass | 1.4.1 | AMFI |
| RestrictEvents | 1.1.6 | board-id / update UI |
| AppleMCEReporterDisabler | 1.2 | MCE panic workaround |
| NoTouchID | 1.0.3 | missing Touch ID |
| USBPro | 11.0 | USB map |
| ItlXhciKick | 0.1.0 | USB controller kick |
| VoodooI2C + VoodooI2CHID | 2.9.1 | precision trackpad |
| VoodooPS2Controller | 2.3.8 | keyboard |
| SimpleBCM | 1.2.1 | Broadcom Wi-Fi and Bluetooth |
| CtlnaSDXC | 1.9.0 | card reader |
| NVMeFix | 1.1.3 | NVMe power |
| Innie | 1.3.1 | internal drive icon |
| ECEnabler | 1.0.6 | embedded controller |
| AsusBridge | 0.4.1 | companion for ASUS Control |
| HoRNDIS | 9.2 | USB tethering |

`itlwm` and `AirportItlwm` are not in this tree. Enable only the Wi-Fi kext that matches the card you actually have.

---

## Install

1. Generate SMBIOS for `MacBookPro16,2` and write it into [EFI/OC/config.plist](EFI/OC/config.plist).
2. Copy `EFI/` to the ESP. Keep the previous EFI on a USB stick.
3. BIOS 312 is still the recommended GA402RJ version. The image is not stored in git. Do not flash a GA402RK file.

ASUS Control 0.4.13 was used with `AsusBridge.kext` on the tested install. The app is not included here.

The BIOS image, UMAF, and the utilities disk image are also not in git. `Utilities.dmg` is over GitHub's 100 MB file limit, and the BIOS image should not be redistributed from this repository.

---

## Disclaimer

Provided as is. The author is not responsible for a failed BIOS flash, data loss, or an unbootable disk. Back up the ESP and the system before replacing EFI.

Questions: [open an issue](https://github.com/ttpro7/ZephyrusG14-2022-Hackintosh/issues).
