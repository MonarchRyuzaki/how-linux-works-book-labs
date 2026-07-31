# Phase 1 — Build a Full CA → Signed Cert → HTTPS Nginx from Scratch

## Objective
Go from zero to a fully working HTTPS server running in a Docker container.
No certificates are pre-provided. You generate every file manually.

## Setup

```bash
mkdir ssl   # This folder gets mounted into the container as /etc/nginx/ssl/
docker compose up -d
docker compose exec server sh   # Drop into the container shell
```

Verify HTTP works first:
```bash
curl http://localhost:8080
# Expected: <h1>SSL Lab - Phase 1</h1>
```

---

## Step 1 — Generate a Root CA (Certificate Authority)

Inside the container:

```bash
cd /etc/nginx/ssl
```

Generate the CA private key:
```bash
openssl genrsa -out ca.key 2048
```

Generate the CA certificate (self-signed, valid 10 years):
```bash
openssl req -new -x509 -days 3650 \
  -key ca.key \
  -out ca.c. Why HTTPS needs two different crypto systems
Asymmetric crypto (RSA/ECDSA) is slow and can only encrypt small amounts of data efficiently. Symmetric crypto (AES, ChaCha20) is fast and handles bulk data — but requires both sides to already share a secret key.

TLS's job: use asymmetric crypto once, briefly, to safely agree on a symmetric key. Then switch to symmetric crypto for the actual HTTP traffic.

That's the entire point of the handshake. Everything below is in service of that.

3rt \
  -subj "/CN=LabRootCA/O=SRELab/C=IN"
```

> **Why a CA?** In production, you never self-sign a server cert directly.
> You create a CA and use it to *sign* server certs. This is how trust chains work.

---

## Step 2 — Generate the Server Key and CSR

```bash
# Private key for the server
openssl genrsa -out server.key 2048

# CSR = Certificate Signing Request (what you "submit" to the CA)
openssl req -new \
  -key server.key \
  -out server.csr \
  -subj "/CN=localhost/O=SRELab/C=IN"
```

Inspect the CSR:
```bash
openssl req -in server.csr -noout -text
```

---

## Step 3 — Sign the Server Cert with Your CA

```bash
openssl x509 -req \
  -in server.csr \
  -CA ca.crt \
  -CAkey ca.key \
  -CAcreateserial \
  -out server.crt \
  -days 365
```

Inspect the signed cert:
```bash
openssl x509 -in server.crt -noout -text
# Look for: Issuer (your CA), Subject (localhost), Validity dates
```

Verify the cert was signed by your CA:
```bash
openssl verify -CAfile ca.crt server.crt
# Expected: server.crt: OK
```

---

## Step 4 — Enable HTTPS in Nginx

On the **host machine**, edit `nginx.conf`:
- Uncomment the `server { listen 443 ssl; ... }` block
- Save the file

Validate the config and reload nginx (inside container):
```bash
nginx -t                # Validate — must say "test is successful"
nginx -s reload         # Reload without downtime
```

---

## Step 5 — Test the TLS Handshake

From the **host machine**:

```bash
# Raw TLS handshake inspection (your main SRE debugging tool)
openssl s_client -connect localhost:8443 -CAfile ssl/ca.crt

# Should print the certificate chain and end with:
# Verify return code: 0 (ok)

# Test with curl using your custom CA
curl --cacert ssl/ca.crt https://localhost:8443
# Expected: <h1>SSL Lab - Phase 1</h1>
```

---

## Step 6 — Add Your CA to the System Trust Store (Bonus)

Inside the container:
```bash
cp /etc/nginx/ssl/ca.crt /usr/local/share/ca-certificates/lab-ca.crt
update-ca-certificates

# Now curl works WITHOUT needing --cacert
curl https://localhost:443
```

---

## Verification Checklist

- [X] `openssl verify -CAfile ca.crt server.crt` → `OK`
- [x] `nginx -t` → `test is successful`
- [x] `openssl s_client -connect localhost:8443 -CAfile ssl/ca.crt` → `Verify return code: 0 (ok)`
- [x] `curl --cacert ssl/ca.crt https://localhost:8443` → returns HTML
- [x] `openssl x509 -in ssl/server.crt -noout -text` — Issuer is `LabRootCA`, not the server itself

## Key Concepts to Internalize

| Concept | What it means |
|---|---|
| **CA** | The trusted authority that vouches for server certs. YOU are the CA here. |
| **CSR** | The request sent to the CA: "please sign my server's public key" |
| **Chain of Trust** | Browser trusts Root CA → Root CA vouches for server → browser trusts server |
| **`-CAcreateserial`** | Creates a serial number file (`ca.srl`) to track issued certs |
| **`openssl s_client`** | Your #1 tool for debugging TLS. Shows cert chain, expiry, cipher suite used |
