# Secure-Agentic-Workflow
Notes on how to secure Agentic Workflows

```mermaid
graph TD
    Start([Start])

    %% Layer 1: Who does the agent serve?
    Start --> ServeUser[Serves a user]
    Start --> ServeOrg[Serves the organization / itself]
    Start --> ServeAgents[Serves other agents / multi-agent collaboration]
    Start --> ServeCrossUser[Delegates between users]

    %% Layer 2: Where does it run?
    ServeUser --> UserEnv1[Runs in browser or edge]
    ServeUser --> UserEnv2[Runs on backend / cloud / internal]
    ServeOrg --> OrgEnv1[Kubernetes / microservice infra]
    ServeOrg --> OrgEnv2[Cloud VM / serverless]
    ServeOrg --> OrgEnv3[Edge or public-facing]
    ServeAgents --> AgentEnv1[Internal agent network]
    ServeAgents --> AgentEnv2[Cross-domain or federated]
    ServeCrossUser --> CrossEnv1[User-to-user sharing with consent]

    %% Layer 3: Risk level
    UserEnv1 --> RiskU1[Risk level? Low / Medium / High]
    UserEnv2 --> RiskU2[Risk level? Low / Medium / High]
    OrgEnv1 --> RiskO1[Risk level? Low / Medium / High]
    OrgEnv2 --> RiskO2[Risk level? Low / Medium / High]
    OrgEnv3 --> RiskO3[Risk level? Low / Medium / High]
    AgentEnv1 --> RiskA1[Risk level? Low / Medium / High]
    AgentEnv2 --> RiskA2[Risk level? Low / Medium / High]
    CrossEnv1 --> RiskC1[Risk level? Low / Medium / High]

    %% Decisions: Serve a user paths
    RiskU1 --> U1Low[Low risk → OAuth2 + JWT with narrow scopes]
    RiskU1 --> U1Med[Medium risk → OAuth2 + Phantom Token + API Gateway]
    RiskU1 --> U1High[High risk → OAuth2.1 / RAR + DPoP or Split Token + short-lived tokens]

    RiskU2 --> U2Low[Low risk → OAuth2 client credentials or delegated token]
    RiskU2 --> U2Med[Medium risk → OAuth2 + Phantom Token + introspection]
    RiskU2 --> U2High[High risk → OAuth2.1 RAR + DPoP + audit logging]

    %% Organization/self agent paths
    RiskO1 --> O1Low[Low risk → SPIFFE identity + mTLS]
    RiskO1 --> O1Med[Medium risk → SPIFFE + short-lived certificates + audit]
    RiskO1 --> O1High[High risk → SPIFFE + ZCAPs for delegation + strict policy enforcement]

    RiskO2 --> O2Low[Low risk → Workload Identity Federation]
    RiskO2 --> O2Med[Medium risk → Workload Identity + scoped short creds]
    RiskO2 --> O2High[High risk → Workload Identity + DPoP-like binding + monitoring]

    RiskO3 --> O3Low[Low risk → Split Token]
    RiskO3 --> O3Med[Medium risk → Split Token + audience-bound JWT]
    RiskO3 --> O3High[High risk → Split Token + DPoP + token replay protection]

    %% Multi-agent collaboration
    RiskA1 --> A1Low[Low risk → SPIFFE for mutual auth]
    RiskA1 --> A1Med[Medium risk → SPIFFE + Macaroons for scoped delegation]
    RiskA1 --> A1High[High risk → SPIFFE + ZCAPs chain + audit + revocation]

    RiskA2 --> A2Low[Low risk → Federated capability exchange]
    RiskA2 --> A2Med[Medium risk → DID-backed ZCAPs or Macaroons]
    RiskA2 --> A2High[High risk → Cross-domain ZCAPs + proof-of-possession + verification chains]

    %% Delegation between users
    RiskC1 --> C1Low[Low risk → UMA 2.0 light delegation]
    RiskC1 --> C1Med[Medium risk → OAuth RAR with consent logging]
    RiskC1 --> C1High[High risk → UMA 2.0 + RAR + audit trail + time limits]

    %% Fallback / enhancements (common augmentations)
    U1High --> Enhance1[Add revocation endpoint, session binding, logging]
    U2High --> Enhance2[Add anomaly detection, user re-consent]
    O1High --> Enhance3[Policy engine + dynamic authorization]
    A1High --> Enhance4[Delegation expiration, capability revocation]
```

