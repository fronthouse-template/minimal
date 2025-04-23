# 🏠 FrontHouseClient

**FrontHouseClient** is a modular, high-performance reverse proxy with smart request filtering, rate limiting, mTLS support, and adaptive behavior based on real-time analytics. Built for privacy-aware routing, resilience to abuse, and observability across distributed environments.

---

## 📦 Summary

- ⚡ HTTP(S) + WebSocket + gRPC reverse proxy
- 📊 Per-IP behavior tracking + learning
- ☁️ Distributed Redis-backed state
- 🔄 Self-healing config + cert reload
- 🔐 TLS termination and mTLS upstream
- 🧠 Training/production mode switching
- 🚦 Integrated Redis-based rate limiting

---

## 🚀 Full Feature List

### 🧠 Smart Request Filtering & Learning

- ✅ **Training mode** (learn-only): observes request outcomes without blocking
- ✅ **Production mode** (enforce): blocks based on credibility and blacklists
- ✅ **URL whitelisting** on successful responses (2xx, 3xx)
- ✅ **URL blacklisting** on repeated client-side failures (4xx)
- ❌ **Ignores 5xx** responses to avoid poisoning the blacklist
- 🔁 **Global learning pause** if 5xx errors spike (configurable window and threshold)
- 🧯 **Rate-limited learning** (per IP and URL) to prevent Redis overuse or DDoS abuse
- 🔄 **Graceful IP adaptation**:
  - Allows first 10 requests per new IP
  - Tracks request outcome ratio per IP
  - Blocks IPs falling below credibility thresholds
  - Automatically unblocks well-behaved IPs

---

### 🚦 Rate Limiting (Redis-Based)

- ✅ Global per-IP rate limit (e.g. 100 req/min)
- ✅ Per-IP independent rate limit (separate from routing)
- ✅ Global per-route rate limit (shared across all IPs)
- ✅ Configurable via `.env` and `config.json`
- ✅ Efficient fixed-window algorithm using Redis keys
- ✅ Bucket expiration to prevent memory bloat
- ✅ Clear console logging when rate limits are hit

---

### 🧩 Routing & Load Balancing

- ✅ **Path-based routing** using full regex (`^/api`, `^/admin`, etc.)
- 🎯 **Multiple upstream targets** per route
- ⚖️ **Round-robin or weighted round-robin** strategies
- 🧬 **Sticky sessions** via:
  - IP Hash
  - Cookie strategy with custom name
- 🔄 **Smart circuit breaker** per route:
  - Opens after N failures within time window
  - Auto-close after configured timeout
  - Resets on any successful request
  - Ignores client errors (4xx)
  - Only counts server errors (5xx)
- 🔁 **Retry policy** with configurable delay and attempt count

---

### 🔒 Security & TLS

- ✅ **TLS termination** via:
  - Base64-encoded certificate in `config.json`
  - Legacy support: cert file path + password from `.env`
- ✅ **mTLS support**:
  - Each route can specify a client certificate
  - Certificates are hot-reloaded on change
- 🔐 **Header forwarding** is clean and controllable (inject or strip as needed)
- 🔑 **API Authentication**:
  - Configurable via `config.json` or `.env`
  - Supports API key and token authentication
  - Secure storage of credentials

---

### 🔄 Protocol Support

- ✅ **HTTP**
- ✅ **HTTPS**
- ✅ **WebSocket** pass-through support
- ✅ **gRPC** with streaming support
- ✅ **Chunked and large file uploads/downloads** using streamed proxying

---

### 📡 Health Checking

- ✅ Per-target health checks (optional)
  - Supports HTTP health endpoints (`/health`)
  - Supports TCP connection checks (e.g., port 443 alive)
  - Supports gRPC health checks (`grpc.health.v1.Health/Check`)
- ⛔ Unhealthy targets are automatically excluded from routing

---

### 🧠 Redis-Backed Intelligence

- 🧠 Tracks total and blocked requests per IP
- 🧠 Maintains allow-list flags for trusted IPs
- 🧠 Stores whitelisted/blacklisted URLs
- 🧠 Tracks global 5xx patterns for learning suppression
- 🧠 TTL-based counters (sliding expiration) for IP stats and limits

---

### 🔧 Configuration & DevOps

- ✅ **Configuration priority**:
  1. `config.json` values take precedence
  2. Falls back to `.env` values if not found in `config.json`
