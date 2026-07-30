# Scenario 05 — Wrong Certificate Format (DER vs PEM)

## Situation

A new automation script exported a certificate from an internal PKI system and deployed it.
nginx refuses to start. The cert file exists and looks fine in `ls`.

## Setup

```bash
docker compose up -d --build
```

## Symptom

Check container logs:
```bash
docker compose logs server
```

You will see:
```
nginx: [emerg] PEM_read_bio_X509_AUX() failed
(PEM routines:PEM_read_bio:no start line)
```

## Your Mission

1. **Determine** the format of `server.crt`. Is it PEM or DER?
   (Hint: `cat` it and see if it starts with `-----BEGIN CERTIFICATE-----`)

2. **Explain** the difference between PEM and DER formats.

3. **Convert** the cert to PEM format in-place.

4. **Restart** nginx and verify HTTPS works.

## Hints

<details>
<summary>Hint 1 — Check the file format</summary>

```bash
# PEM files are base64-encoded text — readable with cat
# DER files are raw binary — cat outputs garbage
docker compose exec server cat /etc/nginx/ssl/server.crt

# Use file to detect format
docker compose exec server file /etc/nginx/ssl/server.crt
# DER: "Certificate, Version=3"  (no "PEM" in output)
```
</details>

<details>
<summary>Hint 2 — Convert DER → PEM</summary>

```bash
docker compose exec server sh -c '
  openssl x509 \
    -in /etc/nginx/ssl/server.crt \
    -inform DER \
    -out /etc/nginx/ssl/server.crt \
    -outform PEM
'
```
</details>

<details>
<summary>Hint 3 — Restart nginx</summary>

```bash
# nginx is not running (it crashed on startup), so use 'docker compose restart'
docker compose restart server
```
</details>

## Verification

```bash
# First confirm the cert is now PEM
docker compose exec server head -1 /etc/nginx/ssl/server.crt
# Must output: -----BEGIN CERTIFICATE-----

# Then test HTTPS
curl -k https://localhost:9005
# Expected: <h1>Scenario 05 - Wrong Format</h1>
```

## Key Lesson

> **PEM** = Privacy Enhanced Mail. Base64-encoded text, human-readable.
> Wrapped in `-----BEGIN CERTIFICATE-----` / `-----END CERTIFICATE-----` headers.
> Used by: nginx, Apache, curl, most Linux tools.
>
> **DER** = Distinguished Encoding Rules. Raw binary.
> Used by: Java (JKS/PKCS12), Windows (.cer), some HSMs and PKI systems.
>
> When a PKI exports a cert, always verify the format before deploying.
> `file server.crt` and `head -1 server.crt` are your quick checks.
