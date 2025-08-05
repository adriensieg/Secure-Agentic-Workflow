# Secure-Agentic-Workflow
Notes on how to secure Agentic Workflows

```mermaid
graph TD

    A0([Start])

    %% === LAYER 1: WHO DOES THE AGENT SERVE? ===
    A0 --> Q1["Who does the agent serve?"]
    Q1 --> Q1A["Individual User"]
    Q1 --> Q1B["Multiple Users / Org"]
    Q1 --> Q1C["System or Infrastructure"]

    %% === LAYER 2: WHERE DOES THE AGENT RUN? ===
    Q1A --> Q2A["Where does the agent run?"]
    Q1B --> Q2B["Where does the agent run?"]
    Q1C --> Q2C["Where does the agent run?"]

    Q2A --> L1["Browser or Edge"]
    Q2A --> L2["User's Device or VM"]
    Q2B --> L3["Backend / Cloud"]
    Q2C --> L4["Kubernetes / CI / Internal Infra"]

    %% === LAYER 3: WHAT DOES THE AGENT DO? ===
    L1 --> F1["Accesses personal APIs (e.g., email, calendar)"]
    L2 --> F2["Acts on user behalf (delegation/imitation)"]
    L3 --> F3["Calls multiple APIs or microservices"]
    L4 --> F4["Handles internal tasks / jobs / cron"]

    %% === LAYER 4: WHAT'S THE SECURITY RISK LEVEL? ===
    F1 --> R1["High Risk"]
    F1 --> R2["Medium Risk"]
    F2 --> R1
    F2 --> R2
    F3 --> R2
    F4 --> R3["Low Risk"]

    %% === LAYER 5: DECISION NODES (SECURITY DESIGN) ===
    R1 --> D1["Use OAuth2 + Phantom Token + DPoP"]
    R2 --> D2["Use OAuth2.1 + JWT + Scope Filtering"]
    R3 --> D3["Use SPIFFE / mTLS or Client Credentials"]

    D1 --> O1["Browser? Add Split Token"]
    D2 --> O2["Edge? Use audience-bound JWT"]
    D3 --> O3["Kubernetes? Use SPIRE / SPIFFE"]
    D3 --> O4["Cloud Infra? Use Workload Identity Federation"]

```
