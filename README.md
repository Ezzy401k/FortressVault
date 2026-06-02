<p align="center">
  <img src="screenshots/icon.png" alt="FortressVault" width="120" />
</p>

<h1 align="center">FortressVault</h1>

<p align="center">
  <strong>The zero‑knowledge, offline password manager that can lie to attackers.</strong><br>
  No internet permission. Field‑level encryption. Decoy vault. Matrix‑style dashboard.
</p>

<p align="center">
  <a href="https://github.com/YOUR_USERNAME/FortressVault/releases/latest">
    <img src="https://img.shields.io/badge/Download-APK-brightgreen" alt="Download APK" />
  </a>
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT" />
  </a>
  <a href="https://f-droid.org">
    <img src="https://img.shields.io/badge/F‑Droid-coming_soon-orange" alt="F‑Droid" />
  </a>
</p>

---

## 🧐 Why FortressVault?

Most password managers connect to the internet. Even “offline” ones encrypt the entire database as a single blob – if an attacker cracks the master password, everything is exposed.  
**FortressVault is different.**

| Traditional Managers | FortressVault |
|----------------------|--------------|
| Entire database encrypted with one key | **Field‑level encryption** – every password, TOTP secret, and file gets its own AES‑256 key |
| No built‑in coercion defence | **Decoy Vault** – a second password opens a harmless dummy vault |
| Cloud sync or manual file copy | **Zero internet permission** – completely air‑gapped |
| Leak checks require internet | **Offline leak checker** – import a Bloom filter and check locally |
| Health score hidden or missing | **Matrix‑style dashboard** with live vault integrity gauge |

---

## ✨ Features

### 🔐 Security & Encryption
- **Field‑level encryption** – each entry has its own unique DEK, wrapped by a master KEK
- **Argon2id key derivation** (64 MiB, 3 iterations, 4 parallelism)
- **Hardware‑backed Keystore** – master key stored in TEE/StrongBox
- **Biometric unlock** with invalidated keys on new fingerprint/face
- **Brute‑force protection** – progressive lockout ladder with configurable limits
- **Hardcore self‑destruct** – wipe all data after 10 failed attempts
- **Anti‑screenshot & anti‑clipboard capture**
- **Memory safety** – all plaintext zeroed after use

### 🧅 Anti‑Coercion
- **Decoy Vault** – hand over a fake password under duress; the real vault stays hidden
- **App icon camouflage** (coming soon) – change the launcher icon to a calculator/notepad

### 📊 Dashboard & Monitoring
- **Matrix‑rain health gauge** – see your vault’s integrity in real‑time
- **Leaked / Weak / Reused / Expiring** indicators – tap to filter
- **Offline leak checker** – import a HaveIBeenPwned Bloom filter, check passwords locally

### 🔑 Password Management
- Unlimited entries with custom expiry dates
- **Password generator** with configurable character sets
- **Password history** and reuse warnings
- **Live TOTP codes** linked to entries
- **Search & filter** chips (All, Shared, Expiring)
- **Autofill service** for apps and browsers

### 📁 Encrypted File Attachments
- Import any file (PDF, image, document) – encrypted with its own key
- View, save, or share securely
- **Biometric gate** before viewing/saving
- Filter by type (Images, Documents, Archives, etc.)

### 📲 Offline QR Sharing
- Share passwords and files with trusted contacts via QR codes
- **ECIES encryption** – only the intended recipient can decrypt
- Expiry dates on shared passwords
- **Pending import queue** – receive files even when vault is locked

### 🧬 24‑Word Recovery Phrase
- BIP‑39 compatible backup
- Forced verification during setup
- Unlock screen recovery – never get locked out

### 🎨 User Experience
- Tactical hardware theme (sharp rectangles, monospace fonts)
- Dark / light mode with animated transition
- Glass‑morphic animated dock
- Built‑in user guide accessible from any screen

---

## 📸 Screenshots

<p align="center">
  <img src="screenshots/unlock.png" width="200" />
  <img src="screenshots/dashboard.png" width="200" />
  <img src="screenshots/vault.png" width="200" />
  <img src="screenshots/decoy_setup.png" width="200" />
  <img src="screenshots/matrix_health.png" width="200" />
  <img src="screenshots/leak_warning.png" width="200" />
</p>

> **Tip:** Take screenshots on a Pixel emulator with `scrcpy` or Android Studio. Add them to a `screenshots` folder in your repo.

---

## 📥 Download & Install

### Option 1 – Download the APK (recommended)

1. Go to the [Releases page](https://github.com/YOUR_USERNAME/FortressVault/releases/latest)
2. Download the `FortressVault-vX.X.X.apk` file
3. On your phone, enable “Install from unknown sources” for your browser or file manager
4. Install and enjoy!

**Verify the SHA‑256 checksum:**
