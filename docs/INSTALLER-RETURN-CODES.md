# BillBasket Windows Installer — EXE Return Codes

Reference for Windows App Store / Windows Package Manager (winget) submission.  
These are the standard Win32 error codes returned by the BillBasket Inno Setup installer
for each installation outcome scenario.

---

## Return Code Reference

| Scenario | Return Code | Win32 Constant |
|----------|:-----------:|----------------|
| Installation successful | **0** | `ERROR_SUCCESS` |
| Installation cancelled by user | **1602** | `ERROR_INSTALL_USEREXIT` |
| Application already exists | **1638** | `ERROR_PRODUCT_VERSION` |
| Installation already in progress | **1618** | `ERROR_INSTALL_ALREADY_RUNNING` |
| Disk space is full | **112** | `ERROR_DISK_FULL` |
| Reboot required | **3010** | `ERROR_SUCCESS_REBOOT_REQUIRED` |
| Network failure | **1201, 1203, 1204, 1205** | `ERROR_NO_NETWORK`, `ERROR_NO_NET_OR_BAD_PATH`, `ERROR_BAD_NET_NAME`, `ERROR_NETWORK_BUSY` |
| Package rejected during installation | **1625** | `ERROR_INSTALL_PACKAGE_REJECTED` |

---

## Scenario Details

### Installation successful
**Return code: `0`**  
Installation has been successful. The application is installed and ready to use.

---

### Installation cancelled by user
**Return code: `1602`**  
The install operation was cancelled by the user before it completed.  
Inno Setup maps its "user cancelled" exit (code 2) to Win32 `ERROR_INSTALL_USEREXIT` (1602) for store compatibility.

---

### Application already exists
**Return code: `1638`**  
The application already exists on the device. Another version of this product is already installed.  
Upgrade or uninstall the existing version before reinstalling.

---

### Installation already in progress
**Return code: `1618`**  
Another installation is already in progress. The user needs to complete or cancel the current installation before proceeding with this install.

---

### Disk space is full
**Return code: `112`**  
The disk space is full. The installer requires sufficient free space on the target drive to extract and install all files.

**Minimum disk space required:** ~350 MB (standard build); ~800 MB (AI-bundled build).

---

### Network failure
**Return codes: `1201`, `1203`, `1204`, `1205`**  
Custom return codes for network-related failures during installation:

| Code | Constant | Description |
|------|----------|-------------|
| 1201 | `ERROR_NO_NETWORK` | No network is present |
| 1203 | `ERROR_NO_NET_OR_BAD_PATH` | Network path not found |
| 1204 | `ERROR_BAD_NET_NAME` | Network name cannot be resolved |
| 1205 | `ERROR_NETWORK_BUSY` | Network is busy |

BillBasket is an offline-first installer; network is only required for optional cloud-license activation post-install.

---

### Package rejected during installation
**Return code: `1625`**  
The package was rejected during installation due to a security policy enabled on the device (e.g., Group Policy, Windows Defender Application Control, or enterprise device management restrictions).

---

### Reboot required
**Return code: `3010`**  
A restart is required to complete the installation. The installer has finished, but some components (e.g., runtime redistributables) will not be active until the device is rebooted.

---

## Inno Setup Default Exit Codes

For reference, the native Inno Setup exit codes before Win32 mapping:

| Inno Setup Code | Meaning |
|:-:|---|
| 0 | Setup finished successfully |
| 1 | Setup failed (fatal error) |
| 2 | User cancelled setup |

The installer script in `windows/installer/inno_setup.iss` uses the Win32-compatible codes listed in the table above when submitted via Windows Package Manager.

---

## winget Manifest Reference

When authoring the winget package manifest (`BillBasket-LLP.BillBasket.installer.yaml`),
declare non-zero success codes and expected return codes as follows:

```yaml
InstallerSuccessCodes:
  - 3010   # reboot required but install succeeded

InstallerReturnCode:
  - InstallerReturnCode: 1602
    ReturnResponse: cancel
  - InstallerReturnCode: 1638
    ReturnResponse: alreadyInstalled
  - InstallerReturnCode: 1618
    ReturnResponse: installInProgress
  - InstallerReturnCode: 112
    ReturnResponse: diskFull
  - InstallerReturnCode: 1625
    ReturnResponse: packageInUse
  - InstallerReturnCode: 3010
    ReturnResponse: rebootRequiredToFinish
```

---

*Last updated: 2026-06-26 · BillBasket 1.0.83*
