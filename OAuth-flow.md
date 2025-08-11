# End-to-End OAuth2 Authorization Code Flow with PKCE, MFA, Cookies (BFF Pattern) and Bearer Tokens (SPA Pattern) Using Azure Entra ID

```mermaid
sequenceDiagram
    autonumber
    participant User
    participant Browser
    participant SPA as Frontend-SPA
    participant Backend as Backend-BFF
    participant Session as Session-Store
    participant Azure as Azure-Entra-ID
    participant JWKS as Azure-JWKS
    participant API as Resource-API

    Note right of Browser: Cookie-based BFF flow

    Browser->>Backend: 1) GET /protected (no cookie)
    Backend-->>Browser: 2) Redirect to Azure authorize
    Browser->>Azure: 3) GET /authorize
    Azure-->>User: 4) Prompt login and MFA
    User-->>Azure: 5) Provide credentials
    Azure-->>Browser: 6) Redirect with auth code
    Browser->>Backend: 7) Callback with auth code
    Backend->>Azure: 8) Token request with code and client secret
    Azure-->>Backend: 9) Return access token, id token, refresh token
    Backend->>Session: 10) Store tokens and session id
    Backend-->>Browser: 11) Set cookie with session id (HttpOnly, Secure)

    Note over Browser,Backend: Subsequent requests include cookie automatically

    Browser->>Backend: 12) Request protected API with cookie
    Backend->>Session: 13) Retrieve stored tokens
    Backend->>JWKS: 14) Get public key and verify JWT
    JWKS-->>Backend: 15) Key returned
    Backend-->>Browser: 16) Return protected resource

    alt Access token expired
        Backend->>Azure: 17) Refresh token request
        Azure-->>Backend: 18) Return new tokens
        Backend->>Session: 19) Update stored tokens
        Backend-->>Browser: 20) Return protected resource
    end

    Note right of Browser: SPA Bearer token flow

    Browser->>SPA: 21) User clicks login
    SPA-->>Browser: 22) Redirect to Azure authorize with PKCE
    Browser->>Azure: 23) GET /authorize
    Azure-->>User: 24) Prompt login and MFA
    User-->>Azure: 25) Provide credentials
    Azure-->>Browser: 26) Redirect with auth code
    Browser->>SPA: 27) Callback with auth code
    SPA->>Azure: 28) Token request with code and code verifier
    Azure-->>SPA: 29) Return access token, id token, refresh token
    SPA->>API: 30) Request API with bearer access token
    API->>JWKS: 31) Get public key and verify JWT
    JWKS-->>API: 32) Key returned
    API-->>SPA: 33) Return protected resource

    alt Token expired
        SPA->>Azure: 34) Refresh token request
        Azure-->>SPA: 35) Return new tokens
        SPA->>API: 36) Request API with bearer access token
        API->>JWKS: 37) Verify JWT
        API-->>SPA: 38) Return protected resource
    end
```

- **<mark>Step 1</mark>**: Browser requests a **protected page/resource**.
   
- **<mark>Step 2</mark>**: Backend detects **no valid session cookie** → issues an **OAuth2 Authorization Request** (redirect to Azure /authorize). Important params:
    - `response_type`=code,
    - `client_id`,
    - `redirect_uri`,
    - `scope`=openid profile offline_access <api-scope>,
    - `state`,
    - `nonce`,
    - (for SPA add `code_challenge`/`PKCE`).

- **<mark>Step 3 to 5</mark>**: User authenticates at Azure Entra ID: username/password → MFA step (phone, push, TOTP, FIDO, etc.) as required by **tenant Conditional Access**. Azure then redirects back with an **authorization code**.

- **<mark>Step 6 to 8</mark>**: Token exchange (Backend/BFF): Backend exchanges code at Azure token endpoint with client_secret (confidential client) for access_token (JWT), id_token (JWT), and refresh_token (long, opaque string). Azure signs JWTs with RS256 and publishes keys via JWKS.

- **<mark>Step 9 to 10</mark>**: Backend stores tokens server-side (Session Store) and issues a HttpOnly Secure cookie to the browser: session_id=SESS123; HttpOnly; Secure; SameSite=Strict. Browser cannot read this cookie via JS (protects against XSS); cookie is auto-sent on future requests.

- **<mark>Step 11 to 16</mark>**: On each protected API request the backend reads session_id, pulls tokens from Session Store, validates the JWT (signature → get key from JWKS, then check iss, aud, exp, nbf, required scp/roles). If access token is valid, respond.

- **<mark>Step 17 to 19</mark>**: If the access token is expired, the backend uses the stored refresh_token to get a fresh access_token (token rotation usually returns a new refresh token). Update Session Store.

- **<mark>Step 21 to 26</mark>**: Logout: Backend removes session, optionally calls Azure revocation endpoint to revoke refresh_token, clears cookie, and may redirect user to Azure logout endpoint to clear SSO session.

- **<mark>Step 27 to 36</mark>**: SPA (Bearer) flow: SPA initiates the same /authorize redirect but includes PKCE (code_challenge). After Azure login + MFA, SPA receives a code, then directly POSTs to /token with code_verifier (no client_secret for public clients). Azure returns access_token (JWT), id_token, maybe refresh_token depending on configuration. SPA stores access token in memory (recommended). For API calls it sends Authorization: Bearer <access_token>.

- **<mark>Step 37 to 43</mark>**: Resource server verifies the JWT by fetching the JWKS (cache keys), validating the signature and claims. If expired, SPA uses refresh_token to obtain a new access token or reauthenticates.

