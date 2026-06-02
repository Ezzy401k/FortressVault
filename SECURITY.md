
# Security Architecture of FortressVault

This document describes how FortressVault protects your data, what attacks it defends against, and what its limitations are.  
**Transparency is a core security feature.**

---

## 1. Zero‑Knowledge, Offline‑Only Design

* **No Network Footprint:** FortressVault does **not request the `INTERNET` permission**. The Android sandbox physically prevents it from sending data anywhere.
* **Local Processing:** All encryption, key derivation, and decryption happen exclusively on your device.
* **Complete Privacy:** No analytics, no telemetry, no crash reporters – **nothing leaves your phone**.

---

## 2. Master Key Encryption Key (KEK) Architecture

```text
 ┌─────────────────────────┐
 │    Random Master KEK    │ ← Permanent, never changes
 │        (AES‑256)        │
 └────────────┬────────────┘
              │
   ┌──────────┼──────────┐
   │          │          │
 ┌─▼────────┐┌─▼───────┐┌─▼──────────────┐
 │ Password ││Biometric││Recovery Phrase │
 │   Key    ││   Key   ││      Key       │
 │(Argon2id)││  (TEE)  ││(BIP-39 Phrase) │
 └──────────┘└─────────┘└────────────────┘

```

1. **Master KEK** – A random 256‑bit key generated when the vault is created. It encrypts every database on disk (SQLCipher).
2. **Password Key** – Derived from your master password using **Argon2id** (64 MiB memory, 3 iterations, 4 parallelism). The KEK is wrapped (encrypted) with this key and stored.
3. **Biometric Key** – Stored in Android Keystore (TEE/StrongBox), secured behind a biometric gate. It wraps the same KEK, allowing fingerprint/face unlock.
4. **Recovery Phrase Key** – Derived from your 24‑word BIP‑39 phrase (PBKDF2‑HMAC‑SHA512, 2048 iterations). It wraps the KEK for emergency recovery.

> 💡 **Key advantage:** Changing your master password only re‑wraps the KEK. The databases themselves are never re‑encrypted, making password updates instantaneous.

---

## 3. Field‑Level Encryption (FLE)

Every sensitive field in the vault has its own **Data Encryption Key (DEK)**:

```text
 ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
 │  Password #1  │ │  Password #2  │ │    File #1    │
 │  DEK₁ + IV₁   │ │  DEK₂ + IV₂   │ │  DEK₃ + IV₃   │
 └───────┬───────┘ └───────┬───────┘ └───────┬───────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
                   ┌───────▼───────┐
                   │  Master KEK   │
                   │(wraps all DEKs)│
                   └───────────────┘

```

* Each DEK is a random 256‑bit AES key.
* The plaintext is encrypted with **AES‑256‑GCM** using the DEK and a random 12‑byte IV.
* The DEK itself is wrapped by the Master KEK and stored alongside the ciphertext.
* If an attacker miraculously breaks one DEK, they only gain access to **one specific field/password** – not the entire vault.

---

## 4. Key Derivation (KDF)

* **New vaults:** Argon2id (64 MiB memory, 3 iterations, 4 parallelism). Argon2id is memory‑hard, making GPU/ASIC brute-force attacks extremely expensive.
* **Old vaults:** PBKDF2‑HMAC‑SHA256 (600,000 iterations). Vaults created before the Argon2id migration are automatically upgraded upon the next successful unlock.
* **Decoy vault:** PBKDF2‑HMAC‑SHA256 (100,000 iterations). Intentionally weaker, allowing forensic analysis to distinguish it from the real vault if coerced.

---

## 5. Biometric Unlock

* The biometric secret is stored in the **Android Keystore**, backed by the Trusted Execution Environment (TEE) or StrongBox hardware.
* The key is configured with `setUserAuthenticationRequired(true)`, ensuring it can only be accessed following a valid fingerprint or face scan.
* The key is **permanently invalidated** if a new fingerprint or face profile is enrolled on the device—a critical security best practice.

---

## 6. Brute‑Force Protection