## Primary Decision Paths

# Agent Authentication Decision Tree

| Who Serves                     | Environment             | Risk Level | Authentication Solution                                        | Enhancements                                            | **Example AI Agent Use Case**                                                                                                                                        |
| ------------------------------ | ----------------------- | ---------- | -------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Serves a User**              | Browser/Edge            | Low        | OAuth2 + JWT (narrow scopes)                                   | Token refresh rotation                                  | **Calendar scheduling agent** that lives in browser sidebar, books lunch meetings by reading/writing only the user’s work calendar — cannot touch email or contacts. |
| **Serves a User**              | Browser/Edge            | Medium     | OAuth2 + Phantom Token + API Gateway + CIBA                    | Rate limiting, CORS, push notifications                 | **Browser-based email triage agent** that drafts replies inside Gmail UI — phantom token hides credentials so compromised browser can’t leak sensitive backend keys. |
| **Serves a User**              | Browser/Edge            | High       | OAuth2.1/RAR + DPoP or Split Token + short-lived tokens + CIBA | Revocation endpoint, session binding, anomaly detection | **Mobile banking chatbot** that executes payments — requires proof-of-possession, very short token lifetime, and per-transaction permission scopes.                  |
| **Serves a User**              | Backend/Cloud/Internal  | Low        | OAuth2 client credentials or delegated token                   | Basic token rotation                                    | **Reminder notification agent** running in backend to send SMS alerts based on user’s preferences — simple service account auth.                                     |
| **Serves a User**              | Backend/Cloud/Internal  | Medium     | OAuth2 + Phantom Token + introspection + CIBA                  | Request signing, IP whitelisting, async auth flows      | **Travel booking AI** that compares flights, books tickets, and manages itinerary from backend — tokens are introspected by API gateway before use.                  |
| **Serves a User**              | Backend/Cloud/Internal  | High       | OAuth2.1 RAR + DPoP + audit logging + CIBA                     | Anomaly detection, user re-consent, behavioral analysis | **Robo-advisor AI** making trades — every action is recorded in audit logs with per-asset-class permissions and real-time risk monitoring.                           |
| **Serves Organization/Itself** | Kubernetes/Microservice | Low        | SPIFFE identity + mTLS                                         | Cert rotation, health checks                            | **Data cleanup microservice** in a service mesh — communicates internally to purge stale logs using workload-based identity.                                         |
| **Serves Organization/Itself** | Kubernetes/Microservice | Medium     | SPIFFE + short-lived certs + audit                             | Namespace isolation, RBAC                               | **Model retraining pipeline** — pulls training data and pushes updated model images into production, with short-lived service identity.                              |
| **Serves Organization/Itself** | Kubernetes/Microservice | High       | SPIFFE + ZCAPs + strict policy                                 | Dynamic authz, capability revocation                    | **Compliance AI** that queries sensitive records — capabilities are revocable at runtime based on audit findings.                                                    |
| **Serves Organization/Itself** | Cloud VM/Serverless     | Low        | Workload Identity Federation                                   | Resource tagging, monitoring                            | **Image processing function** triggered by user uploads — authenticates to cloud storage without static keys.                                                        |
| **Serves Organization/Itself** | Cloud VM/Serverless     | Medium     | Workload Identity + scoped short creds                         | VPC restriction, TLS                                    | **ETL AI agent** that runs hourly, pulling customer data, transforming it, and storing it in analytics DB with least privilege.                                      |
| **Serves Organization/Itself** | Cloud VM/Serverless     | High       | Workload Identity + DPoP-like binding + monitoring             | Behavioral analysis, alerting, failover                 | **Autonomous trading engine** — cloud identity is bound to specific execution environment, detects and halts abnormal trade bursts.                                  |
| **Serves Organization/Itself** | Edge/Public-facing      | Low        | Split Token                                                    | CDN integration, basic caching                          | **Public-facing content summarization bot** that fetches and summarizes news articles — edge server uses reference token only.                                       |
| **Serves Organization/Itself** | Edge/Public-facing      | Medium     | Split Token + audience-bound JWT                               | Geo restrictions, rate limiting                         | **Public API gateway for AI inference** — tokens are bound to region-specific users to comply with data laws.                                                        |
| **Serves Organization/Itself** | Edge/Public-facing      | High       | Split Token + DPoP + anti-replay                               | Fraud detection                                         | **Edge-based payment authorization agent** — signs each request to prevent replay attacks during high-value transactions.                                            |
| **Serves Other Agents**        | Internal Agent Network  | Low        | SPIFFE for mutual auth                                         | Load balancing, service discovery                       | **Document indexing swarm** — multiple agents crawl, parse, and share search indexes internally with mTLS mutual trust.                                              |
| **Serves Other Agents**        | Internal Agent Network  | Medium     | SPIFFE + Macaroons                                             | Delegation logging, task queuing                        | **Multi-agent RAG pipeline** — retrieval agents pass limited search capabilities to summarization agents for controlled processing.                                  |
| **Serves Other Agents**        | Internal Agent Network  | High       | SPIFFE + ZCAPs chain + audit                                   | Delegation expiration, capability revocation            | **Real-time fraud detection agent mesh** — delegation chains with instant kill-switch if a compromised agent is detected.                                            |
| **Serves Other Agents**        | Cross-domain/Federated  | Low        | Federated capability exchange                                  | Trust anchors                                           | **Inter-company supplier info bot** — shares only allowed inventory metadata between trusted partners.                                                               |
| **Serves Other Agents**        | Cross-domain/Federated  | Medium     | DID-backed ZCAPs or Macaroons                                  | Identity verification                                   | **Logistics tracking AI agents** — each step in supply chain passes verified updates without exposing full datasets.                                                 |
| **Serves Other Agents**        | Cross-domain/Federated  | High       | Cross-domain ZCAPs + PoP                                       | Multi-party verification                                | **International trade settlement AI agents** — cryptographic proofs and dispute resolution workflows for cross-border payments.                                      |
| **Delegates Between Users**    | User-to-user            | Low        | UMA 2.0 light delegation + CIBA                                | Consent timers                                          | **Shared playlist AI** — user grants another user’s agent temporary playlist edit rights.                                                                            |
| **Delegates Between Users**    | User-to-user            | Medium     | OAuth RAR + consent logging + CIBA                             | Notification system, consent audit                      | **Contract review assistant** — allows another user’s legal AI to review documents with line-by-line restrictions.                                                   |
| **Delegates Between Users**    | User-to-user            | High       | UMA 2.0 + RAR + audit trail + CIBA                             | Automated expiration, compliance                        | **Medical AI assistant** — patient grants doctor’s AI time-limited access to imaging scans for consultation.                                                         |

