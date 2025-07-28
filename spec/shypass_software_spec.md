# Technical Specification Document: ShyPass (Password Manager App)

---

## 1. Overview

ShyPass is a cross-platform secure password manager that allows users to store, manage, and retrieve their credentials across devices. It features end-to-end encryption, biometric access (on mobile), and secure vault sharing.

---
## 2. Definitions

| Term                  | Definition                                                                 |
|-----------------------|----------------------------------------------------------------------------|
| AES-256               | Advanced Encryption Standard using 256-bit keys; strong symmetric encryption. |
| PBKDF2                | Password-Based Key Derivation Function 2; used to derive encryption keys from passwords. |
| JWT                   | JSON Web Token; compact, URL-safe means of representing claims securely.   |
| 2FA / TOTP            | Two-Factor Authentication / Time-based One-Time Password for enhanced login security. |
| Vault                 | A secure collection where encrypted items like passwords are stored.       |
| Encrypted Data        | Information transformed using an algorithm to prevent unauthorized access. |
| Frontend              | The user-facing part of the application (UI).                              |
| Backend               | Server-side logic and APIs that power app features.                        |
| API                   | Application Programming Interface; allows communication between systems.   |
| MongoDB               | A NoSQL document database used to store user data and vault items.         |
| S3                    | Amazon Simple Storage Service; object storage for encrypted file attachments. |
| Node.js               | JavaScript runtime environment used for backend development.               |
| React / React Native  | JavaScript libraries for building web and mobile interfaces respectively.  |
| OAuth                 | Open standard for access delegation, used here for Google login.           |
| GitHub Actions        | CI/CD automation platform used to build, test, and deploy the app.         |
| Firebase              | Mobile and web app development platform used here for mobile distribution. |
| Keystore / Keychain   | Secure storage provided by Android/iOS for storing cryptographic keys.     |
| HTTPS                 | HyperText Transfer Protocol Secure; encrypts data in transit.              |
| Zero Knowledge        | Design principle where server cannot access sensitive data in plaintext.   |
| TOTP                  | Time-based One-Time Password algorithm used for 2FA.                       |
| Docker                | Tool for containerizing apps for consistent deployment across environments.|
| WebSocket             | Protocol enabling real-time communication between clients and servers.     |

## 3. Core Features
<br> 

### 3.1 Vault Management  
SecurePass allows users to create and manage a personal vault containing encrypted items such as passwords, secure notes, and credit card information. Each entry is encrypted on the client device before being stored in the backend.

### 3.2 Cross-Platform Sync  
The application supports seamless synchronization of vault data across web and mobile devices. This ensures that users have up-to-date access to their credentials on any platform, facilitated by secure APIs and token-based authentication.

### 3.3 End-to-End Encryption  
All user data is protected with AES-256 encryption performed on the client side. SecurePass operates on a zero-knowledge principle, meaning plaintext data is never accessible to the server. The master password is never transmitted.

### 3.4 Biometric Unlock (Mobile)  
On supported mobile devices, users can unlock their vault using biometric authentication methods such as Face ID or fingerprint scanning. This enhances security while providing convenience for quick access.

### 3.5 Vault Sharing  
SecurePass includes a vault sharing mechanism that enables users to share specific credentials with trusted contacts. Shared entries are encrypted using shared keys and permissions can be configured for view or edit access.

### 3.6 Password Generator  
A built-in password generator helps users create strong, random passwords that meet specified criteria (e.g., length, symbols, numbers). This tool encourages good security hygiene.

### 3.7 Auto Logout & Timeout  
For security, SecurePass includes automatic logout functionality after a period of user inactivity. This minimizes the risk of unauthorized access on unattended devices.

### 3.8 Emergency Access  
Users can designate emergency contacts who can request access to the vault after a defined waiting period. This feature is useful in scenarios where users become incapacitated.

### 3.9 Offline Access (Mobile)  
The mobile application caches encrypted data locally, allowing users to access their vault even without an internet connection. Changes made offline are synced once the device reconnects.

### 3.10 Two-Factor Authentication (2FA)  
SecurePass supports TOTP-based two-factor authentication, adding an extra layer of security during login. Users can enable 2FA using any compatible authenticator app.

---

## 4. Technology Stack

| Layer          | Technology                                        |
|----------------|---------------------------------------------------|
| Frontend       | React.js (Web), React Native (Mobile)             |
| Backend        | Node.js + Express                                 |
| Database       | MongoDB (vaults encrypted at-rest)                |
| Encryption     | AES-256 + PBKDF2 key derivation                   |
| Authentication | JWT + Google OAuth + TOTP (2FA)                   |
| Hosting        | AWS EC2 + MongoDB Atlas                           |
| Storage        | AWS S3 (for encrypted attachments)                |
| CI/CD          | GitHub Actions + Firebase App Distribution        |

---

## 4. System Components & Data Flow

![ShyPass Architecture Diagram](Assets\Images\shypass_architecture_diagram.png)


### 4.1. **Frontend (React for Web, React Native for Mobile)**
This layer serves as the primary interface for users to interact with **ShyPass**. It is responsible for the login screen, password vault UI, file upload/download modules, and shared vaults.

- Users enter their **master password**, which remains safe with it and never transmitted over the network.
- The frontend initiates the **local key derivation** (PBKDF2) and handles **AES encryption/decryption**.
- When viewing or editing a vault entry:
  - The frontend **requests encrypted data** from the API Server.
  - It **decrypts the data locally** and displays it to the user.
