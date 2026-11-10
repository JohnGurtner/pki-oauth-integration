# PKI & OAuth 2.0 Integration

> Generated a private key, a self-signed X.509 certificate and a PKCS#12 bundle with OpenSSL, then
> set up an OAuth 2.0 authorization-code flow against Google Cloud and used the access token to read a
> Google Sheet through the API.

*CNIT 271 (Cybersecurity Fundamentals II), Lab 2.*

## Lab setup
- Windows 10 VM with OpenSSL 3.6 (Win64)
- Google Cloud Console (OAuth consent screen and client ID)
- Postman as the OAuth client
- Google Sheets API v4

## Procedure

### Part 1: Build the certificate material with OpenSSL
1. Installed OpenSSL for Windows and added its folder to the system environment variables so `openssl`
   runs from any prompt.
2. Generated the keys and certificates:

```bash
# 2048-bit RSA private key
openssl genrsa 2048 > private.key

# Self-signed X.509 certificate from that key, valid 1000 days
openssl req -new -x509 -nodes -sha1 -days 1000 -key private.key > public.cer

# Bundle cert + key into a password-protected PKCS#12 file
openssl pkcs12 -export -in public.cer -inkey private.key -out cert_key.p12
```

| File | What it is | Use in an identity/access server |
| --- | --- | --- |
| `private.key` | The RSA private key | Signs tokens, decrypts data, proves the server's identity. **Never leaves the server.** |
| `public.cer` | Self-signed X.509 cert: public key, subject info, validity period, signature | Given to clients so they can verify the server's signatures and encrypt data to it |
| `cert_key.p12` | PKCS#12 bundle of cert + key, password-protected | Imports the identity into an authentication system in one step |

> **Note:** The lab command used `-sha1`, which modern clients reject for certificate signatures.
> Real use should use `-sha256` and a CA-signed certificate rather than a self-signed one.

### Part 2: Register the OAuth application
1. Created a Google Cloud project and set up the **OAuth consent screen**, adding my account as a **test user**.
2. Created an **OAuth client ID** (web application) with Postman's callback as the **authorized redirect URI**.

### Part 3: Run the authorization-code flow
1. In Postman, set *Authorization → Type: OAuth 2.0* and filled in the client ID/secret, auth URL, token URL,
   callback URL and the Sheets scope.
2. Added two auth-request parameters:
   - `access_type=offline`, which asks for a **refresh token** as well as an access token
   - `prompt=consent`, which forces the consent screen so the refresh token is issued
3. Signed in with Google and granted access. Postman exchanged the authorization code for an **access token**.

### Part 4: Call the API
```
GET https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/Sheet1!A1:C3
Authorization: Bearer <access_token>
```
Result: **`200 OK`** with the contents of the 3×3 range. The bearer token carries the granted scope,
so no password is ever sent to the API.

## Key concepts

- **Authorization code vs access token.** The code is short-lived and single-use. It only gets
  exchanged for a token. The access token is the credential sent to the API. Some flows get a token
  with no code: **client credentials** (server to server, no user) and the **device flow** (TVs and
  other devices with limited input).
- **Front channel vs back channel.** The front channel runs through the user's browser (redirect to
  Google, sign-in, consent, code returned). The back channel is server to server (code plus client
  secret exchanged for a token), so the token never passes through the browser.

## Troubleshooting
The token request kept failing. I had deleted and recreated the OAuth client to fix its name, but
Postman still had the **old client ID and secret**. Copying the new client's credentials fixed it. A
deleted client's secret is dead immediately.

## Skills demonstrated

X.509 / PKI · OpenSSL · PKCS#12 · OAuth 2.0 authorization-code flow · refresh tokens & consent ·
scopes · bearer-token API authentication · Postman

## Screenshots

See [`screenshots/`](screenshots/) for the OpenSSL commands, the OAuth client configuration, the token
retrieval, and the successful Sheets API call.
