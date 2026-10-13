# 🔗 HexTinyURL · Link Alchemy Toolkit

> A resilient, privacy-first, client-side URL transformation suite featuring an automated multi-provider fallback chain, zero-knowledge link encryption (Link Lock), per-link analytics, QR synthesis, and full TinyURL v2 management.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Zero Knowledge](https://img.shields.io/badge/Security-AES--256--GCM-10b981.svg)](#-zero-knowledge-link-lock)
[![Fallback Engine](https://img.shields.io/badge/High--Availability-6--Tier%20Failover-6366f1.svg)](#-multi-provider-fallback-system)
[![Client-Side Only](https://img.shields.io/badge/Architecture-Serverless%20%2F%20Static-f59e0b.svg)](#-technical-architecture)
[![Live Demo](https://img.shields.io/badge/Deployment-GitHub%20Pages-brightgreen.svg)](https://hexsleuth.github.io/HexTinyURL/)

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Technical Architecture](#-technical-architecture)
- [Multi-Provider Fallback System](#-multi-provider-fallback-system)
- [Zero-Knowledge Link Lock](#-zero-knowledge-link-lock)
- [API Integrations & Specifications](#-api-integrations--specifications)
  - [1. No-Auth Public Endpoints](#1-no-auth-public-endpoints)
  - [2. TinyURL v2 REST API](#2-tinyurl-v2-rest-api)
- [Installation & Local Setup](#-installation--local-setup)
- [Security & Threat Model](#-security--threat-model)
- [License](#-license)

---

## 🌐 Overview

**HexTinyURL** is an autonomous, browser-based URL toolkit that eliminates single points of failure in link shortening and distribution. Unlike traditional shorteners that enforce vendor lock-in, track user metrics involuntarily, or require paid accounts for fundamental functionality, HexTinyURL operates as a standalone client-side engine.

It blends zero-dependency REST integrations across multiple independent shortening providers with browser-native Web Cryptography APIs, delivering high availability alongside zero-knowledge operational privacy.

---

## ✨ Key Features

- **6-Tier Cascading Failover:** Automatically cycles to backup URL shortening providers whenever an upstream service encounters rate limits, downtime, or network failures.
- **Zero-Knowledge URL Vault (Link Lock):** Encrypts links client-side using AES-256-GCM and PBKDF2; keys and parameters never touch a remote backend.
- **Parametric Shortening Controls:** Supports custom aliases, expiration time-locks, click quotas, and a dynamic UTM parameter synthesizer (`utm_source`, `utm_medium`, `utm_campaign`, etc.).
- **Telemetry & Deep Analytics:** Fetches click volumes, referrers, device distributions, and geographic metadata (with support for CSV, JSON, and XML exports) where supported.
- **TinyURL v2 Management Console:** Full lifecycle management including creation, target destination updates, alias modification, tagging, archiving, and deletion via custom user API tokens.
- **Vector & Raster QR Code Engine:** Generates high-density SVG/PNG QR codes on the fly without making external image requests.

---

## 🏛 Technical Architecture

The application runs entirely within the client runtime (browser sandbox), requiring no proprietary application server or remote database:

```
+-----------------------------------------------------------------------------------+
|                                 USER BROWSER RUNTIME                              |
+-----------------------------------------------------------------------------------+
|  [ Presentation Layer: Responsive Web UI / UTM Builder / QR Code Canvas ]         |
+-----------------------------------------------------------------------------------+
|                                         |                                         |
|                 +-----------------------+-----------------------+                 |
|                 |                                               |                 |
|                 v                                               v                 |
|  +-----------------------------+                 +-----------------------------+  |
|  |   Zero-Knowledge Engine     |                 |  Fallback Dispatcher Engine |  |
|  |  -------------------------  |                 |  -------------------------  |  |
|  |  * Web Crypto API (Subtle)  |                 |  * Sequential Promise Chain |  |
|  |  * AES-256-GCM / PBKDF2     |                 |  * CORS & Timeout Monitors  |  |
|  |  * Hash Anchor (#v1....)    |                 |  * Input Normalization      |  |
|  +-----------------------------+                 +-----------------------------+  |
|                 |                                               |                 |
+-----------------|-----------------------------------------------|-----------------+
                  | (No network transmission)                     |
                  v                                               v
          [ Client-Side URL ]                     [ Upstream Providers (Sequential) ]
          https://host/#v1.<data>                 1. zip1.io  ───► (Timeout/Fail)
                                                  2. tinysrc.me ─► (Fail)
                                                  3. urlvanish ──► (Fail)
                                                  4. is.gd ──────► (Success)
```

---

## 🔁 Multi-Provider Fallback System

Public shortening providers frequently impose undocumented rate limits or suffer unexpected regional outages. HexTinyURL implements an asynchronous pipeline that cycles sequentially through fallback nodes until a valid, canonical short URL is resolved.

### Failover Order & Service Profiles

| Priority | Provider | Protocol / Auth | Custom Aliases | Expiry / Click Caps |
| :---: | :--- | :--- | :---: | :---: |
| **P1** | `zip1.io` | REST JSON (No Auth) | Yes | Yes |
| **P2** | `tinysrc.me` | REST JSON (No Auth) | Yes | No |
| **P3** | `urlvanish.com` | REST Form (No Auth) | No | Yes |
| **P4** | `is.gd` | GET Query (No Auth) | Yes | No |
| **P5** | `v.gd` | GET Query (No Auth) | Yes | No |
| **P6** | `tinyurl.com` | GET / REST v2 (Token optional) | Yes | Yes (v2 Token) |

### Fallback Implementation Pattern

```javascript
async function shortenWithFallback(targetUrl, options = {}) {
  const providers = [
    { name: 'zip1', fn: () => callZip1(targetUrl, options) },
    { name: 'tinysrc', fn: () => callTinySrc(targetUrl, options) },
    { name: 'isgd', fn: () => callIsGd(targetUrl, options) },
    { name: 'tinyurl', fn: () => callTinyUrlPublic(targetUrl, options) }
  ];

  const errors = [];

  for (const provider of providers) {
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 4000); // 4-second failover threshold
      
      const result = await provider.fn({ signal: controller.signal });
      clearTimeout(timeoutId);
      
      if (result && result.shortUrl) {
        return { shortUrl: result.shortUrl, provider: provider.name };
      }
    } catch (err) {
      errors.push({ provider: provider.name, error: err.message });
      // Proceed to the next candidate
    }
  }

  throw new Error(`All providers failed: ${JSON.stringify(errors)}`);
}
```

---

## 🔐 Zero-Knowledge Link Lock

The **Link Lock** subsystem provides end-to-end privacy for sensitive destinations (e.g., private file shares, internal intranet endpoints, confidential documents). The target URL is encrypted client-side using a passphrase and stored exclusively within the URI fragment identifier (hash anchor).

### Cryptographic Parameters
- **Symmetric Cipher:** AES-256-GCM (Galois/Counter Mode) with a 128-bit authentication tag.
- **Key Derivation Function (KDF):** PBKDF2 using HMAC-SHA-256 with $100{,}000$ iterations.
- **Salt:** 16 bytes of cryptographically secure pseudo-random entropy via `crypto.getRandomValues()`.
- **Initialization Vector (IV):** 12 bytes of unique entropy per encryption invocation.
- **Format Schema:** `#v1.<base64url(salt + iv + ciphertext + tag)>`

Because content past the hash sign (`#`) is never sent to the host server in an HTTP request, neither GitHub Pages nor any intermediate network proxy can access the decrypted destination or encryption parameters.

### Cryptographic Lifecycle

```
[ Plaintext URL ] + [ Password ]
         |
         +──► PBKDF2(Password, Salt=16B, Iterations=100,000, HMAC-SHA256) ──► [ 256-bit Key ]
                                                                                     |
[ Plaintext URL ] + [ IV=12B ] ◄────────────────────────────────────────────────────+
         |
         v
[ AES-256-GCM Encrypt ] ──► [ Ciphertext + 16B Auth Tag ]
                                       |
[ Salt (16B) ] + [ IV (12B) ] + [ Ciphertext + Tag ] ──► Base64URL Encoding
                                                               |
                                                               v
                                        Fragment: #v1.<Encoded Payload>
```

---

## 📡 API Integrations & Specifications

### 1. No-Auth Public Endpoints

#### Provider: `zip1.io`

```http
POST https://zip1.io/api/create
Content-Type: application/json

{
  "url": "https://example.com/target-destination",
  "alias": "optional-custom-alias",
  "max_clicks": 100,
  "expire_days": 7
}
```

**Response (`200 OK`):**
```json
{
  "status": "success",
  "short_url": "https://zip1.io/custom-alias",
  "alias": "custom-alias",
  "created_at": "2026-10-13T04:15:00Z"
}
```

#### Provider: `is.gd`

```http
GET https://is.gd/create.php?format=json&url=https%3A%2F%2Fexample.com&shorturl=optional-alias
```

**Response (`200 OK`):**
```json
{
  "shorturl": "https://is.gd/optional-alias"
}
```

---

### 2. TinyURL v2 REST API

HexTinyURL includes an authenticated client for the TinyURL v2 developer API. Tokens are stored locally in the browser's `localStorage` and sent directly to TinyURL over TLS.

#### Create Link

```http
POST https://api.tinyurl.com/create
Authorization: Bearer <YOUR_API_TOKEN>
Content-Type: application/json

{
  "url": "https://example.com/data",
  "domain": "tinyurl.com",
  "alias": "custom-route",
  "tags": "production,marketing",
  "expires_at": "2026-12-31 23:59:59"
}
```

#### Update Link Destination

```http
PATCH https://api.tinyurl.com/change
Authorization: Bearer <YOUR_API_TOKEN>
Content-Type: application/json

{
  "domain": "tinyurl.com",
  "alias": "custom-route",
  "url": "https://example.com/new-destination"
}
```

#### Analytics & Link Inspection

```http
GET https://api.tinyurl.com/alias/tinyurl.com/custom-route
Authorization: Bearer <YOUR_API_TOKEN>
```

---

## 💻 Installation & Local Setup

Because HexTinyURL has no backend dependencies, it can be served using any static web server.

### Prerequisites
- Node.js (v18+), Python (v3.8+), or an equivalent static file server.

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/hexsleuth/HexTinyURL.git

# 2. Enter directory
cd HexTinyURL

# 3. Serve using Python
python3 -m http.server 8080

# Or serve using Node.js npx
npx serve .
```

Navigate to `http://localhost:8080` in any modern, Web Crypto-compatible browser.

---

## 🛡 Security & Threat Model

| Surface | Threat Vector | Mitigation Strategy |
| :--- | :--- | :--- |
| **API Keys** | Exposure / Scraping | Saved strictly to client-side `localStorage`. API tokens never touch third-party servers or telemetry platforms. |
| **Encrypted Links** | Brute Force Attacks | PBKDF2 key stretching ($100{,}000$ rounds) introduces a high computational barrier against offline wordlist attacks. |
| **Data Interception** | Server-Side Leakage | The payload resides after the `#` anchor. RFC 3986 specifies that URI fragments must be stripped before HTTP transmission. |
| **Tampering** | Ciphertext Modification | AES-GCM's 128-bit authentication tag guarantees tamper detection; tampered links fail decryption immediately. |
| **CORS Policy** | Browser Call Blocks | Built-in routing automatically chooses providers that support CORS headers or public query parameters. |

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for complete details.