## Enhancement Patterns

| Base Solution | Enhancement | Example Application |
|---------------|-------------|-------------------|
| OAuth2.1/RAR + DPoP (High Risk User) | Add revocation endpoint, session binding, logging | Banking app with instant token revocation and session monitoring |
| OAuth2.1 RAR + DPoP (High Risk Backend) | Add anomaly detection, user re-consent | Investment platform detecting unusual trading patterns |
| SPIFFE + ZCAPs (High Risk Org) | Policy engine + dynamic authorization | Healthcare system with dynamic access control based on patient context |
| SPIFFE + ZCAPs chain (High Risk Agents) | Delegation expiration, capability revocation | Autonomous vehicle fleet with time-limited agent interactions |

## Key Terms

- **OAuth2.1/RAR**: Rich Authorization Requests for fine-grained permissions
- **DPoP**: Demonstration of Proof-of-Possession to bind tokens to clients
- **SPIFFE**: Secure Production Identity Framework for Everyone
- **ZCAPs**: Authorization Capabilities for delegation chains
- **Macaroons**: Bearer tokens with embedded caveats for attenuation
- **UMA 2.0**: User-Managed Access for user-controlled authorization
- **Phantom Token**: Opaque reference token that hides actual JWT from client
## Key Terms

- **OAuth2.1/RAR**: Rich Authorization Requests for fine-grained permissions
- **DPoP**: Demonstration of Proof-of-Possession to bind tokens to clients
- **SPIFFE**: Secure Production Identity Framework for Everyone
- **ZCAPs**: Authorization Capabilities for delegation chains
- **Macaroons**: Bearer tokens with embedded caveats for attenuation
- **UMA 2.0**: User-Managed Access for user-controlled authorization
- **Phantom Token**: Opaque reference token that hides actual JWT from client

