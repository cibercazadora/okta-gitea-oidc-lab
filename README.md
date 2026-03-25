# Okta + Gitea OIDC Federation Lab
 
A hands-on identity federation lab connecting Okta as an OIDC identity provider to a self-hosted Gitea instance. Built to demonstrate real-world SSO configuration, token flow, and identity provider integration patterns used in enterprise environments.
 
---
 
## The Problem This Solves
 
Most identity federation documentation assumes you already have an enterprise environment to test in. This lab gives you a fully functional OIDC federation setup on your local machine using free tools, so you can practice and demonstrate SSO configuration without needing access to a corporate IdP or paid SaaS tier.
 
---
 
## Architecture
 
```
User Browser
      |
      v
Gitea (localhost:3000)       <-- Service Provider (SP)
      |
      | OIDC Authorization Request
      v
Okta Developer Tenant        <-- Identity Provider (IdP)
      |
      | Authorization Code
      v
Gitea exchanges code for tokens
      |
      | ID Token (JWT)
      v
User authenticated in Gitea via Okta identity
```
 
**Protocol:** OpenID Connect (OIDC) Authorization Code Flow
 
**Identity Provider:** Okta Developer Tenant (free)
 
**Service Provider:** Gitea (self-hosted via Docker)
 
---
 
## What the OIDC Flow Looks Like
 
1. User clicks "Sign in with Okta" on Gitea login page
2. Gitea redirects browser to Okta with an authorization request
3. User authenticates with Okta credentials
4. Okta returns an authorization code to Gitea's callback URL
5. Gitea exchanges the code for an ID token and access token
6. Gitea reads the ID token claims and creates an authenticated session
7. User is logged into Gitea with their Okta identity
 
You can watch every step of this in your browser's network tab during login.
 
---
 
## Features
 
- Full OIDC authorization code flow between Okta and Gitea
- SP-initiated SSO (login starts from Gitea, redirects to Okta)
- Token exchange and claim mapping
- Self-hosted SP with no paid tier required
- Fully documented attribute mapping decisions
 
---
 
## Project Structure
 
```
okta-gitea-oidc-lab/
├── docker-compose.yml      # Gitea container configuration
├── .gitignore
└── README.md
```
 
---
 
## Prerequisites
 
- Docker Desktop installed (docker.com/products/docker-desktop)
- Free Okta developer tenant (developer.okta.com)
- Basic familiarity with browser dev tools
 
---
 
## Setup
 
### Part 1: Run Gitea Locally
 
**Step 1: Create docker-compose.yml**
 
```yaml
version: "3"
services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    ports:
      - "3000:3000"
      - "222:22"
    volumes:
      - ./gitea-data:/data
    restart: unless-stopped
```
 
**Step 2: Start Gitea**
```bash
docker-compose up -d
```
 
**Step 3: Complete initial setup**
- Open browser and go to http://localhost:3000
- Database: SQLite3
- Site URL: http://localhost:3000
- Create an admin account
- Click Install Gitea
 
---
 
### Part 2: Configure Okta
 
**Step 1: Create an OIDC app in Okta**
- Log into your Okta developer tenant
- Go to Applications > Applications
- Click Create App Integration
- Select OIDC > Web Application
- Name it "Gitea Lab"
 
**Step 2: Set redirect URIs**
- Sign-in redirect URI:
```
http://localhost:3000/user/oauth2/okta/callback
```
- Sign-out redirect URI:
```
http://localhost:3000
```
 
**Step 3: Note your credentials**
- Copy Client ID
- Copy Client Secret
- Copy your Okta domain (dev-xxxxxxxx.okta.com)
 
**Step 4: Assign a user to the app**
- Go to the app > Assignments tab
- Click Assign > Assign to People
- Assign your Okta developer account
 
---
 
### Part 3: Configure Gitea to Use Okta
 
**Step 1: Log into Gitea as admin**
 
**Step 2: Add Okta as authentication source**
- Go to Site Administration > Identity & Access > Authentication Sources
- Click Add Authentication Source
- Fill in:
  - Authentication Type: OAuth2
  - Name: okta
  - OAuth2 Provider: OpenID Connect
  - Client ID: paste from Okta
  - Client Secret: paste from Okta
  - OpenID Connect Auto Discovery URL:
```
https://dev-xxxxxxxx.okta.com/.well-known/openid-configuration
```
- Click Save
 
---
 
### Part 4: Test the Integration
 
- Log out of Gitea
- On the login page click **Sign in with okta**
- You will be redirected to Okta
- Log in with your Okta credentials
- You will be redirected back to Gitea as an authenticated user
 
Open browser dev tools > Network tab during login to watch the full OIDC flow in real time.
 
---
 
## Attribute Mapping Decisions
 
| Okta Claim | Gitea Field | Notes |
|------------|-------------|-------|
| sub | User ID | Unique identifier from Okta |
| email | Email | Used as primary identifier in Gitea |
| name | Display Name | Full name from Okta profile |
| preferred_username | Username | Maps to Gitea login name |
 
**Why these matter:** Attribute mapping is where most OIDC integrations break in production. The SP expects specific claim formats and the IdP may send them differently. Always validate the raw ID token when troubleshooting.
 
---
 
## Security Decisions
 
**Why OIDC over SAML?**
OIDC is JSON and REST-based, which makes it easier to inspect and debug than SAML's XML assertions. For modern SaaS applications OIDC is the preferred protocol. Both are covered in enterprise environments so understanding both matters.
 
**Why authorization code flow?**
The authorization code flow never exposes tokens in the browser URL. The code is short-lived and exchanged server-side for tokens. This is the correct flow for web applications and the one used in enterprise SSO implementations.
 
**Why assign users explicitly in Okta?**
Okta does not grant access to an app by default. Every user must be explicitly assigned. This mirrors the least privilege principle: access is denied unless explicitly granted.
 
---
 
## What I'd Add in Production
 
- HTTPS with a real TLS certificate (never run OIDC over HTTP in production)
- SCIM provisioning to automate user lifecycle between Okta and Gitea
- Group-based access policies so only specific Okta groups can authenticate
- Token lifetime configuration aligned to session security policy
- Centralized logging of authentication events to a SIEM
- MFA enforcement at the Okta policy level before allowing SP access
 
---
 
## Concepts Demonstrated
 
- OpenID Connect (OIDC) authorization code flow
- Identity provider and service provider federation
- SP-initiated SSO
- Token exchange and JWT claim mapping
- Attribute mapping between IdP and SP
- Least privilege access (explicit user assignment)
- Self-hosted service provider configuration
 
---
 
## Related Labs
 
- [JML Identity Lifecycle Automation](https://github.com/cibercazadora/jml-identity-automation) - Python + Microsoft Graph API
- Conditional Access Policy Lab (Entra ID + Intune) - coming soon
- Tailscale ZTNA Home Network Lab - coming soon
- Access Review Automation - coming soon
 
---
 
## Author
 
Priscilla Lopez
CISSP | GCPN | GCHFI
[linkedin.com/in/p-g-lopez](https://www.linkedin.com/in/p-g-lopez)
[cibercazadora.github.io](https://cibercazadora.github.io)
