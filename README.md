# PKI &amp; OAuth 2.0 Integration

> Generated an X.509 certificate chain with OpenSSL, then stood up an OAuth 2.0 flow against the
> Google Cloud API and used the access token to read a Google Sheet.

*CNIT 271 — Cybersecurity Fundamentals II, Lab 2.*

## Summary

Two foundational identity topics, hands-on: public-key certificates (what's actually in a private key
vs a public cert vs a PKCS#12 bundle) and the OAuth 2.0 authorization-code flow (authorization code
vs access token, front channel vs back channel).

## Lab setup

- Windows 10 VM, OpenSSL 3.x
- Google Cloud Console (OAuth consent + client ID), Postman
- Google Sheets API

## What I did &amp; found

**PKI with OpenSSL** — generated three files and can explain each:

| File | Purpose |
| --- | --- |
| `private.key` | Signs tokens, decrypts data, proves server identity — never shared |
| `public.cer` | Self-signed X.509 cert shared with clients to verify identity / encrypt to the server |
| `cert_key.p12` | PKCS#12 bundle of cert + key, password-protected, for importing into auth systems |

**OAuth 2.0** — configured a client ID/secret, ran the authorization-code flow in Postman
(`access_type=offline`, `prompt=consent`), retrieved an access token, and made an authenticated
`GET` to the Sheets API (`200 OK`).

Key distinctions I documented:
- **Auth code vs access token:** the code is single-use and exchanged for a token; server-to-server
  flows (or limited-input devices) can obtain a token without an interactive code.
- **Front vs back channel:** the front channel runs through the user's browser (authorization);
  the back channel is server-to-server (code → token exchange).

## Screenshots

See [`screenshots/`](screenshots/) for the OpenSSL commands, the OAuth client config, the token
retrieval, and the successful Sheets API call.

## Skills demonstrated

X.509 / PKI · OpenSSL · PKCS#12 · OAuth 2.0 authorization-code flow · API authentication ·
access tokens & scopes
