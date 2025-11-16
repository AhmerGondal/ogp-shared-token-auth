<img width="1160" height="534" alt="image" src="https://github.com/user-attachments/assets/31670631-efbf-4f87-b6b3-72b8917654b5" />
# ogp-shared-token-auth

A lightweight, production-tested authentication subsystem providing **compact HMAC-signed tokens** for secure dashboard access across multiple internal platforms (ogPumper, ogSupport, ogFieldTicket, and internal admin tools).

This library powers:

- One-click dashboard access links  
- Secure iframe embedding  
- Tenant-scoped user views  
- Temporary tokens for pumpers, operators, investors, and support staff  
- Cross-application authentication handoff  
- Internal tools running on legacy PHP or cPanel-style hosting

It is intentionally **simpler than JWT**, easier to embed in mixed environments, and optimized for **multi-tenant internal SaaS tools**.

---

# 🚀 Features

### 🔐 Compact URL-safe single-segment token  
No header segment, no multi-part JWT structure:

```
token = b64url( payload_b64 + "." + sig_b64 )
```

### 🛡️ HMAC-SHA256 signatures  
Symmetric crypto: fast, portable, minimal dependencies.

### 🏷️ First-class claims support  
- `sub` — subject (user ID)  
- `aud` — audience restriction  
- `exp` — expiration timestamp  
- `nbf` — not-before timestamp  
- Custom claims like `tenant`, `role`, `flags`  

### 🔍 Automatic secret resolution with fallback search paths  
Supports:

- `~/.ogpumper/secret.ini` (canonical)  
- Environment variables  
- `.env` files  
- PHP array-based configs  
- Legacy paths used in shared hosting / cPanel  
- Multiple variable name fallbacks (`OG_SHARED_TOKEN_SECRET`, `SECRET`)  

### 🧱 Shared request guard for PHP endpoints  
Protect any PHP endpoint instantly:

```php
$claims = ogp_require_token(['aud' => 'dashboard']);
```

Automatically:

- Validates signature  
- Validates `exp`, `nbf`, and `aud`  
- Returns `401` JSON on failure  
- Exits safely  

### 🧩 JS soft validator (browser-friendly)
Client-side utilities allow:

- Parsing and inspecting tokens  
- Checking structure  
- Checking `exp` / `nbf` / `aud`  
- Optional round-trip server verification  

Useful for embedded dashboards and SPA flows.

---

# 📦 Folder Structure

```
ogp-shared-token-auth/
├── README.md
├── src/
│   ├── php/
│   │   └── ogp_token.php          # Main PHP implementation
│   └── js/
│       └── tokenUtils.js          # Client-side parser & validator
└── examples/
    ├── php/
    │   ├── issue_token.php        # Example: server-side token issuing
    │   └── protected_endpoint.php # Example: secured endpoint
    └── js/
        └── demo.html              # Example: browser-side token parsing
```

---

# 🔧 Token Format

The token is intentionally compact and URL-safe.

### 1. Encode JSON payload → base64url  
```
payload_b64 = b64url(JSON.stringify(claims))
```

### 2. Compute signature  
```
sig_b64 = b64url(HMAC_SHA256(payload_b64, secret))
```

### 3. Wrap both into a single URL-safe token  
```
token = b64url( payload_b64 + "." + sig_b64 )
```

### Example Payload

```json
{
  "sub": "user_123",
  "aud": "ogp-dashboard",
  "exp": 1735689600,
  "nbf": 1735686000,
  "tenant": "acme-oilfield",
  "role": "pumper"
}
```

---

# 🐘 PHP Usage

## 1. Load the library

```php
require_once __DIR__ . '/src/php/ogp_token.php';
```

## 2. Issue a token

```php
$claims = [
  'sub'    => 'user_123',
  'aud'    => 'ogp-dashboard',
  'exp'    => time() + 3600,
  'tenant' => 'acme-oilfield',
  'role'   => 'pumper'
];

$secret = ogp_resolve_secret();
$token  = ogp_make_token($claims, $secret);

echo $token;
```

## 3. Protect an endpoint

```php
$claims = ogp_require_token([
  'aud'    => 'ogp-dashboard',
  'leeway' => 60
]);

echo json_encode([
  'success' => true,
  'claims'  => $claims
]);
```

### Behavior on invalid token
`ogp_require_token()` will:

- return HTTP `401`
- output:  
  ```json
  { "success": false, "error": "unauthorized" }
  ```
- immediately `exit`

---

# 🌐 JavaScript Usage

## 1. Parse a compact token

```javascript
import { parseCompactToken } from "./tokenUtils.js";

const parsed = parseCompactToken(token);
if (!parsed.ok) {
  console.error("Token invalid:", parsed.reason);
}
console.log("Claims:", parsed.claims);
```

## 2. Soft validation (client-side)

```javascript
import { softValidateToken } from "./tokenUtils.js";

const result = softValidateToken(token, { audience: "ogp-dashboard" });
if (!result.ok) {
  console.error("Token not valid:", result.reason);
}
```

## 3. Optional server validation

```javascript
import { serverValidateToken } from "./tokenUtils.js";

const res = await serverValidateToken(token, "/api/auth/check");
console.log(res.ok ? "Token OK" : "Token invalid:", res.reason);
```

---

# 🔐 Security Model

### Designed for:
- internal tools  
- dashboard embedding  
- cross-application auth handoff  
- pumpers/investor/support secure links  
- cPanel / legacy hosting  
- multi-tenant internal products  

### NOT designed to replace:
- OAuth2  
- OIDC  
- SSO for public consumer apps  
- Multi-party public token issuance  

---

# ⚠️ Limitations

- Symmetric secret: all verifying services must share the same secret  
- Not intended for consumer-facing public auth  
- HTTPS required to prevent token interception  
- Recommended token size < 1KB  

---

# 📅 Roadmap

- Optional RSA/ECDSA asymmetric mode  
- Token revocation list (Redis or file-backed)  
- One-time-use expiring links  
- Node.js and Python server implementations  
- TypeScript typings for tokenUtils.js  

---

# 📄 License

MIT License.
