# 🔐 Hack Windows Passwords

**Automated Windows password reset tool. Allows removing local user passwords and creating a backup administrator account.**

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![Security](https://img.shields.io/badge/Security-Testing-red) ![License](https://img.shields.io/badge/License-Educational%20Use-only)

---

## ⚠️ IMPORTANT LEGAL WARNING

**This project must only be used in a legal and ethical framework**: laboratories, test accounts you own, or environments explicitly authorized in writing.
Testing or accessing accounts without authorization is **illegal** and **criminally punishable**. By downloading or using this repository, you agree to respect the rules of responsible use.

**I used the PASS REVELATOR API, which I thank for making this program. If you want to learn more about Windows password security and hacking, I invite you to visit their site: https://passwordrevelator.net/en/unlock-windows-account-without-password**

# PassRevelator - Windows Password Reset

![CLEUSB Logo](./cle-usb-bootable-windows.jpg)

## 📋 Features

- **Automatic detection** of Windows partitions
- **User extraction** from the SAM database
- **Password reset** (clearing LM/NT hashes)
- **Creation of a backup administrator** account with empty password
- **Operation from WinPE** without user interaction
- **Complete logging** of operations

## 🏗️ Code Architecture

### Main Structure

#### `SAMHashExtractor`
- Parses the V structure of the SAM database
- Extracts user information: name, LM hash, NT hash
- Locates password field offsets in the binary structure

#### `PassRevelatorLogon`
Main class orchestrating all operations:

1. **Partition detection** (`find_windows_partitions`)
   - Scans drive letters A-Z
   - Checks for `Windows\System32\config\SAM`
   - Identifies valid Windows installations

2. **User extraction** (`list_users`, `_extract_users_from_sam`)
   - Loads the SAM hive via `reg load`
   - Lists user RIDs (Relative IDs)
   - Exports each user key via `reg export`
   - Parses binary data of the V structure
   - Extracts username and hashes

3. **Password reset** (`reset_password`)
   - First tries `chntpw` if available
   - Otherwise, uses direct registry modification
   - Sets LM and NT hash lengths to zero in the V structure
   - Clears hash data to avoid residues
   - Reimports via `reg import`

4. **Admin account creation** (`create_local_admin`)
   - Builds a minimal V structure for the new user
   - Builds an F structure with active account flags
   - Adds the user in `SAM\Domains\Account\Users`
   - Creates the name→RID mapping in `Users\Names`
   - Adds the user to the Administrators group (RID 0x220)
   - Modifies the SOFTWARE hive to enable admin rights

### SAM Data Structures

#### V Structure (User Information)
- Offset 0xCC: start of variable data
- Offset 0x9C: relative offset of LM hash
- Offset 0xA0: LM hash length (int32 LE)
- Offset 0xA8: relative offset of NT hash
- Offset 0xAC: NT hash length (int32 LE)

To remove a password:
- Set LM and NT lengths to 0
- Clear hash data at calculated offsets

#### F Structure (Account Flags)
- Size: 80 bytes (0x50)
- Offset 0x30: User RID
- Offset 0x38: AccountFlags
  - `0x0010`: UF_NORMAL_ACCOUNT
  - `0x0200`: UF_DONT_EXPIRE_PASSWD
- Offset 0x10: AccountExpires (0x7FFFFFFFFFFFFFFF = never)

## 📝 Logging

All operations are logged to:
- Console: real-time display
- File: `C:\PassRevelator_Auto.log`

## 🤝 Support

- Website: https://www.passwordrevelator.net
- Email: support@passrevelator.net
- Copyright 2026

## 📄 License

This software is provided for educational purposes and system recovery. The user is responsible for its use in compliance with applicable laws.