- ✅ **Credentials in `config.json`**:
  - API key and token
  - TLS certificate (base64-encoded)
  - Certificate password
- ✅ **Legacy `.env` support**:
  - API credentials
  - File-based TLS certificates
- 🔄 **Hot-reloads `config.json`** on file change
- 🔄 **Hot-reloads mTLS certificates** automatically
- ✅ Minimal NuGet dependencies for performance and portability

---

### ☁️ Distributed Ready

- 💾 All state shared via Redis — supports horizontal scaling
- 🧊 Stateless proxy instance — restart-safe
- ✅ Shared circuit breaking, load balancing, blacklisting across nodes

---

## 🚫 Blocking Rules (Detailed)

In **production mode**, the proxy evaluates each request against a set of escalating filters:

1. **Training Mode?**
   - ✅ Allow all traffic (no blocking).
   - 🔁 Still observe outcomes for learning.

2. **New IP?**
   - If fewer than 10 total requests seen → ✅ Allow.

3. **Credibility Score**
   - Calculated as:  
     `Credibility = 1.0 - (blocked_requests / total_requests)`
   - If `credibility >= BLOCK_THRESHOLD` → ✅ Allow

4. **URL in Local Whitelist?**
   - ✅ Allow and log

5. **URL in Local Blacklist?**
   - ❌ Block and log

6. **URL in Remote Blacklist?**
   - ❌ Block and log

7. **Credibility < BLOCK_HARD_THRESHOLD?**
   - If URL not in any list → ❌ Block

8. **Final fallback**
   - ✅ Allow, but log as "borderline credibility"

---

## 🔧 Modes of Operation

### 🔨 `training`
- Proxy **never blocks**
- It **observes** request outcomes:
  - ✅ Adds 2xx/3xx URLs to whitelist
  - ❌ Adds 4xx URLs to blacklist

### 🚨 `production`
- Proxy enforces full blocking rules
- Uses Redis-stored whitelist, blacklist, and IP reputation

> Controlled by Redis key: `CLIENT:config:mode`

---

## ⚙️ Configuration

### 🔧 Configuration Priority

1. Values in `config.json` take precedence
2. Falls back to `.env` values if not found in `config.json`

### 📝 Config.json Structure

```json
{
  "Api": {
    "Key": "your-api-key",        // API key for remote blacklist URL lookup and Redis prefix
    "Token": "your-api-token"     // API token for remote API auth
  },
  "Tls": {
    "CertificateBase64": "base64-encoded-cert-data",  // Base64 encoded PFX/PKCS#12
    "CertificatePassword": "your-cert-password"       // Certificate password
  },
  "Routes": [
    {
      "PathRegex": "^/api",
      "RateLimitPerMinute": 30,
      "LoadBalancing": "WeightedRoundRobin",
      "PreserveRequestPath": false,
      "PreserveQueryString": true,
      "StickySession": {
        "Enabled": true,
        "Strategy": "Cookie",
        "CookieName": "STICKY"
      },
      "RetryPolicy": {
        "MaxRetries": 2,
        "RetryDelayMs": 200
      },
      "CircuitBreaker": {
        "FailureThreshold": 10,
        "OpenPeriodSeconds": 30,
        "FailureWindowSeconds": 60
      },
      "ClientCertificate": {
        "CertPath": "certs/client.pfx",
        "CertPassword": "secret"
      },
      "Targets": [
        {
          "Url": "https://api-backend-1.internal",
          "Weight": 3,
          "HealthCheck": {
            "Type": "http",
            "Path": "/health",
            "IntervalSeconds": 10
          }
        }
      ]
    }
  ]
}
```

### 📄 Environment Configuration

#### 🔌 Required Settings
| Key | Description | Default |
|-----|-------------|---------|
| `REDIS_CONNECTION` | Redis connection string | *(required)* |

#### 🔐 Authentication (Legacy Support)
| Key | Description | Default |
|-----|-------------|---------|
| `KEY` | API key for blacklist lookup and Redis prefix | *(falls back to config.json)* |
| `TOKEN` | API token for remote API auth | *(falls back to config.json)* |

#### 🔒 TLS (Legacy Support)
| Key | Description | Default |
|-----|-------------|---------|
| `SSL_CERT_PATH` | Path to TLS certificate file | *(falls back to config.json)* |
| `SSL_CERT_PASSWORD` | Password for the certificate | *(falls back to config.json)* |

