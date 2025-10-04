# M2M-AuthZ-with-OIDC
Service-to-Service Authentication with OAuth 2.0 and OpenID Connect


```mermaid
flowchart TD

A[Start: Need Authentication & Authorization] --> B{Is the actor a Human User or a Machine?}

%% HUMAN FLOW
B -->|Human User| C[Use OIDC with OAuth2 and ID Token]
C --> D{Frontend or Backend?}
D -->|Frontend App| E[Use Authorization Code Flow with PKCE]
E --> F[Client registers redirect_uri and PKCE support]
F --> G[User logs in via Authorization Server]
G --> H[AS issues ID Token and Access Token]
H --> I[Backend validates ID Token]
I --> I1[Check signature, nonce, exp, aud]
I1 --> J[Access Token used to call protected APIs]
J --> K[APIs validate token]
K --> K1[Check signature, audience, scopes]
K1 --> L[✅ AuthN + AuthZ complete for user]

%% MACHINE FLOW
B -->|Machine Service| M[Use OAuth2 Client Credentials Flow]
M --> N{How does the service authenticate to AS?}
N -->|Private key recommended| O[Use JWT Client Assertion]
O --> O1[Service signs JWT with private key]
O1 --> O1a[JWT contains: iss=sub=client_id, aud=token_endpoint, exp, jti]
O1a --> O2[AS verifies JWT signature using registered public key]
O2 --> O3[AS issues Access Token to service]
O3 --> P[Access Token audience = target service]
P --> Q{Does target service need fine-grained control?}
Q -->|Yes| R[Use Scopes for granular permissions]
R --> R1[Example: summarize:write, email:send]
Q -->|No| S[Use Audience-only access check]
R1 --> T[Target service validates token]
S --> T
T --> T1[Check sig, aud, scope, exp]
T1 --> U[✅ AuthN + AuthZ complete for service]

%% TOKEN VALIDATION STRATEGIES
U --> V{How to validate Access Token?}
V -->|JWT self-contained| W[Verify signature using AS public key from JWKS URI]
V -->|Opaque token| X[Introspect via AS introspect endpoint]
W --> Y[Check exp, aud, iss, scope claims]
X --> Y
Y --> Z[✅ Request authorized]

%% SECURE PRACTICES
Z --> Z1[Security Best Practices]
Z1 --> ZA[✅ Always use TLS]
Z1 --> ZB[✅ Use asymmetric keys, no client secrets in code]
Z1 --> ZC[✅ Rotate keys via JWKS]
Z1 --> ZD[✅ Use short-lived tokens, refresh where needed]
Z1 --> ZE[✅ Validate nonce, audience, scope strictly]
Z1 --> ZF[✅ Log and monitor AS and token activity]
```
