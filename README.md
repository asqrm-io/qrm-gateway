# 🌐 QRM Gateway  
## API Gateway & Intelligent Routing Layer

---

## Overview

**QRM Gateway** is the unified entry point for all traffic flowing into the QRM platform. It acts as an **API Gateway, reverse proxy, and intelligent routing layer**, managing how requests are authenticated, routed, secured, and observed across services.

It provides a centralized control plane for traffic management across microservices, applications, and infrastructure components.

---

## Key Responsibilities

QRM Gateway is responsible for:

- Request routing to appropriate services
- Authentication and authorization enforcement
- Workspace and tenant-aware request handling
- Security filtering and header policies
- Traffic logging and observability
- Load balancing and service discovery
- Protocol handling (HTTP, HTTPS, WebSocket, gRPC)

---

## Sample Route Configuration: webadmin.yaml

```yaml
routes:
  # =============================================================================
  # Web Admin Dashboard - Full Professional Configuration Example
  # =============================================================================
  - name: webadmin
    domain: admin.example.com
    path: /
    target: webadmin
    app_status: registered

    # ---------------------------------------------------------------------------
    # Proxy Configuration
    # ---------------------------------------------------------------------------
    proxy:
      enabled: true
      upstream: http://127.0.0.1:3000
      timeout: 60
      preserve_host: true
      strip_path: false

    # ---------------------------------------------------------------------------
    # SSL/TLS Configuration
    # ---------------------------------------------------------------------------
    ssl:
      enabled: true
      cert_file: cert.pem
      key_file: key.pem
      # Or use auto-detection: /qrm/gateway/certificates/{domain}/cert.pem

    # ---------------------------------------------------------------------------
    # IP Filtering (Access Control)
    # ---------------------------------------------------------------------------
    ip_filter:
      enabled: true
      whitelist:
        - 192.168.1.0/24  # Office network
        - 10.0.0.0/8      # Internal network
      blacklist:
        - 0.0.0.0/8       # Block invalid source

    # ---------------------------------------------------------------------------
    # Rate Limiting
    # ---------------------------------------------------------------------------
    rate_limit:
      enabled: true
      requests_per_sec: 100
      burst_size: 200
      window_seconds: 60
      max_requests: 1000
      strategy: token_bucket  # token_bucket, sliding_window
      limit_by: ip            # ip, domain, header, cookie
      block_duration: 300

    # ---------------------------------------------------------------------------
    # Authentication (Professional Feature)
    # ---------------------------------------------------------------------------
    authentication:
      enabled: true
      type: jwt  # basic, jwt, api_key, oauth2

      # Basic Authentication
      basic:
        realm: "Admin Area"
        users:
          - username: admin
            password: "$2a$10$..."  # bcrypt hash

      # JWT Authentication
      jwt:
        secret: "your-secret-key"
        # public_key: "/path/to/public.pem"  # For RS256/ES256
        algorithm: HS256  # HS256, RS256, ES256
        issuer: "https://auth.example.com"
        audience:
          - "admin-api"
        header_name: Authorization
        token_prefix: "Bearer "
        claims_to_pass:
          - sub
          - role
          - email

      # API Key Authentication
      api_key:
        header_name: X-API-Key
        query_param: api_key
        keys:
          - key: "sk_live_abc123..."
            name: "Production Key"
            permissions:
              - read
              - write
            expires_at: "2025-12-31T23:59:59Z"

      # OAuth2 Authentication
      oauth2:
        provider: google  # google, github, custom
        client_id: "your-client-id"
        client_secret: "your-client-secret"
        auth_url: "https://accounts.google.com/o/oauth2/auth"
        token_url: "https://oauth2.googleapis.com/token"
        userinfo_url: "https://www.googleapis.com/oauth2/v2/userinfo"
        redirect_url: "https://admin.example.com/oauth/callback"
        scopes:
          - openid
          - email
          - profile
        allowed_domains:
          - example.com

    # ---------------------------------------------------------------------------
    # CORS Configuration
    # ---------------------------------------------------------------------------
    cors:
      enabled: true
      allow_origins:
        - "https://app.example.com"
        - "https://dashboard.example.com"
      allow_methods:
        - GET
        - POST
        - PUT
        - DELETE
        - OPTIONS
      allow_headers:
        - Content-Type
        - Authorization
        - X-Requested-With
      expose_headers:
        - X-Total-Count
        - X-Page-Count
      allow_credentials: true
      max_age: 86400

    # ---------------------------------------------------------------------------
    # Header Manipulation
    # ---------------------------------------------------------------------------
    headers:
      request:
        X-Forwarded-Proto: https
        X-Real-IP: "$remote_addr"
      response:
        X-Frame-Options: DENY
        X-Content-Type-Options: nosniff
        X-XSS-Protection: "1; mode=block"
        Strict-Transport-Security: "max-age=31536000; includeSubDomains"
      remove:
        - Server
        - X-Powered-By

    # ---------------------------------------------------------------------------
    # Health Check Configuration
    # ---------------------------------------------------------------------------
    health_check:
      enabled: true
      path: /health
      interval: 10
      timeout: 5
      healthy_threshold: 2
      unhealthy_threshold: 3
      expected_status: 200

    # ---------------------------------------------------------------------------
    # Circuit Breaker
    # ---------------------------------------------------------------------------
    circuit_breaker:
      enabled: true
      threshold: 5          # Number of failures before opening
      timeout: 30           # Seconds to wait before half-open
      half_open_requests: 3 # Requests allowed in half-open state

    # ---------------------------------------------------------------------------
    # Response Caching
    # ---------------------------------------------------------------------------
    cache:
      enabled: true
      ttl: 300
      methods:
        - GET
        - HEAD
      status_codes:
        - 200
        - 301
        - 302
      vary_headers:
        - Accept
        - Accept-Encoding
      bypass_header: X-Cache-Bypass
      stale_if_error: 60

    # ---------------------------------------------------------------------------
    # Logging Configuration
    # ---------------------------------------------------------------------------
    logging:
      enabled: true
      level: info  # debug, info, warn, error
      include_body: false
      max_body_size: 4096
      sensitive_keys:
        - password
        - token
        - secret
        - api_key
      format: json  # json, text

    # ---------------------------------------------------------------------------
    # Request/Response Transformation
    # ---------------------------------------------------------------------------
    transform:
      request:
        rewrite_path: "/api/v2$uri"
        add_headers:
          X-Gateway-Version: "2.0"
        remove_headers:
          - X-Debug
        add_query_params:
          source: gateway
      response:
        add_headers:
          X-Cache-Status: "$cache_status"
        remove_headers:
          - X-Internal-Debug
        body_replace:
          "http://": "https://"

    # ---------------------------------------------------------------------------
    # Compression
    # ---------------------------------------------------------------------------
    compression:
      enabled: true
      min_size: 1024
      level: 6
      types:
        - text/html
        - text/css
        - text/javascript
        - application/json
        - application/javascript
      algorithms:
        - gzip
        - brotli

    # ---------------------------------------------------------------------------
    # WebSocket Support
    # ---------------------------------------------------------------------------
    websocket:
      enabled: true
      path: /ws
      ping_interval: 30
      read_buffer_size: 4096
      write_buffer_size: 4096
      max_message_size: 65536

    # ---------------------------------------------------------------------------
    # Retry Configuration
    # ---------------------------------------------------------------------------
    retry:
      enabled: true
      max_retries: 3
      retry_on:
        - 502
        - 503
        - 504
      backoff_ms: 1000
      backoff_type: exponential  # fixed, exponential

    # ---------------------------------------------------------------------------
    # Load Balancer Configuration
    # ---------------------------------------------------------------------------
    load_balancer:
      algorithm: round_robin  # round_robin, least_conn, ip_hash, weighted
      sticky_session: true
      cookie_name: SERVERID
      cookie_ttl: 3600

    # ---------------------------------------------------------------------------
    # Multiple Upstreams (for Load Balancing)
    # ---------------------------------------------------------------------------
    upstreams:
      - url: http://backend1.internal:3000
        weight: 3
        max_conns: 100
      - url: http://backend2.internal:3000
        weight: 2
        max_conns: 100
      - url: http://backend3.internal:3000
        weight: 1
        max_conns: 50
        backup: true
```

---

## Example Use Cases

- Routing API traffic to microservices
- Providing a single endpoint for all QRM services
- Enforcing workspace-level access control
- Acting as an ingress for deployed applications
- Centralized API security and monitoring

---

## Vision

QRM Gateway is designed to be the **traffic brain of the QRM ecosystem** — providing secure, observable, and intelligent routing so that backend services can stay simple, focused, and scalable.