#### 🧠 Learning & Blocking
| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `BLOCK_THRESHOLD` | `double` | `0.5` | If credibility ≥ this, allow IP |
| `BLOCK_HARD_THRESHOLD` | `double` | `0.2` | Below this → block even if URL is clean |
| `LEARNING_GLOBAL_5XX_THRESHOLD` | `int` | `20` | Pause learning if 5xx errors exceed this |

#### 🚦 Rate Limiting
| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `RATE_LIMIT_ENABLED` | `bool` | `true` | Enable/disable all rate limiting |
| `RATE_LIMIT_GLOBAL_PER_MINUTE` | `int` | `100` | Max requests per IP across all paths |
| `RATE_LIMIT_PER_IP_PER_MINUTE` | `int` | `200` | Max requests per IP (independent of route) |

## 🧠 Redis Keys

All Redis keys are prefixed with your API key for isolation and security:

| Redis Key | Purpose |
|-----------|---------|
| `{api-key}:req:total:{ip}` | All requests from IP |
| `{api-key}:req:blocked:{ip}` | Blocked requests from IP |
| `{api-key}:req:allow:{ip}` | IP is trusted even if matching blacklist |
| `{api-key}:urls:whitelist` | Known good URLs |
| `{api-key}:urls:blacklist` | Known bad URLs |
| `{api-key}:config:mode` | Current operational mode |
| `{api-key}:ratelimit:{ip}:{minute}` | Global rate limit bucket per IP |
| `{api-key}:ratelimit:ip:{ip}:{minute}` | Per-IP general bucket |
| `{api-key}:ratelimit:{ip}:{path}:{minute}` | Per-route rate bucket |

> Note: The API key from `config.json` (or fallback to `.env` KEY) is used as a prefix for all Redis keys, providing isolation between different instances and improved security.

### 🔄 Hot Reloading

- ✅ `config.json` is automatically reloaded when changed
- ✅ TLS certificates are hot-reloaded when modified
- ✅ No service restart required for configuration updates

### 🏗️ Deployment Considerations

#### Cloud-Native Deployments (Recommended)
- Use `config.json` for all configuration except Redis connection
- Store certificates as base64 strings in configuration
- Utilize container secrets management
- Enable horizontal scaling

#### Traditional Deployments
- Can use `.env` file for all configuration
- Support for file-based certificates
- Maintains compatibility with existing setups

## Environment Variables

### Redis Configuration
- `REDIS_CONNECTION`: Redis connection string (default: redis://localhost:6379)

### 🔄 gRPC Support

- ✅ **Full gRPC protocol support**:
  - Unary calls
  - Server streaming
  - Client streaming
  - Bidirectional streaming
- 🔧 **Configurable settings**:
  - Maximum message size
  - Compression options
  - Keep-alive intervals
- 🏥 **Health checking**:
  - Native gRPC health protocol support
  - Automatic service discovery
- 🔄 **Protocol handling**:
  - Automatic HTTP/2 upgrade
  - Header forwarding
  - Status code mapping
- ⚡ **Performance optimizations**:
  - Streaming message handling
  - Connection pooling
  - Multiple HTTP/2 connections
- 🔐 **Security**:
  - TLS 1.2/1.3 support
  - Custom certificate validation
  - Header sanitization

### ☁️ Redis Configuration

| Key | Description | Default |
|-----|-------------|---------|
| `REDIS_CONNECTION` | Redis connection string | *(required)* |

> Note: Redis keys are now automatically prefixed with your API key for isolation and security.

## 🧠 Redis Keys

All Redis keys are prefixed with your API key:

| Redis Key | Purpose |
|-----------|---------|
| `{api-key}:req:total:{ip}` | All requests from IP |
| `{api-key}:req:blocked:{ip}` | Blocked requests from IP |
| `{api-key}:req:allow:{ip}` | IP is trusted even if matching blacklist |
| `{api-key}:urls:whitelist` | Known good URLs |
| `{api-key}:urls:blacklist` | Known bad URLs |
| `{api-key}:config:mode` | Current operational mode |
| `{api-key}:ratelimit:{ip}:{minute}` | Global rate limit bucket per IP |
| `{api-key}:ratelimit:ip:{ip}:{minute}` | Per-IP general bucket |
| `{api-key}:ratelimit:{ip}:{path}:{minute}` | Per-route rate bucket |

