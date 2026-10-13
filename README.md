# 🔗 HexTinyURL (Vol. 05 · Link Alchemy)

> A client-side, privacy-first URL toolkit featuring resilient multi-provider shortening, zero-knowledge link encryption (Link Lock), per-link analytics, QR code generation, and official TinyURL v2 management.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Zero Knowledge](https://img.shields.io/badge/Security-AES--256--GCM-green.svg)](#-zero-knowledge-link-lock)
[![No Auth](https://img.shields.io/badge/API-No--Auth%20Available-orange.svg)](#-api-reference)
[![Live Demo](https://img.shields.io/badge/Live%20Demo-HexTinyURL-brightgreen.svg)](https://hexsleuth.github.io/HexTinyURL/)

---

## 📖 Overview

**HexTinyURL** is an all-in-one web utility and API client built to handle link transformation, privacy protection, and management without compulsory registrations or subscriptions. It combines no-auth public shorteners with local browser-grade cryptography and official developer endpoints.

### Core Highlights
- **High-Availability Fallback Chain:** Automatically cycles through multiple providers if one is unreachable.
- **Zero-Knowledge URL Encryption:** Browser-based AES-256-GCM encryption storing ciphertexts entirely inside URL hash fragments.
- **No-Auth REST Endpoints:** Programmatically generate short URLs with zero API keys or logins.
- **Granular Link Controls:** Expiration timestamps, maximum click caps, and built-in UTM campaign tagging.
- **Deep TinyURL v2 Integration:** Manage custom aliases, change target destinations, generate QR codes, and read link telemetry with your personal token.
- **Agent/MCP Friendly:** Ready for automation workflows and AI tool integration.

---

## 🛠 Features Breakdown

### 1. Multi-Provider URL Shortening
HexTinyURL executes a resilient client-side fallback chain to guarantee URL resolution even during third-party downtime:

$$\text{zip1.io} \longrightarrow \text{tinysrc.me} \longrightarrow \text{urlvanish} \longrightarrow \text{is.gd} \longrightarrow \text{v.gd} \longrightarrow \text{tinyurl.com}$$

- **Advanced Options:**
  - Custom alias assignment.
  - Expiration time & date locks.
  - Maximum click thresholds (link auto-terminates once reached).
  - Pre-attached UTM campaign builders (`utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`).

---

### 2. Zero-Knowledge "Link Lock"
Encrypts the target URL directly within the URL hash fragment (`#v1....`). **The server never sees the plaintext URL, password, or encryption keys.**

* **Algorithm:** AES-256-GCM
* **Key Derivation:** PBKDF2 (`SHA-256`, 100,000 iterations)
* **Payload Structure:** `v1.` + Base64URL-encoded (`Salt (16B)` + `IV (12B)` + `Ciphertext`)
* **Use Cases:**
  * Password-protected Dropbox/Google Drive and magnet links.
  * Private, local bookmarks on shared workstations.
  * Censorship evasion and anti-scraping protections.

---

### 3. Per-Link Analytics
For supported providers (such as `zip1.io`), monitor link health and audience reach without building a dedicated backend:
- Click totals & unique visitor metrics.
- Geolocation / country breakdowns.
- HTTP Referrer headers, operating systems, and client browsers.
- Multi-format data exports: **CSV**, **JSON**, **XLSX**, and **XML**.

---

### 4. Official TinyURL v2 API & QR Generator
For users with a TinyURL API token:
- Shorten links using branded domains (e.g., `go.yourdomain.com`).
- Live QR Code generator with instant high-res PNG download.
- Full link lifecycle operations: **Get Info**, **Change Long URL**, **Update Tags/Alias**, **Archive/Unarchive**, and **Delete**.

---

## 🚀 Quickstart & Usage

### Web Interface
Access the static app directly via GitHub Pages:  
👉 **[https://hexsleuth.github.io/HexTinyURL/](https://hexsleuth.github.io/HexTinyURL/)**

### Local Deployment
Since HexTinyURL is completely static, clone and serve it using any web server:

```bash
# Clone the repository
git clone [https://github.com/hexsleuth/HexTinyURL.git](https://github.com/hexsleuth/HexTinyURL.git)

# Navigate to directory
cd HexTinyURL

# Run with any static HTTP server (e.g., Python 3)
python3 -m http.server 8080
