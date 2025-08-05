# Secure-Agentic-Workflow
Notes on how to secure Agentic Workflows

```mermaid
graph TD
    Start([Start: What does your AI agent do?])

    Start --> A1[Acts on behalf of a user (e.g., scheduling, email)]
    Start --> A2[Accesses multiple APIs on user/org behalf]
    Start --> A3[Runs internally (cron, CI/CD, backend job)]
    Start --> A4[Runs in browser or edge (no backend)]
    Start --> A5[Interacts with other AI agents]
    Start --> A6[Delegates access from user A to user B]
    Start --> A7[Accesses cloud resources without secrets]

    %% Delegated User Agent
    A1 --> B1[Need to impersonate or delegate user?]
    B1 --> C1[Impersonate → Use OAuth 2.0 + Phantom Token]
    B1 --> C2[Delegate → Use OAuth 2.1 + Rich Authorization Request]
    C1 --> C1a[Add DPoP or Split Token if high risk]

    %% Multi-API Agent
    A2 --> B2[Centralized policy and introspection needed?]
    B2 --> D1[Yes → Use OAuth 2.0 + API Gateway + Phantom Token]
    B2 --> D2[No → Use OAuth 2.0 + JWT + Scope Filtering]
    D2 --> D2a[Consider DPoP for token misuse prevention]

    %% Internal Automation Agent
    A3 --> E1[Where is it running?]
    E1 --> F1[Kubernetes → Use SPIFFE / SPIRE]
    E1 --> F2[Cloud (GCP, AWS, Azure) → Use Workload Identity Federation]
    E1 --> F3[Server VM → Use Client Credentials Grant + Short-lived JWT]

    %% Edge/Browser Agent
    A4 --> G1[Needs to protect tokens?]
    G1 --> H1[Yes → Use Split Token or DPoP]
    G1 --> H2[Add audience-bound JWT if possible]

    %% Multi-Agent System
    A5 --> I1[Do agents need to delegate to each other?]
    I1 --> J1[Yes → Use ZCAPs or Macaroons]
    I1 --> J2[No → Use SPIFFE / mTLS for mutual auth]

    %% Delegation Between Users
    A6 --> K1[Need user-granted delegation?]
    K1 --> L1[Yes → Use UMA 2.0 or OAuth RAR]
    L1 --> L1a[Ensure fine-grained consent logging]

    %% Cloud Access without Secrets
    A7 --> M1[Use Workload Identity Federation]
    M1 --> M1a[Use native metadata-based tokens]

```
