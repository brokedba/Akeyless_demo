# Akeyless_demo
In AkeyLess Each identity is represented by an Authentication Method object:

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
- On-prem machines using [Universal Identity](https://docs.akeyless.io/docs/universal-identity)™. against secret 0 (no agent to reboot like hashi Vault)
- [Kubernetes](https://docs.akeyless.io/docs/kubernetes-auth). Without a need for password or API key.
- [Certificate](https://docs.akeyless.io/docs/certificate-based-authentication) based Authentication.
- [OAuth2.0/JWT](https://docs.akeyless.io/docs/oauth20jwt)
- [API Keys](https://docs.akeyless.io/docs/api-key)
- OIDC ? (gihub actions Identity)

> **Note:** see how clients are counted [here](https://www.akeyless.io/akeyless-clients/)
# Access Role RBAC
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/adb547e9-ca0f-4a64-bd04-c28f0c48ca4c)
The Workflow relies on **3 main components** 
- **AuthN** Method  + **AuthZ** (Acess Role) + **Token/Secret access** (Secret ITEM) 
<img src="https://files.readme.io/54c7a41-RBAC.JPG" width="400" height="300" />
Access Roles Allow to grant permissions on Secrets & Encryption Keys, Targets, Authentication methods and Access Roles, as long as audit logs, analytics access, Gateways settings and Secure Remote Access (SRA) information.

# Types of Secrets
![image](https://github.com/brokedba/Akeyless_demo/assets/29458929/ba1dc4a1-a53d-4547-a3e7-e45c82816452)


- **Static Secrets**: long lived credentials like Key/value pairs that are manually created and updated. Examples passwords, API Keys (PII,credit card numbers..).

- **Dynamic Secrets**: short lived credentials generated on-demand for limited-time access with restricted permissions like "Just-In-Time" access.  

- **Rotated Secrets**: Periodically updated passwords for privileged-user accounts, stored securely for retrieval when needed.  
>   <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/3616e5a4-caee-481a-9678-83d6aa7032b9" width="600" height="600" />

**Other**:
- **Encryption Keys**: AES, RSA, or EC keys used for encrypting data and signing binaries or application transactions. See Encryption Keys
- **Certificates**: Akeyless acts as a Certificate Authority, supporting PKI/TLS Certificates and SSH certificates for internal environments.
   for example PKI or ssh certificates issuers alow to connect to remote servers using a CA signed ephemeral ssh cert eliminating the need for public keys in the servers.
- **Targets**: Are Connectors that link stored credentials to the systems and applications that need them. It seamlessly connects Akeyless with external systems, databases, and applications. alowing for dynamic secret creations, with granular permissions.

- **OIDC app**: alows to create the establish trust between the app and Akeyless for future access token exchange 
- **Tokenizer**(anonymization):  allows data sanitization, such as social security, PII ([credit card numbers](https://support.bluesnap.com/docs/test-credit-card-numbers) etc) while preserving data format and uniqueness, and allow for data decryption later on. i.e "374245455400126"
      
- **Universal Secrets Connectors**:
 (Previously ESM) - USC Allows to manage the life cycle of secrets within external secret/vaul platforms Such as Kubernetes secrets, Azure KV, AWS-SM, GCP etc. It's also Bidirectional.
> <img src="https://github.com/brokedba/Akeyless_demo/assets/29458929/4f5760e3-2db3-49a1-bcad-4b09cb3c228d" width="700" height="400" />
