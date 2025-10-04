# M2M-AuthZ-with-OIDC
Service-to-Service Authentication with OAuth 2.0 and OpenID Connect


```mermaid
flowchart TD

A[Start: Need Authentication & Authorization] --> B{Is the actor a Human User or a Machine?}

%% HUMAN FLOW
B -->|Human User| C[Use OIDC (OAuth2 + ID Token)]
C --> D{Frontend or Backend?}
D -->|Frontend App| E[Use Authorization Code Flow + PKCE]
E --> F[Client registers redirect_uri & PKCE support]
F --> G[User logs in via Authorization Server (AS)]
G --> H[AS issues ID Token + Access Token]
H --> I[Backend validates ID Token (signature, nonce, exp, aud)]
I --> J[Access Token used to call protected APIs]
J --> K[APIs validate token (signature, audience, scopes)]
K --> L[✅ AuthN + AuthZ complete for user]

%% MACHINE FLOW
B -->|Machine (Service)| M[Use OAuth2 Client Credentials Flow]
M --> N{How does the service authenticate to AS?}
N -->|Private key (recommended)| O[Use JWT Client Assertion (private_key_jwt)]
O --> O1[Service signs JWT with private key: iss=sub=client_id, aud=token_endpoint, exp, jti]
O1 --> O2[AS verifies JWT signature using registered public key (JWKS)]
O2 --> O3[AS issues Access Token to service]
O3 --> P[Access Token audience = target service (e.g. Service B)]
P --> Q{Does target service need fine-grained control?}
Q -->|Yes| R[Use Scopes (e.g. summarize:write, email:send)]
Q -->|No| S[Use Audience-only access check]
R --> T[Target service validates token (sig, aud, scope, exp)]
S --> T
T --> U[✅ AuthN + AuthZ complete for service]

%% TOKEN VALIDATION STRATEGIES
U --> V{How to validate Access Token?}
V -->|JWT (self-contained)| W[Verify signature using AS public key (JWKS URI)]
V -->|Opaque token| X[Introspect via AS /introspect endpoint]
W --> Y[Check exp, aud, iss, scope claims]
X --> Y
Y --> Z[✅ Request authorized]

%% SECURE PRACTICES
Z --> Z1{Security Best Practices}
Z1 --> ZA[✅ Always use TLS]
Z1 --> ZB[✅ Use asymmetric keys (no client secrets in code)]
Z1 --> ZC[✅ Rotate keys via JWKS]
Z1 --> ZD[✅ Use short-lived tokens + refresh where needed]
Z1 --> ZE[✅ Validate nonce/audience/scope strictly]
Z1 --> ZF[✅ Log & monitor AS/token activity]

```
