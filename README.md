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

# Agent Authentication Decision Tree

| Who Serves | Environment | Risk Level | Authentication Solution | Enhancements | Example Use Case |
|------------|-------------|------------|------------------------|--------------|------------------|
| **Serves a User** | Browser/Edge | Low | OAuth2 + JWT with narrow scopes | Token refresh rotation | Smart assistant on phone scheduling meetings - uses OAuth to access calendar with read/write scope only |
| **Serves a User** | Browser/Edge | Medium | OAuth2 + Phantom Token + API Gateway | Rate limiting, CORS protection | Email assistant plugin in Gmail - phantom token hides actual credentials from browser |
| **Serves a User** | Browser/Edge | High | OAuth2.1/RAR + DPoP or Split Token + short-lived tokens | Revocation endpoint, session binding, anomaly detection | Banking chatbot in mobile app - requires proof-of-possession and fine-grained permissions |
| **Serves a User** | Backend/Cloud/Internal | Low | OAuth2 client credentials or delegated token | Basic token rotation | Simple notification service - uses service account to send emails on user's behalf |
| **Serves a User** | Backend/Cloud/Internal | Medium | OAuth2 + Phantom Token + introspection | Request signing, IP whitelisting | LLM agent booking travel on behalf of user - backend validates tokens via introspection |
| **Serves a User** | Backend/Cloud/Internal | High | OAuth2.1 RAR + DPoP + audit logging | Anomaly detection, user re-consent, behavioral analysis | Financial advisor AI making investment decisions - requires detailed permissions and full audit trail |
| **Serves Organization/Itself** | Kubernetes/Microservice | Low | SPIFFE identity + mTLS | Certificate rotation, health checks | Cron job backing up databases - uses service mesh identity for internal communication |
| **Serves Organization/Itself** | Kubernetes/Microservice | Medium | SPIFFE + short-lived certificates + audit | Namespace isolation, RBAC integration | CI/CD pipeline deploying to production - certificates expire quickly with deployment logging |
| **Serves Organization/Itself** | Kubernetes/Microservice | High | SPIFFE + ZCAPs for delegation + strict policy enforcement | Policy engine, dynamic authorization, capability revocation | Compliance agent accessing sensitive customer data - capability-based auth with strict policies |
| **Serves Organization/Itself** | Cloud VM/Serverless | Low | Workload Identity Federation | Resource tagging, basic monitoring | Backend job processing uploaded images - uses cloud provider's workload identity |
| **Serves Organization/Itself** | Cloud VM/Serverless | Medium | Workload Identity + scoped short credentials | VPC restrictions, encryption in transit | Data pipeline ETL process - short-lived credentials with specific resource access |
| **Serves Organization/Itself** | Cloud VM/Serverless | High | Workload Identity + DPoP-like binding + monitoring | Behavioral analysis, real-time alerting, automatic failover | Automated trading system - credentials bound to specific workload with anomaly detection |
| **Serves Organization/Itself** | Edge/Public-facing | Low | Split Token | Basic caching, CDN integration | Content delivery agent - reference token at edge, validation token in backend |
| **Serves Organization/Itself** | Edge/Public-facing | Medium | Split Token + audience-bound JWT | Geographic restrictions, rate limiting | API gateway rate limiting - tokens bound to specific audiences/endpoints |
| **Serves Organization/Itself** | Edge/Public-facing | High | Split Token + DPoP + token replay protection | Anti-replay nonces, request signing, fraud detection | Payment processing edge service - prevents token replay attacks |
| **Serves Other Agents** | Internal Agent Network | Low | SPIFFE for mutual auth | Load balancing, service discovery | Internal agents sharing cached data - mutual TLS authentication between services |
| **Serves Other Agents** | Internal Agent Network | Medium | SPIFFE + Macaroons for scoped delegation | Task queuing, delegation logging | Workflow orchestration - agents delegate specific tasks to other agents |
| **Serves Other Agents** | Internal Agent Network | High | SPIFFE + ZCAPs chain + audit + revocation | Delegation expiration, capability revocation, real-time monitoring | Multi-agent fraud detection system - capability chains with full audit and instant revocation |
| **Serves Other Agents** | Cross-domain/Federated | Low | Federated capability exchange | Trust anchors, basic verification | Cross-company data sharing agents - simple capability exchange between trusted domains |
| **Serves Other Agents** | Cross-domain/Federated | Medium | DID-backed ZCAPs or Macaroons | Identity verification, capability attenuation | Supply chain tracking agents - decentralized identity with capability delegation |
| **Serves Other Agents** | Cross-domain/Federated | High | Cross-domain ZCAPs + proof-of-possession + verification chains | Cryptographic proofs, multi-party verification, dispute resolution | Multi-party financial settlement agents - cryptographic proof chains across organizations |
| **Delegates Between Users** | User-to-user sharing | Low | UMA 2.0 light delegation | Basic consent tracking, expiration timers | Photo sharing app - simple user consent for album access |
| **Delegates Between Users** | User-to-user sharing | Medium | OAuth RAR with consent logging | Granular permissions, consent history, notification system | Document collaboration tool - fine-grained permissions with consent audit |
| **Delegates Between Users** | User-to-user sharing | High | UMA 2.0 + RAR + audit trail + time limits | Full audit trails, automated expiration, compliance reporting | Healthcare record sharing - detailed permissions, full audit, time-bounded access |

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
flowchart TB
    subgraph CookieFlow["Cookie-Based OAuth2 Flow"]
        A1[Browser visits /protected] --> A2[Backend checks session cookie]
        A2 -->|No cookie| A3[Redirect to Azure Entra ID login]
        A3 --> A4[Azure login + MFA]
        A4 --> A5[Redirect back to backend with auth code]
        A5 --> A6[Backend exchanges code for Access + ID + Refresh Tokens]
        A6 --> A7[Backend stores tokens in server session store]
        A7 --> A8[Backend sets HttpOnly Secure Cookie with session ID]
        A8 --> A9[Browser automatically sends cookie on every request]
        A9 --> A10[Backend retrieves tokens from session and verifies JWT]
        A10 --> A11[Serve protected resource]
    end

    subgraph BearerFlow["Bearer Token OAuth2 Flow"]
        B1[Browser visits /protected] --> B2[Backend says: Need Authorization]
        B2 --> B3[Browser redirects user to Azure login]
        B3 --> B4[Azure login + MFA]
        B4 --> B5[Redirect back to frontend with auth code]
        B5 --> B6[Frontend exchanges code (PKCE) directly with Azure for tokens]
        B6 --> B7[Frontend stores Access Token in memory/localStorage]
        B7 --> B8[Frontend sends token in Authorization header: Bearer &lt;token&gt;]
        B8 --> B9[Backend verifies JWT signature + claims]
        B9 --> B10[Serve protected resource]
    end
```