| Failed Attempts | Consequence |
| --- | --- |
| **1–3** | No lockout, standard error toast only |
| **4** | 30‑second lockout |
| **5** | 5‑minute lockout |
| **6** | 2‑hour lockout |
| **7** | 24‑hour lockout |
| **8** | 3‑day lockout |
| **9** | 7‑day lockout |
| **10** | Infinite 7‑day lockout **OR** immediate wipe (if *"Hardcore Self‑Destruct"* is enabled) |

* Lockout timers utilize `SystemClock.elapsedRealtime()` (hardware uptime clock) to resist local clock tampering.
* The attempt counter is stored safely inside **EncryptedSharedPreferences**, preventing modification by a rooted attacker.
* Upon device reboot, any active lockout timer is instantly re‑applied.

---

## 7. Decoy Vault (Anti‑Coercion)

* A alternative master password opens a physically separate, independent encrypted database (`fortress_decoy.db`).
* The decoy vault looks and behaves identically to the real vault but contains only user-configured dummy data.
* Biometric authentication **never** maps to or opens the decoy vault.
* The decoy password implements a lower PBKDF2 iteration count (100,000). This deliberate cryptographic trade‑off allows specialized forensic analysis to distinguish it from the real password under the hood, while remaining virtually invisible to an attacker in a live coercion scenario.

---

## 8. Offline Leak Checker

* Users can securely import a custom Bloom filter file built from the popular *HaveIBeenPwned* password repository.
* The filter is queried **entirely offline**. The password’s SHA‑1 hash is checked locally; your credentials never touch the network.
* The filter file is loaded via memory‑mapped I/O (`MappedByteBuffer`) to prevent loading the entire 800 MB+ file directly into system RAM.

---

## 9. Data at Rest

* All underlying SQLite databases are encrypted using robust **SQLCipher (AES‑256‑CBC)** implementation.
* Data is separated into four isolated databases: `passwords`, `TOTP`, `files`, and `contacts`.
* The passphrase for each isolated database is the primary Master KEK (256‑bit random key).
* All database files reside exclusively inside the app’s private internal storage directory, making them inaccessible to other applications on the device.

---

## 10. Clipboard Sanitisation

* When a password or TOTP code is copied, the system clipboard is automatically **cleared after 30 seconds**.
* This operation is handled by an isolated Android **foreground service**, ensuring it executes flawlessly even if the main application is backgrounded or killed.

---

## 11. Anti‑Screenshot Protection

* Every app Activity explicitly sets the `FLAG_SECURE` window flag.
* The Android OS actively blocks screenshots, screen recording applications, and hides window previews/thumbnails within the recent apps switcher.

---

## 12. Known Limitations

* **Android Only:** The application currently lacks official support for iOS or Desktop environments.
* **No Third‑Party Audit:** The codebase has not been independently verified or audited by an external firm yet.
* **Hardcoded App‑Sharing Key:** The same static AES key is currently baked into every official FortressVault installation. While protected by your master password at rest, a dedicated attacker extracting it from the APK could decipher the metadata of intercepted setup QR codes. *(Note: Actual passwords remain protected by localized ECIES encryption to the specific recipient).*
* **Hardware Token Constraints:** Native YubiKey / NFC hardware token support is still under active development on the project roadmap.

---

## 13. Reporting a Vulnerability

If you discover a security vulnerability, please **do not** open a public GitHub Issue. Instead, securely report it via email to **esraelmekdem@gmail.com** with the following details:

* A clear, concise description of the security issue
* Technical steps required to reliably reproduce the behavior
* The affected application version(s)
* Any proposed mitigations or fixes

Security reports are prioritised immediately. Expect an initial acknowledgment response within **72 hours**. If verified, a patch will be prepared immediately, and you will receive proper attribution and credit in our release notes (if desired).

---

## 14. Verifying the APK

Every formal GitHub Release includes a public SHA‑256 checksum. To verify that your downloaded production APK perfectly matches the audited source code layout, execute the following command in your terminal terminal:

```bash
sha256sum FortressVault-vX.X.X.apk

```
