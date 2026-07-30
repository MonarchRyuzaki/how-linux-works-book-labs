# Scenario 01 — Expired Certificate

## Situation

You're on-call. An internal service starts returning TLS errors.
The service is running and nginx is up, but clients can't connect over HTTPS.

## Setup

```bash
docker compose up -d --build
```

## Symptom

```bash
openssl s_client -connect localhost:9001 -CAfile <(docker compose exec server cat /etc/nginx/ssl/ca.crt)
```

You will see:
```
verify error:num=10:certificate has expired
...
Verify return code: 10 (certificate has expired)
```

## Your Mission

1. **Confirm** the cert is expired. What command shows you the validity dates?
2. **Find** the exact expiry date and when it was issued.
3. **Fix it**: Renew the certificate (re-sign with the same CA) for 365 days.
4. **Reload** nginx and verify the handshake succeeds.

> ⚠️ The CA cert and CA key are inside the container at `/etc/nginx/ssl/`. You'll need them to re-sign.

## Hints

<details>
<summary>Hint 1 — How to inspect expiry</summary>

```bash
docker compose exec server openssl x509 -in /etc/nginx/ssl/server.crt -noout -dates
```
</details>

<details>
<summary>Hint 2 — How to renew (re-sign the existing CSR)</summary>

```bash
docker compose exec server sh -c '
  openssl x509 -req \
    -in /etc/nginx/ssl/server.csr \
    -CA /etc/nginx/ssl/ca.crt \
    -CAkey /etc/nginx/ssl/ca.key \
    -CAcreateserial \
    -out /etc/nginx/ssl/server.crt \
    -days 365
'
```
</details>

<details>
<summary>Hint 3 — Reload nginx</summary>

```bash
docker compose exec server nginx -s reload
```
</details>

## Verification

```bash
openssl s_client -connect localhost:9001 \
  -CAfile <(docker compose exec server cat /etc/nginx/ssl/ca.crt)
# Must end with: Verify return code: 0 (ok)
```

## Key Lesson

> In production, certificate expiry is one of the top causes of P1 outages.
> `openssl s_client -connect <host>:<port>` is your first move.
> Automate expiry checks: `openssl x509 -enddate -noout -in cert.crt`