- When saving a new item:
  - Data is **encrypted locally**.
  - Only ciphertext is sent to the backend via **HTTPS**.

---

### 4.2. **Local Crypto (AES-256, PBKDF2)**
All cryptographic operations are handled on the client using secure platform-native libraries (e.g., WebCrypto, CryptoKit).

- The **master key** is derived from the user’s master password using **PBKDF2**.
- Vault entries are encrypted and decrypted using **AES-256**.
- Plaintext data never leaves the device.

**Data Flow:**
`
Master Password → PBKDF2 → AES Master Key → Encrypt/Decrypt Vault Data  
Encrypted Data → API Server → MongoDB
`

---

### 4.3. **Key Storage**
Session keys are stored using secure local methods depending on the platform:

- **Mobile**: iOS Keychain, Android Keystore  
- **Web**: IndexedDB (in-memory or encrypted)  
- **Desktop**: macOS Keychain, Windows DPAPI  
  
- When a session ends, the key is destroyed and the user must re-authenticate.

**Data Flow:**
`Derived AES Key → Stored in Memory → Used for all crypto operations during session
`
---

### 4.4. **API Server (Node.js)**
A stateless server acting as a **secure proxy** between client and storage.

- Handles **user authentication**, token issuance (JWT), and vault-related CRUD operations.
- Does **not decrypt or access plaintext data**.
- Communicates only via **HTTPS (TLS 1.2+)**.

**Data Flow:**
`Frontend → API Server → Encrypted Data Stored in MongoDB  
API Server → Client → Delivers Ciphertext for Local Decryption
`
---

### 4.5. **MongoDB (Encrypted Vault Storage)**
It is the primary database and stores:

- Encrypted vault items  
- User metadata (hashed passwords, settings)  
- Audit logs  

- Event if it get compromised, the vault data remains **cryptographically secure**.

**Data Flow:**

`API Server → MongoDB → Stores Encrypted Vault Items  
MongoDB → API Server → Returns Encrypted Data to Frontend`


---

### 4.6. **AWS S3 (Encrypted Attachments)**
Used for uploading and retrieving **encrypted files**, such as ID proofs, notes, or secure documents.

- Files are encrypted **on the client** using the user’s derived key.
- Backend issues **signed S3 URLs** for upload and download.
- Files remain encrypted in transit and at rest.

**Data Flow:**

`Client Encrypts File → Upload to S3 (via Signed URL)  
S3 → Download Encrypted File → Decrypt in Frontend
`
---


---

## 5. Security Model
<br>


| Feature           | Explanation                                                  |
|------------------|--------------------------------------------------------------|
| Zero Knowledge   | Server never sees plaintext or master password               |
| End-to-End Crypto| Encryption & decryption happen only on the client side       |
| AES-256          | Industry standard symmetric encryption for vault items       |
| PBKDF2           | Key derivation with high iteration count to resist brute-force |
| TOTP 2FA         | Adds time-based secondary auth during login                  |
| Secure Storage   | Platform-specific secure key storage (Keychain, Keystore)    |

---

## 6. Non-Functional Requirements
<br>
### 6.1 Performance  
The system must respond to user interactions within 300 milliseconds for 95% of operations, ensuring a smooth and responsive experience. API response times should not exceed 500 milliseconds under typical loads.

### 6.2 Scalability  
SecurePass should be horizontally scalable to support an increasing number of users and vault items without significant degradation in performance. The architecture should support load balancing and distributed databases to accommodate growth.

### 6.3 Availability  
The system must maintain at least 99.9% uptime, excluding scheduled maintenance windows. Critical services such as authentication, vault access, and file storage must be highly available through redundancy and failover strategies.

### 6.4 Security  
Security is paramount. All communications must occur over HTTPS. Sensitive data must be encrypted at rest and in transit. The platform must comply with OWASP Top 10 guidelines and undergo regular security audits and penetration testing.

### 6.5 Maintainability  
The codebase must follow modular, well-documented standards and be easy to update or extend. Continuous Integration/Continuous Deployment (CI/CD) pipelines should be used to automate testing and deployment processes.

### 6.6 Usability  
The interface should be intuitive for both technical and non-technical users. Accessibility standards (such as WCAG 2.1) must be met to ensure inclusivity for users with disabilities.

### 6.7 Compatibility  
SecurePass must function consistently across modern web browsers (Chrome, Firefox, Safari, Edge) and mobile operating systems (Android 10+ and iOS 13+). Feature parity must be maintained across all platforms.

### 6.8 Data Integrity  
The system must ensure that data is never lost or corrupted during storage, synchronization, or retrieval. All write operations must be atomic and transactional, especially when syncing across devices.

### 6.9 Logging and Monitoring  
Comprehensive logging must be implemented to capture errors, warnings, and system events. Real-time monitoring should be enabled to detect anomalies or downtime, and alert the DevOps team accordingly.

### 6.10 Compliance  
SecurePass must comply with relevant data privacy regulations such as GDPR, CCPA, and any applicable data localization laws. Users should have the ability to delete their data permanently upon request.




---
### Note
Shypass is a fictious application follows a design architecture inspired by open-source password managers such as Bitwarden, with strong encryption practices, client-side key derivation, and modern web-mobile integrations.


[def]: spec\Assets\shypass_acrhitecture_diagram.png