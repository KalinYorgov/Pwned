# Authentication System Specification - Plunderstorm Mobile

**Document ID:** BACK-010
**Version:** 1.0
**Last Updated:** 2025-11-30
**Status:** Approved

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Authentication Methods](#2-authentication-methods)
3. [Guest Authentication](#3-guest-authentication)
4. [Email/Password Authentication](#4-emailpassword-authentication)
5. [Social Login Integration](#5-social-login-integration)
6. [Platform Authentication](#6-platform-authentication)
7. [Token Management](#7-token-management)
8. [Account Linking](#8-account-linking)
9. [Account Recovery](#9-account-recovery)
10. [Multi-Device Support](#10-multi-device-support)
11. [Security Requirements](#11-security-requirements)
12. [API Specification](#12-api-specification)
13. [Client Implementation](#13-client-implementation)
14. [Compliance](#14-compliance)

---

## 1. Executive Summary

### 1.1 Purpose

This document specifies the authentication system for Plunderstorm Mobile, enabling secure player identification across multiple authentication methods while providing a frictionless onboarding experience.

### 1.2 Design Goals

| Goal | Description | Priority |
|------|-------------|----------|
| Frictionless Entry | Guest play with zero friction | Critical |
| Security | Protect player accounts and data | Critical |
| Flexibility | Multiple auth methods supported | High |
| Persistence | Progress preserved across devices | High |
| Platform Compliance | Meet App Store/Play Store requirements | Critical |

### 1.3 Authentication Methods Overview

| Method | Friction | Security | Features | Target Users |
|--------|----------|----------|----------|--------------|
| Guest | None | Low | Limited | New/Casual |
| Email/Password | Medium | High | Full | Core players |
| Google Sign-In | Low | High | Full | Android users |
| Apple Sign-In | Low | High | Full | iOS users |
| Facebook Login | Low | Medium | Full | Social players |
| Game Center | Low | High | Full | iOS gamers |
| Google Play Games | Low | High | Full | Android gamers |

### 1.4 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT APPLICATION                      │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│  │  Guest    │  │  Email/   │  │  Social   │               │
│  │  Auth     │  │  Password │  │  Providers│               │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘               │
│        │              │              │                      │
│        └──────────────┼──────────────┘                      │
│                       │                                     │
│               ┌───────▼───────┐                            │
│               │ Auth Manager  │                            │
│               └───────┬───────┘                            │
└───────────────────────┼─────────────────────────────────────┘
                        │ HTTPS/TLS
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                      AUTH SERVICE                            │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐   │
│  │ Auth Gateway  │  │ Token Service │  │ Provider      │   │
│  │               │◀─│               │◀─│ Validators    │   │
│  └───────┬───────┘  └───────────────┘  └───────────────┘   │
│          │                                                  │
│  ┌───────▼───────┐  ┌───────────────┐  ┌───────────────┐   │
│  │ Account       │  │ Session       │  │ Rate          │   │
│  │ Service       │◀─│ Manager       │◀─│ Limiter       │   │
│  └───────┬───────┘  └───────────────┘  └───────────────┘   │
│          │                                                  │
│  ┌───────▼───────────────────────────────────────────────┐ │
│  │                    Database Layer                      │ │
│  │  • Player Accounts  • Sessions  • Linked Providers    │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Authentication Methods

### 2.1 Method Comparison

| Feature | Guest | Email | Google | Apple | Facebook | Game Center | Play Games |
|---------|-------|-------|--------|-------|----------|-------------|------------|
| Instant Play | Yes | No | No | No | No | No | No |
| IAP Enabled | Yes* | Yes | Yes | Yes | Yes | Yes | Yes |
| Cross-Device | No | Yes | Yes | Yes | Yes | Yes | Yes |
| Friend Features | Limited | Yes | Yes | Yes | Yes | Yes | Yes |
| Account Recovery | No | Yes | Yes | Yes | Yes | Yes | Yes |
| Leaderboards | Limited | Yes | Yes | Yes | Yes | Yes | Yes |

*Guest IAP requires account upgrade for purchase protection

### 2.2 Authentication Flow Priority

1. **First Launch**: Offer Guest play prominently
2. **Before IAP**: Require account linking
3. **Social Features**: Prompt for account upgrade
4. **Periodically**: Gentle reminders to link account

### 2.3 Method Selection Matrix

| Platform | Primary | Secondary | Tertiary |
|----------|---------|-----------|----------|
| iOS | Apple Sign-In | Guest | Email |
| Android | Google Sign-In | Guest | Email |
| Cross-Platform | Email | Social (any) | Guest |

---

## 3. Guest Authentication

### 3.1 Overview

Guest authentication allows immediate gameplay without registration, using a device-generated identifier.

### 3.2 Device ID Generation

```
Device ID = SHA256(
    platform_id +
    vendor_id (iOS) OR android_id (Android) +
    installation_timestamp +
    random_salt
)
```

**Storage:**
- iOS: Keychain (kSecAttrAccessibleAfterFirstUnlock)
- Android: EncryptedSharedPreferences

### 3.3 Guest Account Creation Flow

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Client    │      │ Auth Service│      │  Database   │
└──────┬──────┘      └──────┬──────┘      └──────┬──────┘
       │                    │                    │
       │ POST /auth/guest   │                    │
       │ {device_id}        │                    │
       │───────────────────▶│                    │
       │                    │                    │
       │                    │ Check existing     │
       │                    │ guest account      │
       │                    │───────────────────▶│
       │                    │                    │
       │                    │◀───────────────────│
       │                    │                    │
       │                    │ Create/retrieve    │
       │                    │ guest account      │
       │                    │───────────────────▶│
       │                    │                    │
       │                    │◀───────────────────│
       │                    │                    │
       │ {access_token,     │                    │
       │  refresh_token,    │                    │
       │  player_id,        │                    │
       │  is_guest: true}   │                    │
       │◀───────────────────│                    │
       │                    │                    │
```

### 3.4 Guest Account Limitations

| Feature | Guest | Full Account |
|---------|-------|--------------|
| Play Matches | Yes | Yes |
| Earn Rewards | Yes | Yes |
| Make Purchases | Yes* | Yes |
| Friend List | View Only | Full Access |
| Leaderboards | Anonymous | Named |
| Chat | Receive Only | Full |
| Cross-Device | No | Yes |

*Purchases require account linking before confirmation

### 3.5 Guest Data Persistence

- Guest progress stored server-side
- Device ID links to progress
- App reinstall on same device recovers progress
- Factory reset loses progress (prompt to link)

### 3.6 Implementation

```swift
// iOS Guest Authentication
class GuestAuthProvider {
    func authenticate() async throws -> AuthResult {
        let deviceId = DeviceIdentifier.getOrCreate()

        let request = GuestAuthRequest(
            deviceId: deviceId,
            platform: "ios",
            appVersion: Bundle.main.appVersion
        )

        let response = try await authService.authenticateGuest(request)

        // Store tokens securely
        try SecureStorage.shared.save(
            accessToken: response.accessToken,
            refreshToken: response.refreshToken
        )

        return AuthResult(
            playerId: response.playerId,
            isGuest: true,
            displayName: response.displayName
        )
    }
}
```

---

## 4. Email/Password Authentication

### 4.1 Registration Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    REGISTRATION FLOW                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. User enters email + password                              │
│          │                                                    │
│          ▼                                                    │
│  2. Client validates format                                   │
│     • Email format valid                                      │
│     • Password meets requirements                             │
│          │                                                    │
│          ▼                                                    │
│  3. Server validates                                          │
│     • Email not already registered                            │
│     • Password strength check                                 │
│          │                                                    │
│          ▼                                                    │
│  4. Create account (unverified)                               │
│          │                                                    │
│          ▼                                                    │
│  5. Send verification email                                   │
│          │                                                    │
│          ▼                                                    │
│  6. User clicks verification link                             │
│          │                                                    │
│          ▼                                                    │
│  7. Account verified → Full access                            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Password Requirements

| Requirement | Specification |
|-------------|---------------|
| Minimum Length | 8 characters |
| Maximum Length | 128 characters |
| Character Types | At least 3 of: uppercase, lowercase, number, special |
| Forbidden | Common passwords (top 10,000 list) |
| Forbidden | Contains email or username |
| Encoding | UTF-8 supported |

### 4.3 Password Storage

```
Password Hash = Argon2id(
    password,
    salt = random(16 bytes),
    time_cost = 3,
    memory_cost = 65536 KB,
    parallelism = 4,
    hash_length = 32 bytes
)

Stored Format: $argon2id$v=19$m=65536,t=3,p=4$<salt>$<hash>
```

### 4.4 Login Flow

```
POST /auth/login
{
    "email": "player@example.com",
    "password": "SecurePassword123!",
    "device_info": {
        "device_id": "abc123...",
        "platform": "ios",
        "os_version": "17.0",
        "app_version": "1.0.0"
    }
}

Response (Success):
{
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJl...",
    "expires_in": 900,
    "player_id": "player_123456",
    "display_name": "PirateCaptain",
    "email_verified": true
}

Response (Failure):
{
    "error": "invalid_credentials",
    "message": "Email or password is incorrect",
    "remaining_attempts": 4
}
```

### 4.5 Rate Limiting

| Action | Limit | Window | Lockout |
|--------|-------|--------|---------|
| Login Attempts | 5 | 15 minutes | 30 min lockout |
| Registration | 3 | 1 hour | 24 hour lockout |
| Password Reset | 3 | 1 hour | 1 hour cooldown |
| Verification Email | 3 | 1 hour | 1 hour cooldown |

### 4.6 Email Verification

```
Verification Token = Base64URL(
    HMAC-SHA256(
        email + account_id + timestamp,
        server_secret
    )
)

Verification Link:
https://auth.plunderstorm.com/verify?
    token=<verification_token>&
    email=<encoded_email>

Token Validity: 24 hours
```

---

## 5. Social Login Integration

### 5.1 Google Sign-In

#### Configuration

| Platform | Client ID Type |
|----------|---------------|
| iOS | iOS Client ID |
| Android | Web Client ID (for backend) |
| Backend | Web Client ID |

#### Flow

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│ Client  │     │ Google  │     │ Backend │     │   DB    │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │
     │ Sign In UI    │               │               │
     │──────────────▶│               │               │
     │               │               │               │
     │ ID Token      │               │               │
     │◀──────────────│               │               │
     │               │               │               │
     │ POST /auth/google              │               │
     │ {id_token}    │               │               │
     │───────────────────────────────▶│               │
     │               │               │               │
     │               │ Verify Token  │               │
     │               │◀──────────────│               │
     │               │               │               │
     │               │ Token Valid   │               │
     │               │──────────────▶│               │
     │               │               │               │
     │               │               │ Find/Create   │
     │               │               │ Account       │
     │               │               │──────────────▶│
     │               │               │               │
     │               │               │◀──────────────│
     │               │               │               │
     │ {access_token, player_id}     │               │
     │◀──────────────────────────────│               │
     │               │               │               │
```

#### Backend Verification

```python
from google.oauth2 import id_token
from google.auth.transport import requests

def verify_google_token(token: str) -> dict:
    try:
        idinfo = id_token.verify_oauth2_token(
            token,
            requests.Request(),
            GOOGLE_CLIENT_ID
        )

        if idinfo['iss'] not in ['accounts.google.com', 'https://accounts.google.com']:
            raise ValueError('Invalid issuer')

        return {
            'provider': 'google',
            'provider_id': idinfo['sub'],
            'email': idinfo.get('email'),
            'email_verified': idinfo.get('email_verified', False),
            'name': idinfo.get('name'),
            'picture': idinfo.get('picture')
        }
    except ValueError as e:
        raise AuthenticationError(f'Invalid Google token: {e}')
```

### 5.2 Apple Sign-In

#### Requirements (iOS)

- Required for apps with any third-party login (App Store requirement)
- Must be displayed as prominently as other login options

#### Flow

```swift
// iOS Implementation
class AppleAuthProvider: ASAuthorizationControllerDelegate {
    func signIn() {
        let provider = ASAuthorizationAppleIDProvider()
        let request = provider.createRequest()
        request.requestedScopes = [.fullName, .email]

        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.performRequests()
    }

    func authorizationController(
        controller: ASAuthorizationController,
        didCompleteWithAuthorization authorization: ASAuthorization
    ) {
        guard let credential = authorization.credential as? ASAuthorizationAppleIDCredential,
              let identityToken = credential.identityToken,
              let tokenString = String(data: identityToken, encoding: .utf8) else {
            return
        }

        // Send to backend
        Task {
            let result = try await authService.authenticateApple(
                identityToken: tokenString,
                authorizationCode: String(data: credential.authorizationCode!, encoding: .utf8)!,
                fullName: credential.fullName,
                email: credential.email
            )
            // Handle result
        }
    }
}
```

#### Backend Verification

```python
import jwt
from jwt import PyJWKClient

APPLE_JWKS_URL = "https://appleid.apple.com/auth/keys"

def verify_apple_token(identity_token: str) -> dict:
    # Get Apple's public keys
    jwks_client = PyJWKClient(APPLE_JWKS_URL)
    signing_key = jwks_client.get_signing_key_from_jwt(identity_token)

    # Verify the token
    payload = jwt.decode(
        identity_token,
        signing_key.key,
        algorithms=["RS256"],
        audience=APPLE_CLIENT_ID,
        issuer="https://appleid.apple.com"
    )

    return {
        'provider': 'apple',
        'provider_id': payload['sub'],
        'email': payload.get('email'),
        'email_verified': payload.get('email_verified', False)
    }
```

### 5.3 Facebook Login

#### Permissions Requested

| Permission | Purpose | Required |
|------------|---------|----------|
| public_profile | User identity | Yes |
| email | Contact/recovery | Yes |
| gaming_profile | Gaming features | No |

#### Backend Verification

```python
import requests

def verify_facebook_token(access_token: str) -> dict:
    # Verify token with Facebook
    debug_url = f"https://graph.facebook.com/debug_token?input_token={access_token}&access_token={APP_TOKEN}"
    debug_response = requests.get(debug_url).json()

    if not debug_response['data']['is_valid']:
        raise AuthenticationError('Invalid Facebook token')

    if debug_response['data']['app_id'] != FACEBOOK_APP_ID:
        raise AuthenticationError('Token not for this app')

    # Get user info
    user_url = f"https://graph.facebook.com/me?fields=id,name,email,picture&access_token={access_token}"
    user_response = requests.get(user_url).json()

    return {
        'provider': 'facebook',
        'provider_id': user_response['id'],
        'email': user_response.get('email'),
        'name': user_response.get('name'),
        'picture': user_response.get('picture', {}).get('data', {}).get('url')
    }
```

---

## 6. Platform Authentication

### 6.1 Game Center (iOS)

#### Flow

```swift
class GameCenterAuthProvider {
    func authenticate() async throws -> AuthResult {
        let localPlayer = GKLocalPlayer.local

        return try await withCheckedThrowingContinuation { continuation in
            localPlayer.authenticateHandler = { viewController, error in
                if let error = error {
                    continuation.resume(throwing: error)
                    return
                }

                if let vc = viewController {
                    // Present Game Center login UI
                    self.presentViewController(vc)
                    return
                }

                if localPlayer.isAuthenticated {
                    Task {
                        do {
                            let result = try await self.verifyWithBackend(localPlayer)
                            continuation.resume(returning: result)
                        } catch {
                            continuation.resume(throwing: error)
                        }
                    }
                }
            }
        }
    }

    private func verifyWithBackend(_ player: GKLocalPlayer) async throws -> AuthResult {
        // Get signature for server verification
        let (publicKeyURL, signature, salt, timestamp) = try await player.fetchItems(
            forIdentityVerificationSignature: ()
        )

        let request = GameCenterAuthRequest(
            playerId: player.gamePlayerID,
            publicKeyURL: publicKeyURL.absoluteString,
            signature: signature.base64EncodedString(),
            salt: salt.base64EncodedString(),
            timestamp: timestamp
        )

        return try await authService.authenticateGameCenter(request)
    }
}
```

### 6.2 Google Play Games (Android)

#### Flow

```kotlin
class PlayGamesAuthProvider(private val activity: Activity) {
    private val signInClient = PlayGames.getGamesSignInClient(activity)

    suspend fun authenticate(): AuthResult {
        // Check if already signed in
        val isAuthenticated = signInClient.isAuthenticated.await()

        if (!isAuthenticated.isAuthenticated) {
            signInClient.signIn().await()
        }

        // Get server auth code
        val serverAuthCode = PlayGames.getGamesSignInClient(activity)
            .requestServerSideAccess(SERVER_CLIENT_ID, false)
            .await()

        // Send to backend
        return authService.authenticatePlayGames(
            PlayGamesAuthRequest(serverAuthCode = serverAuthCode)
        )
    }
}
```

#### Backend Verification

```python
from google.oauth2 import id_token
from google.auth.transport import requests as google_requests

def verify_play_games_token(server_auth_code: str) -> dict:
    # Exchange auth code for tokens
    token_response = requests.post(
        'https://oauth2.googleapis.com/token',
        data={
            'code': server_auth_code,
            'client_id': GOOGLE_CLIENT_ID,
            'client_secret': GOOGLE_CLIENT_SECRET,
            'grant_type': 'authorization_code'
        }
    ).json()

    # Verify ID token
    idinfo = id_token.verify_oauth2_token(
        token_response['id_token'],
        google_requests.Request(),
        GOOGLE_CLIENT_ID
    )

    return {
        'provider': 'play_games',
        'provider_id': idinfo['sub'],
        'email': idinfo.get('email'),
        'name': idinfo.get('name')
    }
```

---

## 7. Token Management

### 7.1 Token Types

| Token | Type | Lifetime | Storage | Purpose |
|-------|------|----------|---------|---------|
| Access Token | JWT | 15 minutes | Memory | API authorization |
| Refresh Token | Opaque | 30 days | Secure storage | Token refresh |
| ID Token | JWT | 1 hour | Memory | User info |

### 7.2 Access Token Structure (JWT)

```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "key_2024_01"
  },
  "payload": {
    "iss": "https://auth.plunderstorm.com",
    "sub": "player_123456",
    "aud": "plunderstorm-api",
    "exp": 1704067200,
    "iat": 1704066300,
    "jti": "unique_token_id",
    "scope": "play profile purchases",
    "device_id": "device_abc123",
    "is_guest": false,
    "email_verified": true
  },
  "signature": "..."
}
```

### 7.3 Token Refresh Flow

```
┌─────────────┐                              ┌─────────────┐
│   Client    │                              │ Auth Server │
└──────┬──────┘                              └──────┬──────┘
       │                                            │
       │ API Request with expired access token      │
       │───────────────────────────────────────────▶│
       │                                            │
       │ 401 Unauthorized (token_expired)           │
       │◀───────────────────────────────────────────│
       │                                            │
       │ POST /auth/refresh                         │
       │ {refresh_token}                            │
       │───────────────────────────────────────────▶│
       │                                            │
       │ Validate refresh token                     │
       │ Check not revoked                          │
       │ Check device match                         │
       │                                            │
       │ {new_access_token, new_refresh_token}      │
       │◀───────────────────────────────────────────│
       │                                            │
       │ Retry original request                     │
       │───────────────────────────────────────────▶│
       │                                            │
```

### 7.4 Token Revocation

**Revocation Triggers:**
- User logout
- Password change
- Security concern
- Admin action
- Device removal

**Implementation:**
```python
class TokenRevocationService:
    def revoke_all_tokens(self, player_id: str, reason: str):
        """Revoke all tokens for a player"""
        # Increment token version to invalidate all existing tokens
        self.db.increment_token_version(player_id)

        # Add all refresh tokens to revocation list
        self.db.revoke_refresh_tokens(player_id)

        # Log security event
        self.audit_log.log(
            event='tokens_revoked',
            player_id=player_id,
            reason=reason
        )

    def revoke_device_tokens(self, player_id: str, device_id: str):
        """Revoke tokens for specific device"""
        self.db.revoke_device_refresh_token(player_id, device_id)
```

### 7.5 Token Security

| Measure | Implementation |
|---------|----------------|
| Signing | RS256 with rotating keys |
| Key Rotation | Every 30 days |
| Audience Validation | Required |
| Issuer Validation | Required |
| Expiration Check | Required |
| Replay Prevention | JTI claim + short expiry |

---

## 8. Account Linking

### 8.1 Linking Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    ACCOUNT LINKING FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Guest Account                    Full Account                │
│  ┌─────────────┐                 ┌─────────────┐             │
│  │ Device ID   │                 │ Email or    │             │
│  │ Progress    │────Merge───────▶│ Social      │             │
│  │ Purchases*  │                 │ All Progress │             │
│  └─────────────┘                 └─────────────┘             │
│                                                               │
│  * Purchases require linking before confirmation              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 8.2 Linking Options

```
POST /auth/link
{
    "current_token": "<guest_access_token>",
    "link_method": "email",
    "credentials": {
        "email": "player@example.com",
        "password": "SecurePassword123!"
    }
}

OR

POST /auth/link
{
    "current_token": "<guest_access_token>",
    "link_method": "google",
    "credentials": {
        "id_token": "<google_id_token>"
    }
}
```

### 8.3 Conflict Resolution

When linking to an existing account:

```
┌─────────────────────────────────────────────────────────────┐
│              ACCOUNT CONFLICT RESOLUTION                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Scenario: Guest links to existing Google account            │
│                                                              │
│  Guest Account (Device)    Existing Account (Google)         │
│  ┌──────────────────┐     ┌──────────────────┐              │
│  │ Level: 15        │     │ Level: 25        │              │
│  │ Gold: 5,000      │     │ Gold: 12,000     │              │
│  │ Wins: 20         │     │ Wins: 50         │              │
│  │ Skins: 3         │     │ Skins: 8         │              │
│  └──────────────────┘     └──────────────────┘              │
│                                                              │
│  Resolution Options:                                         │
│                                                              │
│  1. KEEP EXISTING (Recommended if existing is higher)        │
│     → Guest progress discarded                               │
│     → Guest purchases transferred                            │
│                                                              │
│  2. MERGE (If allowed)                                       │
│     → Higher level kept                                      │
│     → Currencies combined                                    │
│     → All items combined                                     │
│     → Stats merged                                           │
│                                                              │
│  3. KEEP GUEST (Replace existing)                            │
│     → Requires confirmation                                  │
│     → Existing progress archived                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 8.4 Multiple Provider Linking

A single account can link multiple providers:

```json
{
  "player_id": "player_123456",
  "linked_providers": [
    {
      "provider": "email",
      "identifier": "player@example.com",
      "linked_at": "2024-01-15T10:00:00Z"
    },
    {
      "provider": "google",
      "identifier": "google_user_id",
      "linked_at": "2024-01-16T14:30:00Z"
    },
    {
      "provider": "apple",
      "identifier": "apple_user_id",
      "linked_at": "2024-01-20T09:15:00Z"
    }
  ],
  "primary_provider": "email"
}
```

---

## 9. Account Recovery

### 9.1 Recovery Methods

| Method | Availability | Security Level |
|--------|--------------|----------------|
| Email Reset | Email accounts | High |
| Social Re-auth | Social accounts | High |
| Support Ticket | All accounts | Medium (manual) |
| Device Recovery | Guest (same device) | Low |

### 9.2 Password Reset Flow

```
┌─────────────────────────────────────────────────────────────┐
│                   PASSWORD RESET FLOW                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. User requests reset                                      │
│     POST /auth/password/reset-request                        │
│     {email: "player@example.com"}                            │
│          │                                                   │
│          ▼                                                   │
│  2. Server generates reset token                             │
│     Token = random(32 bytes)                                │
│     Expiry = 1 hour                                         │
│          │                                                   │
│          ▼                                                   │
│  3. Email sent with reset link                               │
│     https://auth.plunderstorm.com/reset?token=xxx            │
│          │                                                   │
│          ▼                                                   │
│  4. User clicks link, enters new password                    │
│     POST /auth/password/reset                                │
│     {token: "xxx", new_password: "NewSecure123!"}            │
│          │                                                   │
│          ▼                                                   │
│  5. Validate token, update password                          │
│     Revoke all existing sessions                             │
│     Send confirmation email                                  │
│          │                                                   │
│          ▼                                                   │
│  6. User logs in with new password                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 9.3 Social Account Recovery

If user loses access to social account:

1. Attempt re-authentication with same provider
2. Use alternative linked provider
3. If no alternatives, contact support with:
   - Purchase receipts
   - Device information
   - Account details (username, level, etc.)

---

## 10. Multi-Device Support

### 10.1 Concurrent Sessions

| Feature | Behavior |
|---------|----------|
| Login on new device | Allowed (creates new session) |
| Max concurrent devices | 5 |
| Session conflict | Newest session wins |
| Active gameplay | Only one device active in match |

### 10.2 Session Management

```json
// GET /auth/sessions
{
  "sessions": [
    {
      "session_id": "sess_abc123",
      "device_name": "iPhone 15 Pro",
      "platform": "ios",
      "last_active": "2024-01-15T10:30:00Z",
      "is_current": true,
      "location": "San Francisco, US"
    },
    {
      "session_id": "sess_def456",
      "device_name": "Pixel 8",
      "platform": "android",
      "last_active": "2024-01-14T18:00:00Z",
      "is_current": false,
      "location": "New York, US"
    }
  ]
}
```

### 10.3 Device Management

```
// Revoke specific session
DELETE /auth/sessions/{session_id}

// Revoke all other sessions
POST /auth/sessions/revoke-others

// Revoke all sessions (logout everywhere)
POST /auth/sessions/revoke-all
```

---

## 11. Security Requirements

### 11.1 Brute Force Protection

| Control | Implementation |
|---------|----------------|
| Login Rate Limit | 5 attempts / 15 min |
| Account Lockout | 30 min after 5 failures |
| CAPTCHA | After 3 failures |
| IP Blocking | After 20 failures / hour |
| Notification | Email on suspicious activity |

### 11.2 Session Security

| Measure | Implementation |
|---------|----------------|
| Session Binding | Bind to device ID |
| Token Rotation | Refresh token rotated on use |
| Secure Storage | Keychain/Keystore only |
| Transport | TLS 1.3 only |
| Logout | Server-side revocation |

### 11.3 Audit Logging

```json
{
  "event_type": "auth.login.success",
  "timestamp": "2024-01-15T10:30:00Z",
  "player_id": "player_123456",
  "auth_method": "google",
  "device_id": "device_abc",
  "ip_address_hash": "sha256:...",
  "user_agent": "Plunderstorm/1.0 iOS/17.0",
  "location_country": "US",
  "session_id": "sess_abc123"
}
```

**Logged Events:**
- Login success/failure
- Registration
- Password change/reset
- Account linking
- Token refresh
- Session revocation
- Suspicious activity

---

## 12. API Specification

### 12.1 Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/auth/guest` | POST | Guest authentication |
| `/auth/register` | POST | Email registration |
| `/auth/login` | POST | Email login |
| `/auth/google` | POST | Google authentication |
| `/auth/apple` | POST | Apple authentication |
| `/auth/facebook` | POST | Facebook authentication |
| `/auth/gamecenter` | POST | Game Center authentication |
| `/auth/playgames` | POST | Play Games authentication |
| `/auth/refresh` | POST | Refresh tokens |
| `/auth/logout` | POST | Logout (revoke tokens) |
| `/auth/link` | POST | Link provider to account |
| `/auth/unlink` | POST | Unlink provider |
| `/auth/password/reset-request` | POST | Request password reset |
| `/auth/password/reset` | POST | Complete password reset |
| `/auth/sessions` | GET | List active sessions |
| `/auth/sessions/{id}` | DELETE | Revoke session |

### 12.2 Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `invalid_credentials` | 401 | Wrong email/password |
| `account_locked` | 403 | Too many failures |
| `token_expired` | 401 | Access token expired |
| `token_invalid` | 401 | Token malformed/invalid |
| `refresh_token_revoked` | 401 | Refresh token revoked |
| `email_not_verified` | 403 | Email verification required |
| `email_already_registered` | 409 | Email in use |
| `provider_already_linked` | 409 | Provider already linked |
| `rate_limited` | 429 | Too many requests |
| `server_error` | 500 | Internal error |

---

## 13. Client Implementation

### 13.1 Auth Manager (Swift)

```swift
class AuthManager: ObservableObject {
    @Published var isAuthenticated = false
    @Published var isGuest = true
    @Published var playerId: String?

    private let secureStorage = SecureStorage()
    private let authService = AuthService()

    func autoLogin() async throws {
        // Try to restore session
        if let refreshToken = secureStorage.loadRefreshToken() {
            do {
                let tokens = try await authService.refresh(refreshToken)
                updateSession(tokens)
                return
            } catch {
                // Refresh failed, clear and continue to guest
                secureStorage.clear()
            }
        }

        // Fall back to guest auth
        try await guestLogin()
    }

    func guestLogin() async throws {
        let result = try await GuestAuthProvider().authenticate()
        updateSession(result)
    }

    func loginWithGoogle() async throws {
        let result = try await GoogleAuthProvider().authenticate()
        updateSession(result)
    }

    func linkAccount(provider: AuthProvider) async throws {
        guard let currentToken = secureStorage.loadAccessToken() else {
            throw AuthError.notAuthenticated
        }

        let result = try await authService.link(
            currentToken: currentToken,
            provider: provider
        )
        updateSession(result)
    }

    func logout() async throws {
        try await authService.logout()
        secureStorage.clear()
        isAuthenticated = false
        playerId = nil
    }

    private func updateSession(_ result: AuthResult) {
        secureStorage.save(
            accessToken: result.accessToken,
            refreshToken: result.refreshToken
        )
        isAuthenticated = true
        isGuest = result.isGuest
        playerId = result.playerId
    }
}
```

### 13.2 Auth Manager (Kotlin)

```kotlin
class AuthManager(private val context: Context) {
    private val secureStorage = SecureStorage(context)
    private val authService = AuthService()

    val isAuthenticated = MutableStateFlow(false)
    val isGuest = MutableStateFlow(true)
    val playerId = MutableStateFlow<String?>(null)

    suspend fun autoLogin() {
        secureStorage.loadRefreshToken()?.let { refreshToken ->
            try {
                val tokens = authService.refresh(refreshToken)
                updateSession(tokens)
                return
            } catch (e: Exception) {
                secureStorage.clear()
            }
        }

        guestLogin()
    }

    suspend fun guestLogin() {
        val result = GuestAuthProvider(context).authenticate()
        updateSession(result)
    }

    suspend fun loginWithGoogle(activity: Activity) {
        val result = GoogleAuthProvider(activity).authenticate()
        updateSession(result)
    }

    suspend fun linkAccount(provider: AuthProvider) {
        val currentToken = secureStorage.loadAccessToken()
            ?: throw AuthException("Not authenticated")

        val result = authService.link(currentToken, provider)
        updateSession(result)
    }

    suspend fun logout() {
        authService.logout()
        secureStorage.clear()
        isAuthenticated.value = false
        playerId.value = null
    }

    private fun updateSession(result: AuthResult) {
        secureStorage.save(result.accessToken, result.refreshToken)
        isAuthenticated.value = true
        isGuest.value = result.isGuest
        playerId.value = result.playerId
    }
}
```

---

## 14. Compliance

### 14.1 Platform Requirements

| Platform | Requirement |
|----------|-------------|
| iOS | Apple Sign-In required if any social login |
| iOS | Keychain for credential storage |
| Android | Google Play Games integration recommended |
| Android | EncryptedSharedPreferences for storage |
| Both | COPPA compliance for under-13 |
| Both | GDPR data access/deletion |

### 14.2 Data Retention

| Data Type | Retention | Deletion |
|-----------|-----------|----------|
| Account Data | Until deletion request | 30 days after request |
| Auth Logs | 90 days | Automatic |
| Sessions | Until revoked + 30 days | Automatic |
| Failed Attempts | 24 hours | Automatic |

### 14.3 Privacy Considerations

- Minimal data collection
- Email optional for social login
- No tracking across providers
- Clear data usage disclosure
- Easy account deletion

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Backend Lead | _____________ | _____________ | ___/___/___ |
| Security Lead | _____________ | _____________ | ___/___/___ |
| Product Manager | _____________ | _____________ | ___/___/___ |

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-30 | [Author] | Initial authentication specification |

---

*This document contains implementation details. Handle according to company security policies.*
