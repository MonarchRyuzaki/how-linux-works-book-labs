# Phase 3 — Mutual TLS (mTLS)

## What is mTLS?

In normal HTTPS, only the **server** proves its identity with a cert. The client is anonymous.

In **mTLS**, **both sides** must present a valid certificate:
- Server → proves it is the real server
- Client → proves it is an authorized client

This is the standard for **service-to-service auth** in banking and microservice architectures.
If a service doesn't present a valid cert, the connection is rejected at the TLS layer —
before any application code runs.

## Architecture

```
[client container]  --mTLS-->  [server container / nginx]
   presents client.crt           demands client cert via
   signed by shared CA            ssl_verify_client on;
```

Both containers are on the same Docker network (`mtls-net`) and share an `ssl/` volume.

## Setup

```bash
mkdir ssl   # Shared cert directory (mounted into both containers)
docker compose up -d --build
```

## Step 1 — Generate the Shared CA

All certs in an mTLS system must be signed by the **same CA**.
Both the server and client trust this CA.

```bash
# Run on the HOST inside the ssl/ directory
cd ssl

openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 \
  -key ca.key \
  -out ca.crt \
  -subj "/CN=mTLSRootCA"
```

## Step 2 — Generate the Server Certificate

```bash
openssl genrsa -out server.key 2048

# SAN must match the container name ("server") since that's how the client resolves it
printf '[v3_req]\nsubjectAltName=DNS:server\n' > server-san.ext

openssl req -new -key server.key -out server.csr -subj "/CN=server"

openssl x509 -req \
  -in server.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out server.crt \
  -days 365 \
  -extfile server-san.ext -extensions v3_req
```

## Step 3 — Start the Server

The `ssl/` directory is already volume-mounted. Start/restart the server:

```bash
docker compose restart server
docker compose logs server
# Must show: nginx started successfully (no errors)
```

## Step 4 — Test WITHOUT a Client Cert (Should Fail)

```bash
docker compose exec client sh

# Inside client container:
curl --cacert /ssl/ca.crt https://server:443
# Expected FAILURE:
# curl: (56) OpenSSL SSL_read: error:... sslv3 alert certificate required
```

This confirms the server is enforcing mTLS — it rejected the request because
the client didn't present a cert.

## Step 5 — Generate the Client Certificate

```bash
# Back on the HOST in ssl/
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -subj "/CN=mtls-client"
openssl x509 -req \
  -in client.csr \
  -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client.crt \
  -days 365
```

## Step 6 — Test WITH a Client Cert (Should Succeed)

```bash
# Inside client container:
curl \
  --cacert /ssl/ca.crt \
  --cert /ssl/client.crt \
  --key /ssl/client.key \
  https://server:443

# Expected SUCCESS:
# <h1>mTLS Server — Auth OK</h1>
```

## Step 7 — Deep Inspection with openssl s_client

```bash
# Inside client container:
openssl s_client -connect server:443 \
  -CAfile /ssl/ca.crt \
  -cert /ssl/client.crt \
  -key /ssl/client.key

# Look for in the output:
# "Server certificate" — the server's cert details
# "Acceptable client certificate CA names: CN=mTLSRootCA" — server advertising what CA it trusts
# Verify return code: 0 (ok)
```

## Verification Checklist

- [ ] `curl --cacert ca.crt https://server:443` → rejected with "certificate required"
- [ ] `curl --cacert ca.crt --cert client.crt --key client.key https://server:443` → returns HTML
- [ ] `openssl s_client` shows both server cert chain and client cert were exchanged
- [ ] Modifying `client.crt` or using a cert signed by a different CA → rejected

## Bonus Challenge

Generate a **second client** cert signed by a **different CA** and confirm the server rejects it.

```bash
# Generate a rogue CA
openssl genrsa -out rogue-ca.key 2048
openssl req -new -x509 -days 365 -key rogue-ca.key -out rogue-ca.crt -subj "/CN=RogueCA"

# Generate a client cert signed by the rogue CA
openssl genrsa -out rogue-client.key 2048
openssl req -new -key rogue-client.key -out rogue-client.csr -subj "/CN=rogue"
openssl x509 -req -in rogue-client.csr -CA rogue-ca.crt -CAkey rogue-ca.key \
  -CAcreateserial -out rogue-client.crt -days 365

# Try to connect with the rogue cert — must be rejected
curl --cacert /ssl/ca.crt --cert /ssl/rogue-client.crt --key /ssl/rogue-client.key \
  https://server:443
```

## Key Lesson

> mTLS is the backbone of **zero-trust networking** in banking systems.
> Instead of "trust anyone inside the network", every service must prove its identity.
>
> In JPMC-style auth systems:
> - Each microservice has its own cert issued by an internal CA.
> - Services only talk to each other if both present valid certs.
> - A compromised service without a valid cert can't authenticate at all.
>
> `ssl_verify_client on` in nginx + a shared CA is the simplest form of this pattern.
