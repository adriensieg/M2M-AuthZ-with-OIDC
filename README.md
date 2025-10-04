# M2M-AuthZ-with-OIDC
Service-to-Service Authentication with OAuth 2.0 and OpenID Connect


```mermaid
flowchart TD
    A[Start: Need authentication/authorization between services] --> B{Is the caller a human user or a system?}

    %% USER-FACING AUTHENTICATION FLOW
    B -->|Human user| C[Use OpenID Connect Authorization Code Flow]
    C --> C1[OIDC Authorization Code Flow]
    C1 --> C1a[User authenticates via Authorization Server]
    C1a --> C1b[AS issues ID Token + Access Token to client]
    C1b --> C2[Client verifies ID Token signature & claims]
    C2 --> C3[Client uses Access Token to call protected APIs]
    C3 --> C4[Resource Server validates Access Token]
    C4 --> C5[Access granted if token valid + scopes OK]

    %% MACHINE-TO-MACHINE AUTHENTICATION FLOW
    B -->|System / Service| D{Does it act on behalf of a user?}
    D -->|Yes| E[Use OAuth 2.0 On-Behalf-Of Flow]
    E --> E1[Service exchanges user token for new token]
    E1 --> E2[AS validates upstream token, issues new token]
    E2 --> E3[Downstream service validates new token]
    E3 --> E4[Access granted to downstream API]

    D -->|No: pure system-to-system| F[Use OAuth 2.0 Client Credentials Flow]
    F --> F1[Service authenticates to AS using Client ID + Credential]
    F1 --> F2{How does the service prove its identity?}

    %% CLIENT AUTHENTICATION MODES
    F2 -->|Shared Secret| G[Use symmetric secret]
    G --> G1[Less secure: same secret stored everywhere]
    G1 --> G2[Recommended only for simple environments]

    F2 -->|Private Key JWT| H[JWT Client Authentication]
    H --> H1[Service signs JWT with private key]
    H1 --> H2[JWT includes: iss, sub, aud, exp, jti]
    H2 --> H3[AS verifies JWT signature with public key]
    H3 --> H4[AS issues Access Token with audience]
    H4 --> H5[Service uses token to call downstream API]
    H5 --> H6[Downstream validates Access Token]

    %% TOKEN VALIDATION OPTIONS
    H6 --> I{How does downstream validate tokens?}
    I -->|Locally| J[Use JWKS URI to verify signature]
    I -->|Remotely| K[Call AS introspection endpoint]

    J --> J1[Check: signature, exp, aud, scope]
    K --> K1[AS returns active status and claims]

    %% TOKEN TYPES
    J1 --> L{Which token type?}
    L -->|JWT Access Token| M[Stateless validation]
    L -->|Opaque Token| N[Requires introspection]

    M --> O[Lightweight, good for distributed services]
    N --> P[Centralized control, can revoke anytime]

    %% KEY MANAGEMENT
    O --> Q{How do clients get public keys?}
    Q --> Q1[Use AS JWKS endpoint with rotation support]
    Q1 --> Q2[Clients refresh keys periodically]
    Q2 --> R[✅ End: secure distributed trust]

    %% EXTRA PATHS
    F1 -.->|Needs fine-grained trust| S[Consider Mutual TLS authentication]
    S --> S1[mTLS proves identity via certificate]
    S1 --> S2[Strongest auth, operationally complex]
```