## Decision Tree Structure

### Layer 1: Primary Service Model
- **Serves a user**: Agent acts on behalf of individual users
- **Serves the organization / itself**: Agent operates for organizational needs or autonomous operations
- **Serves other agents / multi-agent collaboration**: Agent participates in multi-agent systems
- **Delegates between users**: Agent facilitates user-to-user interactions with proper consent

### Layer 2: Runtime Environment Categories

#### For User-Serving Agents:
- **Runs in browser or edge**: Client-side execution, limited trust environment
- **Runs on backend / cloud / internal**: Server-side execution, controlled environment

#### For Organization-Serving Agents:
- **Kubernetes / microservice infra**: Container orchestration, service mesh environments
- **Cloud VM / serverless**: Virtual machines or serverless computing platforms
- **Edge or public-facing**: Deployed at network edge or public interfaces

#### For Multi-Agent Systems:
- **Internal agent network**: Agents within same trust domain
- **Cross-domain or federated**: Agents across different trust boundaries

#### For User Delegation:
- **User-to-user sharing with consent**: Facilitating controlled sharing between users

### Layer 3: Risk Assessment
Each environment path branches into three risk levels:
- **Low Risk**: Basic security requirements, standard threat model
- **Medium Risk**: Elevated security needs, additional protections required
- **High Risk**: Critical security requirements, comprehensive protection needed


