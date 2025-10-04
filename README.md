# M2M-AuthZ-with-OIDC
Service-to-Service Authentication with OAuth 2.0 and OpenID Connect


```mermaid
flowchart TD

A[Start: Need authentication/authorization between services] --> B{Is the caller a human user or a system?}

%% USER-FACING AUTHENTICATION FLOW
B -->|Human user| C[Use OpenID Connect Authorization Code Flow]
C --> C1[OIDC Authorization Code Flow:
 - User authenticates via Authorization Server (AS)
 - AS issues ID Token + Access Token to client]
C1 --> C2[Client verifies ID Token signature & claims]
C2 --> C3[Client uses Access Token to call protected APIs]
C3 --> C4[Resource Server validates Access Token (via JWKS / Introspection)]
C4 --> C5[Access granted if token valid + scopes OK]

%% MACHINE-TO-MACHINE AUTHENTICATION FLOW
B -->|System / Service| D{Does it act on behalf of a user?}
D -->|Yes| E[Use OAuth 2.0 On-Behalf-Of (OBO) Flow]
E --> E1[Service exchanges user's token for new token scoped to downstream API]
E1 --> E2[AS validates upstream token, issues new token for downstream audience]
E2 --> E3[Downstream service validates new token (audience, scope, signature)]
E3 --> E4[Access granted to downstream API]

D -->|No (pure system-to-system)| F[Use OAuth 2.0 Client Credentials Flow]
F --> F1[Service authenticates to AS using Client ID + Credential]
F1 --> F2{How does the service prove its identity?}

%% CLIENT AUTHENTICATION MODES
F2 -->|Shared Secret (client_secret_basic / post)| G[Use symmetric secret]
G --> G1[Less secure: same secret must be stored in every instance]
G1 --> G2[Recommended only for simple / non-critical environments]

F2 -->|Private Key JWT (RFC 7523)| H[JWT Client Authentication]
H --> H1[Service signs a JWT with its private key]
H1 --> H2[JWT includes:
 - iss = client_id
 - sub = client_id
 - aud = token_endpoint
 - exp, jti (for replay protection)]
H2 --> H3[AS verifies JWT signature with registered public key (via JWKS)]
H3 --> H4[AS issues Access Token with audience = downstream service]
H4 --> H5[Service uses token to call downstream API]
H5 --> H6[Downstream validates Access Token claims & signature]

%% TOKEN VALIDATION OPTIONS
H6 --> I{How does downstream validate tokens?}
I -->|Locally| J[Use JWKS URI to verify signature and claims]
I -->|Remotely| K[Call AS introspection endpoint]

J --> J1[Check:
 - Signature (RSA/ECDSA)
 - Expiration (exp)
 - Audience (aud)
 - Scope (scope)]
K --> K1[AS returns active=true if valid; includes claims/scopes]

%% TOKEN TYPES
J1 --> L{Which token type?}
L -->|JWT Access Token| M[Stateless validation]
L -->|Opaque Token| N[Requires introspection]

M --> O[Lightweight, good for distributed services]
N --> P[Centralized control, can revoke anytime]

%% KEY MANAGEMENT
O --> Q{How do clients get public keys?}
Q --> Q1[Use AS JWKS endpoint (rotation supported)]
Q1 --> Q2[Clients refresh keys periodically]
Q2 --> R[✅ End: secure distributed trust]

%% EXTRA PATHS
F1 -->|Needs fine-grained trust| S[Consider Mutual TLS (mTLS) client authentication]
S --> S1[mTLS proves client identity via certificate]
S1 --> S2[Strongest auth, but operationally complex]


```
