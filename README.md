# bltool – BitLocker-compatible Disk Decryption & NTFS Reader for Windows

`bltool` is a command‑line utility for Windows that unlocks a BitLocker‑encrypted drive using a 48‑digit recovery key, decrypts sectors with the AES‑XTS algorithm, and mounts the contained NTFS volume as a regular drive letter – **without** calling any Microsoft BitLocker APIs or tools. It implements its own BitLocker metadata parser, key derivation (PBKDF2 + AES‑CCM), sector decryption, NTFS structures, and WinFsp integration.

---

## Features

- 🔓 **Unlock BitLocker volumes** using a 48‑digit recovery key  
- 🧩 **Support multiple BitLocker versions** (Windows 7 / 8 / 10 / 11)  
- 🔑 **Full decryption chain**: recovery key → PBKDF2 → AES‑CCM → VMK → KDF → FVEK  
- 🛡️ **AES‑XTS sector decryption** (128/256 bit)  
- 📁 **Complete NTFS reader**  
  - Boot sector parsing  
  - MFT with fixup, attribute lists, runlist decoding  
  - Fragmented file support  
  - Large directory handling (index allocation)  
  - Path resolution (`\folder\file.txt` → MFT reference)  
- 🚀 **Performance caches** (sector cache + MFT record cache)  
- 💾 **Mount as a Windows drive** using [WinFsp](https://winfsp.dev/)  
- 🖥️ **Simple CLI** with `info`, `unlock`, `mount`, `list`, `read` commands  

All this is achieved **without** any reliance on `manage-bde`, the BitLocker GUI, or internal Windows encryption services.

---

## Prerequisites

- **Windows 11** (should also work on Windows 10 / 8.1 / 7)  
- **OpenSSL 3.x** (libcrypto) – for AES, PBKDF2, CCM, KDF  
- **WinFsp** – filesystem mounting framework  

You can install WinFsp from [winfsp.dev](https://winfsp.dev/). OpenSSL can be obtained via [vcpkg](https://vcpkg.io/), NuGet, or prebuilt binaries.

---

## Building from Source

### 1. Clone the repository
```bash
git clone https://github.com/yourname/bltool.git
cd bltool

2. Install dependencies
OpenSSL: e.g. with vcpkg

bash
vcpkg install openssl:x64-windows
WinFsp: download and run the installer from the official site.

3. Configure with CMake
bash
mkdir build
cd build
cmake .. -DOPENSSL_ROOT_DIR="C:\path\to\openssl" -DWINFSP_ROOT_DIR="C:\path\to\winfsp"
Adjust the paths to where OpenSSL and WinFsp are installed.

4. Build
bash
cmake --build . --config Release
The executable bltool.exe will be placed in build\Release\.

Usage
text
bltool <command> [options]
Commands
Command	Description
info <drive>	Show BitLocker version, protectors, and encryption parameters.
unlock <drive> --recovery-key KEY	Test unlocking (derives FVEK, no mount).
mount <drive> --recovery-key KEY --output Z:	Unlock and mount the volume as drive Z:.
list <path>	List contents of a directory (e.g., Z:\folder).
read <file>	Dump a file’s content to stdout (e.g., Z:\file.txt).
Examples
bash
bltool info E:
bltool unlock E: --recovery-key 123456-123456-123456-123456-123456-123456-123456-123456
bltool mount E: --recovery-key KEY --output Z:
bltool list Z:\Windows\System32
bltool read Z:\readme.txt > readme.txt
While mounted, you can browse the drive with Windows Explorer, run dir Z:, etc. The filesystem is read‑only – writes are not supported.

Architecture Overview
text
Raw Disk Reader (WinAPI)
        ↓
BitLocker Metadata Parser (FVE headers, protectors)
        ↓
Recovery Key → PBKDF2 → AES-CCM → Decrypted VMK
        ↓
VMK → KDF → FVEK (AES-XTS key)
        ↓
AES-XTS Sector Decryptor
        ↓
Decrypted Disk Layer (with optional sector cache)
        ↓
NTFS Boot Sector Parser
        ↓
MFT Parser (fixup, attribute list, runlist)
        ↓
Runlist Engine (VCN→LCN translation)
        ↓
Path Resolver (Windows path → MFT reference)
        ↓
File Reader / Directory Parser
        ↓
WinFsp Filesystem Callbacks
        ↓
Windows Mount Point (e.g., Z:\)
All modules are independent and communicate through well‑defined interfaces, making the code easy to maintain and extend.

Limitations
Only recovery key unlock – TPM, password, and other protectors are not supported.

Read‑only – no writing or encryption of new data.

No EFS / compressed file support – these NTFS features are ignored.

Windows only – relies on WinAPI and WinFsp.

Administrator privileges required for raw disk access and mounting.

License
This project is released under the MIT License. You are free to use, modify, and distribute it.

Contributing
Contributions are welcome! If you find a bug or have an idea for an improvement, please open an issue or submit a pull request.
Make sure to follow the existing code style and add appropriate tests.

Acknowledgments
The WinFsp team for the excellent filesystem framework.

OpenSSL for the cryptographic primitives.

The dislocker and libbde projects for inspiring the BitLocker metadata analysis.

Microsoft for (unintentionally) providing a challenge with BitLocker’s closed‑source design.