```mermaid
sequenceDiagram
    autonumber
    %% Participants
    participant User
    participant Browser
    participant SPA as Frontend (SPA JS)
    participant Backend as Backend (Python / Confidential / BFF)
    participant Session as Session Store (DB)
    participant Azure as Azure Entra ID (IdP)
    participant JWKS as Azure JWKS (keys)
    participant API as Resource API (Backend or separate)

    Note over Browser,Backend: --- User tries to access a protected resource ---

    %% ===== Cookie-based (Backend/BFF) =====
    Note right of Browser: Cookie-based / BFF flow (Backend does token exchange; browser gets HttpOnly session cookie)
    Browser->>Backend: 1) GET /protected (no session cookie)
    Backend-->>Browser: 2) 302 Redirect to Azure /authorize?response_type=code&client_id=CFG&redirect_uri=https://app/callback&scope=openid%20profile%20offline_access%20api.read&state=S&nonce=N
    Browser->>Azure: 3) GET /authorize (user lands on IdP)
    Note right of Azure: 4) Azure shows login UI -> user enters credentials -> Azure triggers MFA (conditional access)
    Azure-->>Browser: 5) 302 Redirect to https://app/callback?code=AUTH_CODE&state=S
    Browser->>Backend: 6) GET /callback?code=AUTH_CODE&state=S
    Backend->>Azure: 7) POST /token  (grant_type=authorization_code, code=AUTH_CODE, redirect_uri=..., client_id, client_secret[, pkce_verifier])
    Azure-->>Backend: 8) 200 { access_token: <JWT>, id_token: <JWT>, refresh_token: <opaque>, expires_in:3600, token_type:Bearer }
    Backend->>Session: 9) Store session: { session_id: SESS123, access_token, refresh_token, user_claims_from_id_token, expires_at }
    Backend-->>Browser: 10) Set-Cookie: session_id=SESS123; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
    Note over Browser,Backend: 11) Subsequent browser requests automatically send cookie

    Browser->>Backend: 12) GET /api/data (Cookie: session_id=SESS123)
    Backend->>Session: 13) lookup SESS123 -> get access_token + expiry
    alt access_token still valid
        Backend->>JWKS: 14) (cached) get public key by kid ; verify JWT signature & claims (iss,aud,exp,scope)
        JWKS-->>Backend: 15) public key
        Backend-->>Browser: 16) 200 {protected resource}
    else access_token expired
        Backend->>Azure: 17) POST /token (grant_type=refresh_token, refresh_token=..., client_id, client_secret)
        Azure-->>Backend: 18) 200 { new access_token, new refresh_token, expires_in }
        Backend->>Session: 19) update session tokens
        Backend-->>Browser: 20) 200 {protected resource}
    end

    %% Logout / Revoke for Cookie flow
    Browser->>Backend: 21) POST /logout (Cookie: SESS123)
    Backend->>Session: 22) delete session SESS123 (remove tokens)
    Backend->>Azure: 23) POST /revoke (token=refresh_token OR token=access_token)  -- optional
    Azure-->>Backend: 24) 200 OK
    Backend-->>Browser: 25) Set-Cookie: session_id=; Max-Age=0; Path=/; HttpOnly; Secure
    Backend-->>Browser: 26) Redirect -> Azure logout endpoint (end SSO) optionally

    %% ===== Bearer Token (SPA) =====
    Note right of SPA: Bearer token / SPA flow (PKCE). SPA exchanges code -> keeps access token in memory (or localStorage, less safe).
    Browser->>SPA: 27) User clicks "Login" in SPA
    SPA-->>Browser: 28) 302 Redirect to Azure /authorize?response_type=code&client_id=SPA_CLIENT&redirect_uri=https://spa/callback&scope=openid%20profile%20offline_access%20api.read&state=S2&nonce=N2&code_challenge=CH&code_challenge_method=S256
    Browser->>Azure: 29) GET /authorize (SPA)
    Note right of Azure: 30) Azure login + MFA
    Azure-->>Browser: 31) 302 Redirect to https://spa/callback?code=AUTH_CODE_SP&state=S2
    Browser->>SPA: 32) GET /callback?code=AUTH_CODE_SP&state=S2
    SPA->>Azure: 33) POST /token (grant_type=authorization_code, code=AUTH_CODE_SP, redirect_uri=https://spa/callback, client_id=SPA_CLIENT, code_verifier=VER)
    Azure-->>SPA: 34) 200 { access_token: <JWT>, id_token: <JWT>, refresh_token: <rotating?>, expires_in }
    Note over SPA: 35) Store access_token in memory (recommended); if storing anywhere persistent -> risk of XSS
    SPA->>API: 36) GET /api/data Authorization: Bearer <access_token>
    API->>JWKS: 37) fetch jwks_uri (cached) -> select key by kid -> verify JWT signature and claims (iss,aud,exp,scope,azp)
    JWKS-->>API: 38) public key
    alt token valid
        API-->>SPA: 39) 200 { protected resource }
    else token expired
        SPA->>Azure: 40) POST /token (grant_type=refresh_token, refresh_token=...)
        Azure-->>SPA: 41) 200 { new access_token, new_refresh_token? }
        SPA->>API: 42) GET /api/data Authorization: Bearer <new_access_token>
        API->>JWKS: 43) verify and respond 200
    end

    %% Error / Invalid refresh handling
    Note right of SPA: If refresh fails (invalid_grant) -> SPA must re-initiate login (redirect to /authorize)

    %% Token inspection & shape notes
    Note over Azure,Backend: Token Response sample (abbrev): 
      {"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI...","id_token":"eyJ...","refresh_token":"0.ABc..."}
    Note over API,Backend: JWT decoded payload sample (abbrev):
      {"iss":"https://login.microsoftonline.com/{tid}/v2.0","aud":"api-client-id","sub":"user-id","scp":"user.read api.read","iat":169..., "exp":169...,"amr":["pwd","mfa"]}
```

