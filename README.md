# Lenovo Thinkpad T450 & T450s Hackintosh Guide for Mojave, Catalina, Big Sur, Monterey, Ventura & Sequoia with OpenCore 1.0.7
This repo contains the installation guide and EFI files required to get a perfectly functional Monterey, Big Sur, Catalina and Mojave Hackintosh on your T450 or T450s since they share the same hardware. Everything is stable and functional as described in this Readme.

**This EFI also supports macOS Ventura and macOS Sequoia using the same OpenCore version. Ventura and Sequoia require additional post-install root patches through [OpenCore Legacy Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher), and both require additional Intel Wi-Fi setup as described in the guide.**

## A few worthy mentions about this repo:

- **This guide is not for models with Haswell 4th gen CPU**
- **The patched ACPI files were first created by [EchoEspirit](https://github.com/EchoEsprit/Hackintosh-Catalina-OpenCore-Lenovo-T450s-efi). I tweaked a couple of things and fixed some errors that were happening on T450 + added Intel WiFi drivers from [Openintelwireless](https://github.com/OpenIntelWireless)**
- **I will try my best to keep the repo updated with the latest kexts and OpenCore version**
- **This EFI works with macOS Monterey, Big Sur, Catalina, Mojave, Ventura and Sequoia using OpenCore 1.0.7**
- **This EFI is Configured with Monterey in mind. If you are using it on Big Sur, Catalina or Mojave read the the whole guide to know where to make the necessary changes**
- **With every EFI update you retrieve from here please remember to go through the post install guide**

![img](https://img.shields.io/badge/Last%20Update-June-red) ![img](https://img.shields.io/badge/macOS%20Support-Monterey--12-blue) ![img](https://img.shields.io/badge/OpenCore%20Version-0.7.0-yellow)

![About macOS Monterey](https://i.imgur.com/hIZ3lkb.png)

## Quick Navigation

- [SMBIOS Generation & Configuration](#smbios-generation--configuration)
- [macOS Monterey Online Installer](#macos-monterey-online-installer)
  - [Windows Guide](#windows-guide)
  - [macOS Guide](#macos-guide)
  - [Linux Guide](#linux-guide)
- [macOS Ventura & Sequoia Online Installer](#macos-ventura--sequoia-online-installer)
  - [Installation](#installation)
  - [Post-Install — OpenCore Legacy Patcher (Ventura & Sequoia)](#post-install--opencore-legacy-patcher-ventura--sequoia)
  - [Ventura & Sequoia Post-Install — Intel Wi-Fi](#ventura--sequoia-post-install--intel-wi-fi)
- [macOS Monterey Offline Installer](#macos-monterey-offline-installer)

# Introduction

EFI folder and Guide for Thinkpad T450 and T450s Hackintosh Monterey.

- `Tested CPUs`: **i5-5200U/5300u & i7-5600u**
- `Integrated Graphics`: **HD Graphics 5500**
- `Sound Card`: **ALC292**
- `Wireless Cards Tested`: **Intel 7265/7260 & Intel AX210**

# Bios

- `Security -> Security Chip`: **Disabled**;
- `Memory Protection -> Execution Prevention`: **Enabled**;
- `Virtualization -> Intel Virtualization Technology`: **Enabled**;
- `Virtualization -> Vt-directed IO`: **Disabled**;
- `Internal Device Access -> Bottom Cover Tamper Detection`: must be **Disabled**;
- `Anti-Theft -> Computrace -> Current Setting`: **Disabled**;
- `Secure Boot -> Secure Boot`: **Disabled**;
- `UEFI/Legacy Boot`: **UEFI Only**;
- `Fingerprint Sensor`: **Disabled** `(Causes issues with wake from sleep)`;
- `CSM Support`: **Yes**.

**Note: If you can't see any boot entries with CMS support set to Yes, change it to NO. After this you will get a garbled screen, to fix this put the laptop to sleep by closing the lid till the light starts blinking and wake it up**


# What works

- Sleep / Wake
- Wifi and Bluetooth (Intel® Dual Band Wireless-AC 7265 or 7260 cards with Airportitlwm.kext) **(Note: the intel kexts for wifi and bluetooth come with some issues, see post install notes for more info, new Airportitlwm Monterey kext & fixes)**
- AirPort Extreme (Broadcom BCM94360CSAX & NGFF A/E Adapter) **Recommended Upgrade to get native WiFi & Bluetooth**
- Handoff, Continuity, AirDrop
- iMessage, FaceTime, App Store, iTunes Store (see post install guide for more info)
- Ethernet
- Onboard audio (see post install guide for more info)
- USB 2.0 / USB 3.0
- Dual Batteries
- Touchpad
- Trackpoint
- miniDP
- SD Card Reader (Enable Sinetek-rtsx.kext in Config.plist because it is unstable to be left on by default)
- HiDPI (Use [one-key-hidpi](https://github.com/xzhih/one-key-hidpi))
- Sidecar (see post install guide for more info)

# What doesn't work

- VGA

## Note: If you need to edit Config.plist, don't use OpenCore configurator or Clover configurator, use PlistEdit pro, PropperTree, or Xcode.

# Installation Guide


## SMBIOS Generation & Configuration

Before installing macOS, it is recommended to generate a **unique SMBIOS** for your machine and apply it to `config.plist`.

A properly configured SMBIOS is important for Apple services such as **iMessage, FaceTime, iCloud, App Store, and other iServices**.

### Required Tools

- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) — Generate SMBIOS information.
- [OpenCore Auxiliary Tools (OCAT)](https://github.com/ic005k/OCAuxiliaryTools) — Edit and validate `config.plist`.

### 1. Choose an Appropriate SMBIOS

Choose an SMBIOS that is appropriate for the hardware and macOS version you are installing.

For this T450/T450s EFI, the configured SMBIOS may be:

```text
MacBookPro12,1
```

**Do not copy SMBIOS values from another Hackintosh or from another person's configuration.** Generate your own values.

### 2. Generate SMBIOS with GenSMBIOS

Download and run [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).

1. Select **Install/Update MacSerial** if required.
2. Select **Generate SMBIOS**.
3. Enter the SMBIOS model you selected, for example:

```text
MacBookPro12,1
```

GenSMBIOS will generate values including:

- `SystemProductName`
- `SystemSerialNumber`
- `MLB`
- `SystemUUID`
- `ROM`

> **Important — the SMBIOS must correspond to a real Apple Mac:** For iServices to work correctly, the generated SMBIOS should correspond to a real Apple Mac model/device. After generating the SMBIOS, verify the **serial number** using Apple's official [Check Coverage](https://checkcoverage.apple.com/) website. The serial should be recognized by Apple and the result should correspond to the Mac model you selected. If Apple does not recognize the serial or it does not correspond to the expected model, generate another SMBIOS and check again until you get a valid device match.

> **Do not use the SMBIOS of a real Mac that you own or have access to.** Generate a new SMBIOS with GenSMBIOS instead. Never publish your serial number, MLB, SystemUUID, or ROM in this repository, screenshots, or support requests.

### 3. Apply the SMBIOS Using OCAT

Open your EFI's `config.plist` with [OpenCore Auxiliary Tools (OCAT)](https://github.com/ic005k/OCAuxiliaryTools).

Navigate to:

```text
PlatformInfo → Generic
```

Enter the values generated by GenSMBIOS:

| OpenCore field | GenSMBIOS value |
|---|---|
| `SystemProductName` | SMBIOS Type |
| `SystemSerialNumber` | Serial |
| `MLB` | Board Serial |
| `SystemUUID` | SmUUID |
| `ROM` | ROM |

Example:

```text
PlatformInfo
└── Generic
    ├── SystemProductName = MacBookPro12,1
    ├── SystemSerialNumber = <your generated serial>
    ├── MLB = <your generated board serial>
    ├── SystemUUID = <your generated UUID>
    └── ROM = <your generated ROM>
```

Save the `config.plist` after applying the values.

### 4. Verify the SMBIOS

Before continuing with the installation, check the generated serial number using Apple's [Check Coverage](https://checkcoverage.apple.com/) page.

The serial should:

- Be recognized by Apple.
- Correspond to the Mac model selected in GenSMBIOS.
- Not be copied from another Hackintosh or from a Mac you own.

If the serial is not recognized or does not correspond to the selected Mac model, generate another SMBIOS and check it again.

### 5. iServices

A valid and properly generated SMBIOS is an important requirement for iServices. It does **not** guarantee that every iService will work by itself.

After applying the SMBIOS, follow the [Dortania iServices guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) for the remaining iServices configuration.

This is especially relevant for:

- iMessage
- FaceTime
- iCloud
- App Store
- Other Apple services that use device identification

> **Keep all SMBIOS identifiers private.** Do not publish your serial number, MLB, SystemUUID, or ROM in your GitHub repository.

---

## macOS Monterey & Big Sur Online Installer

This section applies both for **macOS Monterey** & **macOS Big Sur**. The online/recovery installer is recommended because it downloads the required macOS files during installation.

### Windows Guide

1. Download [Rufus](https://rufus.ie/en/).
2. Select the USB flash drive under **Device**.
3. Select `Non-bootable` as the boot selection (**REQUIRED**).
4. Select `FAT-32` or `Large FAT-32` as the partition scheme.
5. Open the USB partition in File Explorer and delete the files created by Rufus.
6. Create a folder named `com.apple.recovery.boot` on the USB.
7. Install [Python](https://www.python.org/downloads/) and make sure **Add Python to PATH** is enabled.
8. Download and extract the [OpenCore Package](https://github.com/acidanthera/OpenCorePkg/releases).
9. Open `Utilities/macrecovery/` from the extracted OpenCore package.
10. Right-click the folder and choose **Copy as path**.
11. Open Command Prompt, type `cd `, paste the copied path, and press Enter.
12. Run:

```bash
python macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000
```

13. Copy **`BaseSystem.dmg`** and **`BaseSystem.chunklist`** to the `com.apple.recovery.boot` folder on the USB.
14. Download the latest EFI from the [Releases](https://github.com/racka98/Lenovo-Thinkpad-T450-T450s-Hackintosh-Guide-Opencore/releases) page.
15. Copy the `EFI` folder to the root of the USB partition.
16. **Make sure the correct BIOS settings above have been applied before continuing.**
17. Restart the laptop and press `F12`.
18. Select the USB flash drive as the temporary boot device.
19. In the OpenCore picker, select the USB installer/recovery entry.
20. Follow the macOS installer and install **Monterey**.

### macOS Guide

1. Launch **Disk Utility**.
2. Select **View → Show All Devices**.
3. Select your USB flash drive and format it as `MS-DOS (FAT)` / `FAT-32`.
4. Open the USB partition and create `com.apple.recovery.boot`.
5. Download and extract the [OpenCore Package](https://github.com/acidanthera/OpenCorePkg/releases).
6. Open `Utilities/macrecovery/`.
7. Right-click the folder and select **New Terminal at Folder**.
8. Run:

```bash
python3 macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000
```

9. Copy **`BaseSystem.dmg`** and **`BaseSystem.chunklist`** into `com.apple.recovery.boot`.
10. Download the latest EFI from the [Releases](https://github.com/racka98/Lenovo-Thinkpad-T450-T450s-Hackintosh-Guide-Opencore/releases) page.
11. Copy the `EFI` folder to the root of the USB partition.
12. **Make sure the correct BIOS settings above have been applied before continuing.**
13. Restart the laptop and press `F12`.
14. Select the USB flash drive as the temporary boot device.
15. In the OpenCore picker, select the USB installer/recovery entry.
16. Follow the macOS installer and install **Monterey**.

### Linux Guide

Follow the [Dortania macOS Online Installer guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/linux-install.html#downloading-macos) to create the online macOS installer in Linux.

---

## macOS Ventura & Sequoia Online Installer

The same online/recovery installer method can be used for **macOS Ventura and macOS Sequoia**. Both versions require **OpenCore Legacy Patcher (OCLP)** on the T450 because the Intel HD Graphics 5500 is a Broadwell iGPU and requires root patching on these newer macOS versions. [OCLP Post-Install](https://dortania.github.io/OpenCore-Legacy-Patcher/POST-INSTALL.html)

### Installation

Use the Windows, macOS, or Linux online-installer procedure from the Monterey section above to create the recovery USB. When booting the installer, select the macOS installer/recovery entry from OpenCore and complete the installation.

Make sure the correct BIOS settings and your generated SMBIOS are already configured before installation.

### Post-Install — OpenCore Legacy Patcher (Ventura & Sequoia)

After Ventura or Sequoia boots:

1. Download the latest [OpenCore Legacy Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Open OCLP and allow it to detect the system.
3. Build/install OpenCore to your internal EFI if required by your setup.
4. Open **Post-Install Root Patch** / **Post-Install Volume Patch**.
5. Allow OCLP to download any required patching resources.
6. Apply the available graphics/root patches.
7. Restart when prompted.
8. After reboot, verify that Intel HD 5500 graphics acceleration is working.

The T450's **Intel HD Graphics 5500 (Broadwell)** requires root patching on Ventura, Sonoma, and Sequoia. OCLP provides the required legacy graphics patches. Root patches can be removed by macOS updates, so they may need to be re-applied after future updates. [OCLP Post-Install](https://dortania.github.io/OpenCore-Legacy-Patcher/POST-INSTALL.html)

### Ventura & Sequoia Post-Install — Intel Wi-Fi

**This section applies to both Ventura and Sequoia.** Both versions require the HeliPort setup described below for Intel Wi-Fi.

The required **`itlwm.kext` is already included in this EFI**, so there is no need to download or add another copy. This guide only covers installing **HeliPort** and configuring it to start automatically after every login.

Do **not** use `AirportItlwm.kext` for this Ventura/Sequoia setup.

For Ventura and Sequoia, use **HeliPort v2.0.0-alpha** from the official OpenIntelWireless GitHub releases page:

- [HeliPort Releases](https://github.com/OpenIntelWireless/HeliPort/releases)
- [HeliPort v2.0.0-alpha](https://github.com/OpenIntelWireless/HeliPort/releases/tag/v2.0.0-alpha)

> **Warning:** HeliPort v2.0.0-alpha is a testing/pre-release build. Use it for Sequoia compatibility and be aware that it may contain bugs.

#### 1. Copy the EFI from the USB to the SSD

After successfully installing macOS and confirming that the system boots correctly from the USB:

1. Boot macOS using the **EFI on the USB**.
2. Download and open [OpenCore Auxiliary Tools (OCAT)](https://github.com/ic005k/OCAuxiliaryTools).
3. Mount the **USB EFI partition** and the **internal SSD EFI partition**.
4. Make a backup of the EFI currently on the SSD.
5. Copy the working **`EFI` folder from the USB** to the **EFI partition of the internal SSD**.
6. Open the SSD's `EFI/OC/config.plist` in OCAT and verify that it contains the configuration you used to successfully boot from USB.
7. Save the configuration.
8. Reboot and remove the USB.
9. Confirm that macOS boots directly from the internal SSD.

> **Important:** Do not skip this step. The USB EFI is the known-working EFI used during installation. Copy it to the SSD before continuing with the HeliPort setup.

#### 2. Install HeliPort

1. Boot into Sequoia from the **internal SSD EFI**.
2. Download **HeliPort v2.0.0-alpha** from the official GitHub release page above.
3. Open the downloaded file and install **`HeliPort.app`** into `/Applications`.
4. Launch HeliPort.
5. Select your Wi-Fi network and connect.

> **Note:** `itlwm.kext` provides Intel Wi-Fi through HeliPort rather than Apple's native Wi-Fi menu. This is expected behavior with the `itlwm + HeliPort` method.

#### 3. Add HeliPort as a Login Item

To make HeliPort start automatically after every boot/login:

1. Open **System Settings**.
2. Go to **General → Login Items & Extensions**.
3. Under **Open at Login**, click the **`+`** button.
4. Select **HeliPort.app** from the **Applications** folder.
5. Make sure **HeliPort** appears under **Open at Login**.

After this, HeliPort will automatically launch when you log into macOS, so you do not need to manually open it after every boot.

> **Tip:** If HeliPort does not automatically connect to Wi-Fi after login, open HeliPort once and connect to your Wi-Fi network manually. The exact auto-connect behavior may vary with the HeliPort version.

### Ventura vs. Sequoia Post-Install

| Feature | Ventura | Sequoia |
|---|---|---|
| OCLP required | Yes | Yes |
| Broadwell HD 5500 root patching | Yes | Yes |
| HeliPort required for Intel Wi-Fi | **Yes** | **Yes** |
| `itlwm.kext` + HeliPort | Required | Required |
| OCLP root patches after macOS updates | May be required | May be required |

If OCLP needs internet access to obtain additional patching resources, use Ethernet or establish Wi-Fi first. OCLP may require another root-patching run after networking is available so that all required patches can be installed.

For both Ventura and Sequoia, reinstall OCLP root patches after macOS updates if they are removed. [OCLP Updating Guide](https://dortania.github.io/OpenCore-Legacy-Patcher/UPDATE.html)

---

## macOS Monterey Offline Installer

The offline installer method is provided as an alternative when an online/recovery installer is not suitable.

### macOS Guide

1. Download [gibMacOS](https://github.com/corpnewt/gibMacOS).
2. Run `gibMacOS.command` on a Mac.
3. Select the desired macOS Monterey version.
4. Download the installer files.
5. Open the downloaded `InstallAssistant.pkg` to place the macOS installer application in `/Applications`.
6. Open **Disk Utility** and format your USB as `Mac OS Extended (Journaled)` with a `GUID Partition Map`.
7. Create the installer using Apple's `createinstallmedia` tool. For example:

```bash
sudo /Applications/Install\ macOS\ Monterey.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume
```

Replace `MyVolume` with the name of your USB partition.

8. Mount the USB's EFI partition.
9. Download the latest EFI from the [Releases](https://github.com/racka98/Lenovo-Thinkpad-T450-T450s-Hackintosh-Guide-Opencore/releases) page.
10. Copy the `EFI` folder to the EFI partition.
11. **Make sure the correct BIOS settings above have been applied before continuing.**
12. Restart the laptop and press `F12`.
13. Select the USB flash drive as the temporary boot device.
14. In the OpenCore picker, select **Install macOS Monterey**.
15. Complete the installation and then follow the **Post Install** section.

### Important Notes

- The online installer is recommended for Monterey.
- Ventura and Sequoia are also supported by this EFI, and both require additional OCLP graphics patching and Intel Wi-Fi setup as described above.
- If the online installer gives you a black screen or freezes during installation, try recreating the installer using macOS.
- Make sure your BIOS settings are correct before troubleshooting OpenCore or the installer.
- After installation, complete the SMBIOS/iServices configuration and the rest of the Post Install guide.

# Post Install
Once you have verifed that your machine boots properly without any issues as described in the "What Works section", proceed to do the following

### 1. Disable Verbose mode (the black screen with logs on boot up)
In Config.plist, navigate to NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args and delete the `-v` argument

### 2. Disable AppleDebug, ApplePanic & ShowPicker
In the Config.plist, naviaget to Misc -> Debug and change both `AppleDebug` and `ApplePanic` to False (NO)

You can also disable the boot picker screen so that you boot straight to th Apple logo by setting `ShowPicker` under Misc -> Boot to False (NO)

Note: you can still see the boot picker with ShowPicker set to no/false by spamming Esc before the apple logo is displayed during boot.

### 3. Enable WiFi with the Intel card on Monterey, Catalina and Mojave
If you are on Catalina or Mojave, you can enable WiFi on the Intel card by navigating (in config.plist) to Kernel -> Add -> 20 and set Enabled to False/NO (Disabling Airportitlwm.kext) and in 21 set Enabled to True/YES (Enabling itlwm.kext). After enabling these and rebooting install Heliport App (included in Utilities).

Or you can use Airportitlwm.kext for Catalina from Intel WiFi Kexts folder and get native wifi on Catalina in the expense of loosing trackpad after wake from sleep.

**For those on Monterey & Big Sur you can comfortably use the Airportitlwm.kext included as the trackpad issues after sleep do not happen on Big Sur.**

**Note:**

  **1. Airportitlwm.kext gives you native WiFi menu and enables location services, but often causes issues with the trackpad & trackpoint after waking from sleep (it doesn't work) on Catalina and Mojave (not Big Sur). A quick fix is to put the laptop to sleep again by closing the lid until the red sleep light starts to blink then waking the laptop again. Also it only happens when you put the laptop to sleep for a very long time (more than 2 or 3 hours). So for those who don't put their laptop to sleep for a very long time and just turn it off after use, this kext is ok to use.**
  
  **2. The Airportitlwm.kext included in this EFI is for Big Sur. For those in Monterey take the kext in Monterey folder inside Intel WiFi kexts and replace the one in EFI -> Kexts. For those in Catalina or Mojave you should use the one in Intel WiFi Kexts Folder of this repo (Recommended) and replace the one in EFI -> Kexts.**
Note: the Airportitlwm kext for macOS Monterey is very new and may have issues. Please report those issues [here](https://github.com/OpenIntelWireless/itlwm/issues).
  
 **3. Airportitlwm causes the bluetooth to be unstable and because so you may experience stutters or interruptions while using bluetooth headphones. To fix this you can turn off wifi and connect via ethernet or you can get 8x series cards to fix this or buy the recomended cards (DW1820A 00JT494 or Broadcom BCM94360CSAX)**

  
### 4. Enable Caps lock indicator and additional Thinkpad features you used to get on Windows
  Using [YogaSMC](https://github.com/zhen-zen/YogaSMC) you can gain this functionality back. Install the YogaSMC App-Release from [here](https://github.com/zhen-zen/YogaSMC/releases).
  Install it then open it to set it up.
  
### 5. Enabling bluetooth toggle to be able to turn off bluetooth
You can enable IntelBluetoothFirmware.kext & IntelBluetoothInjector.kext to be able to turn off bluetooth by enabling those kexts in config.plist
This is not done by default because it increases boot times
***For those on macOS Monterey do not enable these kexts because the system will not boot***
  
### 6. Add Device Properties for Serial number, MLB, ROM, Sytem-UUID and optionally SystemProductName.
Follow this [guide](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html#generate-a-new-serial) to set up serial number and the accompanying info to get iServices

If you want to get wired sidecar working, in Config.plist change the string in Platforminfo > Generic> SystemProductName to `MacBook9,1` (note: this causes the battery to drain faster)


### 7. Fix USB mouse side buttons.
If you are using a usb mouse with side buttons, you can spoof apple usb mouse by change the pid and vid in AnyAppleUSBMouse.kext/Info.plist and enable it in Config.plist.


### 8. Fixing static noise
When you connect headphones/earbuds via the headphone jack you will hear static noise. To fix this install alc_fix_new located in Utilities folder of this repo.
