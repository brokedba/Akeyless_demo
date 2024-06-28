# Akeyless_demo
In AkeyLess Each identity is represented by an Authentication Method object:

The Workflow relies on **3 main components** 
- **AuthN**  + **AuthZ** (RBAC) + **Token/Secret access** (Secret ITEM) 
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/adb547e9-ca0f-4a64-bd04-c28f0c48ca4c)
# Types of Identity 
- **Humans**
  - Anything manually entered/authenticated by a human ( i.e username |password)
- **Machine**
  - Automated access from a workload programatically (VM , a Dokcer container, A pod , cluster/service account, IAM Asumed role workload, OIDC Identity ) 
# Akeyless Authentication Methods

**1. For humans:**
- [LDAP](https://docs.akeyless.io/docs/ldap)
- [SAML](https://docs.akeyless.io/docs/saml)
- [OIDC](https://docs.akeyless.io/docs/openid)
- [OAuth2.0/JWT](https://docs.akeyless.io/docs/oauth20jwt)
- Email
- [API Keys](https://docs.akeyless.io/docs/api-key)

**2. For machine:**
- Cloud identities (IAM based) such as AWS IAM, Azure AD, GCP IAM, and OCI IAM.
- On-prem machines using [Universal Identity](https://docs.akeyless.io/docs/universal-identity)™.
- [Kubernetes](https://docs.akeyless.io/docs/kubernetes-auth). witohut need for password or API key.
- [Certificate](https://docs.akeyless.io/docs/certificate-based-authentication) based Authentication.
- [OAuth2.0/JWT](https://docs.akeyless.io/docs/oauth20jwt)
- [API Keys](https://docs.akeyless.io/docs/api-key)
- OIDC ? (gihub actions Identity)

# RBAC
<img src="https://files.readme.io/54c7a41-RBAC.JPG" width="600" height="500" />
- Allows to grant permissions on Secrets & Encryption Keys, Targets, Authentication methods and Access Roles, as long as audit logs, analytics access, Gateways settings and Secure Remote Access (SRA) information.

# Types of Secrets

- **Static Secrets**: long lived credentials like Key/value pairs that are manually created and updated. Examples passwords, API Keys (PII,credit card numbers..).

- **Dynamic Secrets**: short lived credentials generated on-demand for limited-time access with restricted permissions like "Just-In-Time" access.  

- **Rotated Secrets**: Periodically updated passwords for privileged-user accounts, stored securely for retrieval when needed.  
>   <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/3616e5a4-caee-481a-9678-83d6aa7032b9" width="600" height="600" />

**Other**:
- **Encryption Keys**: AES, RSA, or EC keys used for encrypting data and signing binaries or application transactions. See Encryption Keys
- **Certificates**: Akeyless acts as a Certificate Authority, supporting PKI/TLS Certificates and SSH certificates for internal environments.
   for example PKI or ssh certificates issuers alow to connect to remote servers using a CA signed ephemeral ssh cert eleiminating the need for public keys in the servers.
- **Targets**: Connectors between credentials and the items that need them, ensuring seamless integration and protection from credential breakage.
- **Oidc app**: alows to create the establish trust between the app and Akeyless for future access token exchange 
- **tokenizer**(anonymization):  allows data sanitization, such as social security, PII (credit card numbers etc) while preserving data format and uniqueness, and allow for data decryption later on.
      
- **Universal Secrets Connectors**:
 Previously called ESM - USC Allows to manage the life cycle of secrets within external secret/vaul platforms Such as Kubernetes secrets, Azure KV, AWS-SM, GCP etc.
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/4f5760e3-2db3-49a1-bcad-4b09cb3c228d" width="700" height="400" />
