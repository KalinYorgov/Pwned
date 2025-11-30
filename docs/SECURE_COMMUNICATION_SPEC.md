# Secure Communication Specification - Plunderstorm Mobile

**Document ID:** BACK-018
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved
**Classification:** Internal - Security Sensitive

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Security Architecture Overview](#2-security-architecture-overview)
3. [Transport Layer Security (TLS)](#3-transport-layer-security-tls)
4. [Game Traffic Encryption (DTLS)](#4-game-traffic-encryption-dtls)
5. [Certificate Management](#5-certificate-management)
6. [Certificate Pinning](#6-certificate-pinning)
7. [Client-Side Key Storage](#7-client-side-key-storage)
8. [Data Protection in Transit](#8-data-protection-in-transit)
9. [Logging Security](#9-logging-security)
10. [Implementation Guidelines](#10-implementation-guidelines)
11. [Security Testing](#11-security-testing)
12. [Incident Response](#12-incident-response)
13. [Compliance](#13-compliance)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the security requirements and implementation guidelines for all client-server communication in Plunderstorm Mobile. The goal is to ensure that all data transmitted between clients and servers is encrypted, authenticated, and protected from interception or tampering.

### 1.2 Security Objectives

| Objective | Description | Priority |
|-----------|-------------|----------|
| Confidentiality | All data encrypted in transit | Critical |
| Integrity | Data cannot be modified undetected | Critical |
| Authentication | Verify identity of communicating parties | Critical |
| Non-repudiation | Actions can be attributed to specific users | High |
| Availability | Security doesn't impede legitimate use | High |

### 1.3 Threat Model

| Threat | Risk Level | Mitigation |
|--------|------------|------------|
| Man-in-the-Middle (MITM) | High | TLS/DTLS, Certificate Pinning |
| Packet Sniffing | High | Encryption at all layers |
| Replay Attacks | Medium | Nonces, timestamps, sequence numbers |
| Session Hijacking | Medium | Secure tokens, short expiry |
| Data Tampering | High | Message authentication codes |
| Credential Theft | High | Secure storage, no plaintext |

### 1.4 Communication Channels

| Channel | Protocol | Port | Purpose |
|---------|----------|------|---------|
| REST API | HTTPS (TLS 1.3) | 443 | Authentication, profiles, store |
| WebSocket | WSS (TLS 1.3) | 443 | Real-time notifications |
| Game Server | DTLS 1.2/1.3 over UDP | 7777-7877 | Gameplay traffic |
| Voice Chat | DTLS-SRTP | Dynamic | Voice communication |

---

## 2. Security Architecture Overview

### 2.1 Defense in Depth

```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT DEVICE                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ Secure Keychain │  │ Certificate     │                   │
│  │ (Keys, Tokens)  │  │ Pin Store       │                   │
│  └────────┬────────┘  └────────┬────────┘                   │
│           │                    │                             │
│  ┌────────▼────────────────────▼────────┐                   │
│  │         TLS/DTLS Client Layer        │                   │
│  │  • Certificate Validation            │                   │
│  │  • Pin Verification                  │                   │
│  │  • Session Management                │                   │
│  └────────┬─────────────────────────────┘                   │
└───────────┼─────────────────────────────────────────────────┘
            │ Encrypted Traffic
            ▼
┌───────────────────────────────────────────────────────────┐
│                    NETWORK (INTERNET)                      │
│              All traffic encrypted (TLS/DTLS)              │
└───────────┬───────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────┐
│                      SERVER INFRASTRUCTURE                   │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Load Balancer│  │ API Gateway  │  │ Game Servers │       │
│  │ (TLS Term)   │  │ (TLS Term)   │  │ (DTLS)       │       │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                 │                 │                │
│  ┌──────▼─────────────────▼─────────────────▼──────┐        │
│  │              Internal Service Mesh               │        │
│  │           (mTLS between services)                │        │
│  └──────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Trust Boundaries

| Boundary | Trust Level | Security Controls |
|----------|-------------|-------------------|
| Client Device | Untrusted | Input validation, secure storage |
| Public Network | Untrusted | Encryption, authentication |
| Edge (LB/CDN) | Semi-trusted | TLS termination, WAF |
| Application Layer | Trusted | mTLS, service auth |
| Database Layer | Trusted | Encryption at rest, access control |

---

## 3. Transport Layer Security (TLS)

### 3.1 TLS Configuration

#### Minimum Requirements

| Parameter | Requirement |
|-----------|-------------|
| TLS Version | TLS 1.3 (preferred), TLS 1.2 (minimum) |
| Certificate Key Size | RSA 2048+ or ECDSA P-256+ |
| Certificate Validity | Maximum 1 year |
| OCSP Stapling | Required |
| HSTS | Required (max-age=31536000) |

#### Supported Cipher Suites (TLS 1.3)

```
TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_CHACHA20_POLY1305_SHA256
```

#### Supported Cipher Suites (TLS 1.2 - Fallback Only)

```
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
```

#### Disabled Features

| Feature | Reason |
|---------|--------|
| SSL 2.0/3.0 | Known vulnerabilities |
| TLS 1.0/1.1 | Deprecated, weak |
| RC4 | Weak cipher |
| 3DES | Weak cipher |
| MD5 | Weak hash |
| SHA-1 | Weak hash |
| Export ciphers | Weak |
| NULL ciphers | No encryption |
| Compression | CRIME attack |
| Renegotiation | DoS vector |

### 3.2 API Endpoint Configuration

#### Production Endpoints

| Endpoint | Domain | TLS |
|----------|--------|-----|
| Auth API | `auth.plunderstorm.com` | TLS 1.3 |
| Game API | `api.plunderstorm.com` | TLS 1.3 |
| Store API | `store.plunderstorm.com` | TLS 1.3 |
| WebSocket | `ws.plunderstorm.com` | WSS (TLS 1.3) |
| CDN | `cdn.plunderstorm.com` | TLS 1.3 |

#### Server Configuration Example (nginx)

```nginx
server {
    listen 443 ssl http2;
    server_name api.plunderstorm.com;

    # TLS Configuration
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'TLS_AES_256_GCM_SHA384:TLS_AES_128_GCM_SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';

    # Certificate
    ssl_certificate /etc/ssl/certs/plunderstorm.crt;
    ssl_certificate_key /etc/ssl/private/plunderstorm.key;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/ssl/certs/chain.crt;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;

    # Session Configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
}
```

### 3.3 Client TLS Implementation

#### iOS (Swift)

```swift
// URLSession Configuration with TLS requirements
let config = URLSessionConfiguration.default
config.tlsMinimumSupportedProtocolVersion = .TLSv12
config.tlsMaximumSupportedProtocolVersion = .TLSv13

// Certificate validation delegate
class SecureSessionDelegate: NSObject, URLSessionDelegate {
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // Implement certificate pinning here
        guard let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        // Validate and pin certificate
        if validateCertificate(serverTrust) {
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

#### Android (Kotlin)

```kotlin
// OkHttp Configuration with TLS requirements
val spec = ConnectionSpec.Builder(ConnectionSpec.MODERN_TLS)
    .tlsVersions(TlsVersion.TLS_1_3, TlsVersion.TLS_1_2)
    .cipherSuites(
        CipherSuite.TLS_AES_256_GCM_SHA384,
        CipherSuite.TLS_AES_128_GCM_SHA256,
        CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
        CipherSuite.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    )
    .build()

val client = OkHttpClient.Builder()
    .connectionSpecs(listOf(spec))
    .certificatePinner(certificatePinner)
    .build()
```

---

## 4. Game Traffic Encryption (DTLS)

### 4.1 DTLS Overview

Game traffic uses UDP for low latency, secured with DTLS (Datagram TLS).

| Parameter | Value |
|-----------|-------|
| Protocol | DTLS 1.2 (minimum), DTLS 1.3 (preferred) |
| Handshake | Full handshake on connect, abbreviated on reconnect |
| Anti-Replay | 64-bit sliding window |
| MTU | 1200 bytes (safe for most networks) |
| Retransmission | Exponential backoff (1s, 2s, 4s, 8s, 16s) |

### 4.2 DTLS Configuration

#### Cipher Suites (DTLS 1.3)

```
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
```

#### Cipher Suites (DTLS 1.2)

```
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
```

### 4.3 Game Packet Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    DTLS Record Header                        │
├─────────────────────────────────────────────────────────────┤
│ Content Type (1) │ Version (2) │ Epoch (2) │ Seq Num (6)    │
├─────────────────────────────────────────────────────────────┤
│                    Length (2 bytes)                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                    Encrypted Payload                         │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Game Packet Header                                     │  │
│  │ • Packet Type (1 byte)                                │  │
│  │ • Sequence Number (4 bytes)                           │  │
│  │ • Timestamp (4 bytes)                                 │  │
│  │ • Flags (1 byte)                                      │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ Game Packet Data                                       │  │
│  │ • Player inputs, positions, actions                   │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│                    Authentication Tag (16 bytes)             │
└─────────────────────────────────────────────────────────────┘
```

### 4.4 Session Establishment

```
Client                                              Server
   │                                                   │
   │─────── ClientHello (with DTLS cookie) ──────────▶│
   │                                                   │
   │◀────── HelloVerifyRequest (cookie) ──────────────│
   │                                                   │
   │─────── ClientHello (with cookie) ────────────────▶│
   │                                                   │
   │◀────── ServerHello, Certificate, Done ───────────│
   │                                                   │
   │─────── ClientKeyExchange, ChangeCipher, Finished─▶│
   │                                                   │
   │◀────── ChangeCipherSpec, Finished ───────────────│
   │                                                   │
   │═══════ Encrypted Game Traffic ═══════════════════│
   │                                                   │
```

### 4.5 Anti-Replay Protection

```
Implementation:
1. Each DTLS record has a 48-bit sequence number
2. Server maintains a 64-bit sliding window
3. Records outside window or already seen are dropped
4. Window advances as new records are accepted

Window Example:
┌─────────────────────────────────────────────────┐
│  Received: ████░███░████████████████████████░░░ │
│  Window:   [───────────────64 bits────────────] │
│            ▲                                  ▲  │
│         Oldest                            Newest │
└─────────────────────────────────────────────────┘
█ = Received, ░ = Not yet received/Gap
```

### 4.6 Photon Fusion Integration

If using Photon Fusion for networking:

```csharp
// Photon Fusion Security Configuration
public class NetworkSecurityConfig : SimulationBehaviour
{
    public override void Spawned()
    {
        // Enable encryption
        Runner.Config.Simulation.DefaultEncryption = EncryptionMode.Enabled;

        // Set DTLS parameters
        Runner.Config.Network.UseDtls = true;
        Runner.Config.Network.DtlsProtocol = DtlsProtocol.Dtls13;
    }
}
```

---

## 5. Certificate Management

### 5.1 Certificate Hierarchy

```
Root CA (Offline, HSM)
    │
    ├── Intermediate CA (API Services)
    │       │
    │       ├── api.plunderstorm.com
    │       ├── auth.plunderstorm.com
    │       └── store.plunderstorm.com
    │
    └── Intermediate CA (Game Servers)
            │
            ├── game-us-east.plunderstorm.com
            ├── game-eu-west.plunderstorm.com
            └── game-ap-south.plunderstorm.com
```

### 5.2 Certificate Requirements

| Attribute | Requirement |
|-----------|-------------|
| Key Algorithm | ECDSA P-256 (preferred) or RSA 2048 |
| Signature Algorithm | SHA-256 or SHA-384 |
| Validity Period | 90 days (auto-renewed) |
| Revocation | OCSP Stapling required |
| SCT | Certificate Transparency required |
| SAN | Include all relevant domains |

### 5.3 Certificate Lifecycle

| Phase | Timeline | Action |
|-------|----------|--------|
| Issuance | Day 0 | Generate key, obtain certificate |
| Deployment | Day 0-1 | Deploy to servers |
| Monitoring | Ongoing | Check expiry, OCSP status |
| Renewal | Day 60 | Auto-renew via ACME |
| Rotation | Day 75 | Deploy new certificate |
| Expiry | Day 90 | Old certificate expires |

### 5.4 Certificate Automation

```yaml
# Example: cert-manager configuration for Kubernetes
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: plunderstorm-api
spec:
  secretName: plunderstorm-api-tls
  duration: 2160h  # 90 days
  renewBefore: 720h  # 30 days before expiry
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: api.plunderstorm.com
  dnsNames:
    - api.plunderstorm.com
    - auth.plunderstorm.com
    - store.plunderstorm.com
```

---

## 6. Certificate Pinning

### 6.1 Pinning Strategy

| Approach | Pros | Cons | Recommendation |
|----------|------|------|----------------|
| Leaf Certificate Pin | Most secure | Rotation difficult | Not recommended |
| Intermediate CA Pin | Good security | Moderate flexibility | Recommended |
| Public Key Pin | Best balance | More complex | Recommended |
| Backup Pins | Recovery option | Adds complexity | Required |

**Strategy:** Pin the SPKI (Subject Public Key Info) hash of intermediate certificates with backup pins.

### 6.2 Pin Configuration

```json
{
  "pinning_config": {
    "version": 1,
    "domains": [
      {
        "domain": "*.plunderstorm.com",
        "pins": [
          {
            "type": "sha256",
            "hash": "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=",
            "label": "Primary Intermediate CA"
          },
          {
            "type": "sha256",
            "hash": "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=",
            "label": "Backup Intermediate CA"
          }
        ],
        "includeSubdomains": true,
        "reportOnly": false,
        "reportUri": "https://report.plunderstorm.com/pin-failure"
      }
    ],
    "updateUrl": "https://config.plunderstorm.com/pins.json",
    "updateInterval": 86400
  }
}
```

### 6.3 iOS Implementation

```swift
class CertificatePinningManager {
    // Pinned public key hashes (SHA-256 of SPKI)
    private let pinnedHashes: Set<String> = [
        "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=", // Primary
        "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB="  // Backup
    ]

    func validate(serverTrust: SecTrust) -> Bool {
        // Get certificate chain
        guard let certificateChain = SecTrustCopyCertificateChain(serverTrust) as? [SecCertificate] else {
            return false
        }

        // Check each certificate in chain
        for certificate in certificateChain {
            if let publicKey = SecCertificateCopyKey(certificate),
               let publicKeyData = SecKeyCopyExternalRepresentation(publicKey, nil) as Data? {
                let hash = SHA256.hash(data: publicKeyData)
                let hashString = Data(hash).base64EncodedString()

                if pinnedHashes.contains(hashString) {
                    return true
                }
            }
        }

        // No pin matched - report and fail
        reportPinFailure(chain: certificateChain)
        return false
    }

    private func reportPinFailure(chain: [SecCertificate]) {
        // Send failure report to server
        let report = PinFailureReport(
            timestamp: Date(),
            domain: currentDomain,
            certificateChain: chain.map { $0.fingerprint }
        )
        AnalyticsService.shared.report(event: .pinFailure, data: report)
    }
}
```

### 6.4 Android Implementation

```kotlin
class CertificatePinnerFactory {
    companion object {
        fun create(): CertificatePinner {
            return CertificatePinner.Builder()
                // Primary pin
                .add("*.plunderstorm.com",
                     "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
                // Backup pin
                .add("*.plunderstorm.com",
                     "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=")
                .build()
        }
    }
}

// Usage with OkHttp
val client = OkHttpClient.Builder()
    .certificatePinner(CertificatePinnerFactory.create())
    .eventListener(object : EventListener() {
        override fun secureConnectFailed(
            call: Call,
            inetSocketAddress: InetSocketAddress?,
            proxy: Proxy?,
            protocol: Protocol?,
            exception: SSLException?
        ) {
            // Report pin failure
            if (exception is SSLPeerUnverifiedException) {
                reportPinFailure(call.request().url, exception)
            }
        }
    })
    .build()
```

### 6.5 Pin Update Mechanism

```
Client App                                    Config Server
    │                                              │
    │──── GET /pins.json (signed) ────────────────▶│
    │                                              │
    │◀─── Pin Config (signed, versioned) ─────────│
    │                                              │
    │     Verify signature                         │
    │     Compare version                          │
    │     Update local pins if newer               │
    │                                              │
```

**Important:** Always include backup pins that can be deployed server-side to recover from pin failures.

---

## 7. Client-Side Key Storage

### 7.1 Storage Requirements

| Data Type | Storage Location | Protection |
|-----------|------------------|------------|
| Auth Tokens | Secure Enclave/Keystore | Biometric/PIN optional |
| Refresh Tokens | Secure Enclave/Keystore | Required |
| Session Keys | Memory only | Cleared on background |
| User Credentials | Never stored | N/A |
| API Keys | Obfuscated in binary | Tamper detection |

### 7.2 iOS Keychain Implementation

```swift
class SecureStorage {
    private let service = "com.plunderstorm.mobile"

    func saveToken(_ token: String, for key: String) throws {
        let data = token.data(using: .utf8)!

        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]

        // Delete existing item
        SecItemDelete(query as CFDictionary)

        // Add new item
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw SecureStorageError.saveFailed(status)
        }
    }

    func loadToken(for key: String) throws -> String? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]

        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)

        guard status == errSecSuccess,
              let data = result as? Data,
              let token = String(data: data, encoding: .utf8) else {
            return nil
        }

        return token
    }

    func deleteToken(for key: String) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        SecItemDelete(query as CFDictionary)
    }
}
```

### 7.3 Android Keystore Implementation

```kotlin
class SecureStorage(private val context: Context) {
    private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply { load(null) }
    private val sharedPrefs = EncryptedSharedPreferences.create(
        context,
        "plunderstorm_secure_prefs",
        getMasterKey(),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    private fun getMasterKey(): MasterKey {
        return MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .setUserAuthenticationRequired(false)
            .build()
    }

    fun saveToken(key: String, token: String) {
        sharedPrefs.edit().putString(key, token).apply()
    }

    fun loadToken(key: String): String? {
        return sharedPrefs.getString(key, null)
    }

    fun deleteToken(key: String) {
        sharedPrefs.edit().remove(key).apply()
    }

    fun clearAll() {
        sharedPrefs.edit().clear().apply()
    }
}
```

### 7.4 Token Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│                     TOKEN LIFECYCLE                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Login ──▶ Receive Tokens ──▶ Store Securely                │
│                │                    │                        │
│                ▼                    ▼                        │
│         Access Token          Refresh Token                  │
│         (15 min TTL)          (30 day TTL)                  │
│                │                    │                        │
│                ▼                    │                        │
│         Use for API ◀──────────────┤                        │
│                │                    │                        │
│         Expired? ──Yes──▶ Use Refresh Token                 │
│                │                    │                        │
│                No                   ▼                        │
│                │            Get New Access Token             │
│                │                    │                        │
│                ▼                    │                        │
│         Continue ◀─────────────────┘                        │
│                                                              │
│         Logout ──▶ Clear All Tokens ──▶ Revoke Server-Side  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Data Protection in Transit

### 8.1 Sensitive Data Classification

| Classification | Examples | Encryption Required |
|----------------|----------|---------------------|
| Critical | Passwords, payment info | TLS + Application encryption |
| High | Auth tokens, player ID | TLS mandatory |
| Medium | Gameplay data, stats | TLS mandatory |
| Low | Public game state | TLS recommended |

### 8.2 Request/Response Security

#### Request Headers (Required)

```http
POST /api/v1/match/join HTTP/1.1
Host: api.plunderstorm.com
Authorization: Bearer <jwt_token>
X-Request-ID: <uuid>
X-Client-Version: 1.0.0
X-Platform: iOS
X-Device-ID: <hashed_device_id>
Content-Type: application/json
```

#### Response Headers (Required)

```http
HTTP/1.1 200 OK
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'none'
Cache-Control: no-store
```

### 8.3 Payload Encryption (Additional Layer)

For highly sensitive operations, add application-level encryption:

```json
{
  "encrypted_payload": "<base64_encrypted_data>",
  "key_id": "key_2024_01",
  "iv": "<base64_iv>",
  "tag": "<base64_auth_tag>"
}
```

Implementation:
```swift
class PayloadEncryption {
    func encrypt(_ data: Data) throws -> EncryptedPayload {
        let key = try KeyManager.shared.getCurrentKey()
        let iv = AES.GCM.Nonce()

        let sealedBox = try AES.GCM.seal(data, using: key, nonce: iv)

        return EncryptedPayload(
            encryptedData: sealedBox.ciphertext.base64EncodedString(),
            keyId: KeyManager.shared.currentKeyId,
            iv: Data(iv).base64EncodedString(),
            tag: sealedBox.tag.base64EncodedString()
        )
    }
}
```

---

## 9. Logging Security

### 9.1 Data Never Logged

| Data Type | Reason | Alternative |
|-----------|--------|-------------|
| Passwords | Credential exposure | Log "password_provided: true/false" |
| Auth Tokens | Session hijacking | Log token hash (first 8 chars) |
| Credit Card | PCI compliance | Log last 4 digits only |
| Player IP | Privacy (GDPR) | Log anonymized/hashed IP |
| Device ID | Privacy | Log hashed device ID |
| Location | Privacy | Log region only (country) |

### 9.2 Logging Guidelines

#### Allowed Logging

```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "event": "user_login",
  "user_id_hash": "abc123...",
  "success": true,
  "region": "US",
  "client_version": "1.0.0"
}
```

#### Prohibited Logging

```json
// NEVER LOG THIS:
{
  "password": "user_password",
  "auth_token": "eyJhbGciOiJIUzI1NiIs...",
  "ip_address": "192.168.1.100",
  "device_id": "ABC123-DEF456-...",
  "email": "user@example.com"
}
```

### 9.3 Log Sanitization

```swift
class LogSanitizer {
    private static let sensitivePatterns = [
        "password": #"\"password\"\s*:\s*\"[^\"]+\""#,
        "token": #"Bearer\s+[A-Za-z0-9\-_]+\.[A-Za-z0-9\-_]+\.[A-Za-z0-9\-_]+"#,
        "email": #"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"#
    ]

    static func sanitize(_ message: String) -> String {
        var result = message
        for (name, pattern) in sensitivePatterns {
            if let regex = try? NSRegularExpression(pattern: pattern, options: []) {
                result = regex.stringByReplacingMatches(
                    in: result,
                    range: NSRange(result.startIndex..., in: result),
                    withTemplate: "[\(name)_redacted]"
                )
            }
        }
        return result
    }
}
```

---

## 10. Implementation Guidelines

### 10.1 Development Checklist

#### Before Release

- [ ] All endpoints use HTTPS
- [ ] TLS 1.2+ enforced (TLS 1.3 preferred)
- [ ] Certificate pinning implemented
- [ ] Secure storage used for all credentials
- [ ] No sensitive data in logs
- [ ] DTLS enabled for game traffic
- [ ] Security headers configured
- [ ] HSTS enabled

#### Code Review Security Checks

- [ ] No hardcoded credentials
- [ ] No disabled certificate validation
- [ ] No logging of sensitive data
- [ ] Proper error handling (no stack traces to client)
- [ ] Input validation on all endpoints
- [ ] Output encoding applied

### 10.2 Testing Environments

| Environment | TLS Config | Pinning | Notes |
|-------------|------------|---------|-------|
| Development | TLS 1.2+ | Disabled | Self-signed OK |
| Staging | TLS 1.3 | Enabled (test pins) | Real certificates |
| Production | TLS 1.3 | Enabled (prod pins) | Production certs |

### 10.3 Error Handling

```swift
enum SecurityError: Error {
    case tlsHandshakeFailed
    case certificatePinningFailed
    case tokenExpired
    case tokenInvalid
    case encryptionFailed
    case keyNotFound
}

func handleSecurityError(_ error: SecurityError) {
    switch error {
    case .certificatePinningFailed:
        // Log event, clear credentials, force re-auth
        Analytics.log(event: .securityEvent, params: ["type": "pin_failure"])
        SecureStorage.shared.clearAll()
        forceReAuthentication()

    case .tokenExpired:
        // Attempt refresh
        attemptTokenRefresh()

    case .tlsHandshakeFailed:
        // Network error, retry with backoff
        scheduleRetry(delay: exponentialBackoff())

    default:
        // Generic security error
        showError(message: "Security error. Please restart the app.")
    }
}
```

---

## 11. Security Testing

### 11.1 Test Categories

| Category | Tools | Frequency |
|----------|-------|-----------|
| TLS Configuration | SSL Labs, testssl.sh | Weekly |
| Certificate Pinning | Manual test, MITMProxy | Per release |
| Penetration Testing | External firm | Quarterly |
| Code Security Scan | SonarQube, Checkmarx | Per commit |
| Dependency Scan | Snyk, Dependabot | Daily |

### 11.2 TLS Verification Script

```bash
#!/bin/bash
# Test TLS configuration

DOMAIN="api.plunderstorm.com"

echo "Testing TLS configuration for $DOMAIN"

# Check TLS versions
echo "=== TLS Versions ==="
nmap --script ssl-enum-ciphers -p 443 $DOMAIN

# Check certificate
echo "=== Certificate Info ==="
echo | openssl s_client -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -dates -subject -issuer

# Check for weak ciphers
echo "=== Weak Cipher Check ==="
testssl.sh --vulnerable $DOMAIN

# Check HSTS
echo "=== HSTS Header ==="
curl -sI https://$DOMAIN | grep -i strict-transport-security
```

### 11.3 Certificate Pinning Test

```swift
// Test certificate pinning is working
func testCertificatePinning() {
    // Test 1: Valid certificate should succeed
    let validRequest = URLRequest(url: URL(string: "https://api.plunderstorm.com/health")!)
    URLSession.shared.dataTask(with: validRequest) { _, response, error in
        XCTAssertNil(error, "Valid certificate should not error")
        XCTAssertEqual((response as? HTTPURLResponse)?.statusCode, 200)
    }.resume()

    // Test 2: Intercepted request should fail
    // (Requires MITM proxy in test environment)
    // Verify app rejects the proxy's certificate
}
```

---

## 12. Incident Response

### 12.1 Security Incident Types

| Incident | Severity | Response Time |
|----------|----------|---------------|
| Certificate compromise | Critical | Immediate |
| Pin bypass detected | Critical | < 1 hour |
| Token breach | High | < 4 hours |
| TLS vulnerability | High | < 24 hours |
| Failed penetration test | Medium | < 1 week |

### 12.2 Certificate Compromise Response

```
1. IMMEDIATE (0-1 hour)
   □ Revoke compromised certificate
   □ Deploy backup certificate
   □ Notify security team
   □ Begin investigation

2. SHORT-TERM (1-24 hours)
   □ Issue new certificate
   □ Update certificate pins (if needed)
   □ Push client update (if pins changed)
   □ Audit access logs

3. POST-INCIDENT (24-72 hours)
   □ Complete root cause analysis
   □ Update security procedures
   □ Document lessons learned
   □ Review with stakeholders
```

### 12.3 Emergency Contacts

| Role | Contact | Responsibility |
|------|---------|----------------|
| Security Lead | [TBD] | Incident commander |
| DevOps Lead | [TBD] | Certificate rotation |
| Backend Lead | [TBD] | API security |
| Mobile Lead | [TBD] | Client-side security |

---

## 13. Compliance

### 13.1 Regulatory Requirements

| Regulation | Requirement | Our Compliance |
|------------|-------------|----------------|
| GDPR | Encryption of personal data | TLS for all data in transit |
| PCI-DSS | Strong cryptography | TLS 1.2+, no weak ciphers |
| COPPA | Protection of children's data | Full encryption |
| App Store | ATS compliance (iOS) | TLS 1.2+, ATS compliant |
| Play Store | Security best practices | Certificate pinning |

### 13.2 Audit Requirements

| Audit Type | Frequency | Scope |
|------------|-----------|-------|
| Internal security review | Monthly | Code, configuration |
| External penetration test | Quarterly | Full application |
| Compliance audit | Annually | All security controls |
| Certificate audit | Monthly | All certificates |

### 13.3 Documentation Requirements

- Maintain this document with all changes
- Log all security-related changes
- Keep incident reports for 3 years
- Annual security training for all developers

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Security Lead | _____________ | _____________ | ___/___/___ |
| Backend Lead | _____________ | _____________ | ___/___/___ |
| CTO | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial security specification |

---

*This document contains security-sensitive information. Handle according to company security policies.*
