# SSL/TLS Practice Lab

A hands-on, fully offline lab for mastering SSL/TLS certificate management in Docker containers.
Directly relevant to SRE work in production auth systems.

## Structure

```
ssl-tls-practice/
├── phase1-baseline/   # Build a full CA → signed cert → HTTPS nginx from scratch
├── phase2-broken/     # 5 pre-sabotaged environments to diagnose and fix
│   ├── s01-expired-cert/
│   ├── s02-san-mismatch/
│   ├── s03-missing-chain/
│   ├── s04-key-mismatch/
│   └── s05-wrong-format/
└── phase3-mtls/       # Mutual TLS between two containers
```

## Workflow

**Phase 1** — Start here. Build everything from scratch inside a container.
No certs pre-provided. You generate every file manually.

**Phase 2** — Each scenario boots a broken HTTPS server.
Your job: hit it, observe the failure, diagnose with `openssl s_client`, fix it.

**Phase 3** — Two-container setup. Neither side trusts the other until you wire up mTLS.

## Key Tools You'll Use

| Tool | Purpose |
|---|---|
| `openssl genrsa` | Generate private keys |
| `openssl req` | Generate CSRs or self-signed certs |
| `openssl x509` | Inspect, sign, or convert certs |
| `openssl s_client` | Debug TLS handshakes (your main prod tool) |
| `openssl verify` | Verify a cert against a CA |
| `curl --cacert` | Test HTTPS with a custom CA |
| `nginx -t` | Validate nginx config before reload |
| `nginx -s reload` | Reload nginx without downtime |

## Quick Reference: Full Cert Lifecycle

```bash
# 1. Generate CA key + self-signed cert
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt -subj "/CN=LabRootCA"

# 2. Generate server key + CSR
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/CN=localhost"

# 3. Sign server cert with your CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt -days 365

# 4. Inspect what you made
openssl x509 -in server.crt -noout -text

# 5. Test the TLS handshake
openssl s_client -connect localhost:8443 -CAfile ca.crt
```
