# Scenario 02 — SAN Mismatch (Hostname Mismatch)

## Situation

A service's HTTPS endpoint was working fine. A team rotated the certificate and now
clients are getting SSL errors, even though the cert is valid and not expired.

## Setup

```bash
docker compose up -d --build
```

## Symptom

```bash
curl --cacert <(docker compose exec server cat /etc/nginx/ssl/ca.crt) https://localhost:9002
```

You will see:
```
curl: (60) SSL: no alternative certificate subject name matches target host name 'localhost'
```

## Your Mission

1. **Diagnose** the cert. What hostname is it actually valid for?
2. **Explain** why CN (Common Name) doesn't save you here. (Modern TLS only checks SAN)
3. **Fix it**: Regenerate the server cert with the correct SAN for `localhost`.
4. **Reload** nginx and verify.

## Hints

<details>
<summary>Hint 1 — Inspect the cert's SANs</summary>

```bash
docker compose exec server openssl x509 -in /etc/nginx/ssl/server.crt -noout -text \
  | grep -A2 "Subject Alternative Name"
```
</details>

<details>
<summary>Hint 2 — Generate a new cert with correct SAN</summary>

Inside the container:
```bash
# Create a SAN extension file
printf '[v3_req]\nsubjectAltName=DNS:localhost,IP:127.0.0.1\n' > /tmp/san.ext

# Re-sign the CSR with the correct SAN
openssl x509 -req \
  -in /etc/nginx/ssl/server.csr \
  -CA /etc/nginx/ssl/ca.crt \
  -CAkey /etc/nginx/ssl/ca.key \
  -CAcreateserial \
  -out /etc/nginx/ssl/server.crt \
  -days 365 \
  -extfile /tmp/san.ext \
  -extensions v3_req
```
</details>

## Verification

```bash
curl --cacert <(docker compose exec server cat /etc/nginx/ssl/ca.crt) https://localhost:9002
# Must return: <h1>Scenario 02 - SAN Mismatch</h1>
```

## Key Lesson

> Modern TLS clients (curl, browsers, Go's `http.Client`) completely ignore the `CN` field.
> They ONLY check the **Subject Alternative Name (SAN)** extension.
> Always include SANs when generating certs. A cert without SANs will fail on any modern client.
>
> In banking/auth systems, this is a common source of microservice-to-microservice failures
> after a certificate rotation.
