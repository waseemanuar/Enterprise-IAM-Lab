# Enterprise-IAM-Lab
A hands-on Identity & Access Management (IAM) lab demonstrating Federated Authentication between a Linux-based Keycloak IdP and a Windows Server Active Directory domain.

### Project Overview
This project simulates a real-world enterprise environment where Single Sign-On (SSO) is required across a hybrid infrastructure. By bridging a Windows Domain Controller with a Keycloak Identity Provider, this lab achieves centralized user management and secure authentication for modern web applications.

### Core Objectives
* **Centralized Identity:** Use Active Directory as the "Source of Truth" for all user identities.
* **Identity Federation:** Implement LDAP protocol to sync users into Keycloak.
* **Security Hardening:** Configure service accounts with the Principle of Least Privilege and manage firewall rules for cross-VM communication.
* **Network Orchestration:** Design a private internal network with static IP addressing for reliable service-to-service discovery.

###  Architecture Overview
This lab simulates a modern enterprise environment where a hybrid infrastructure requires a centralized Single Sign-On (SSO) solution.

* **Primary Domain Controller (PDC):** Acts as the "Source of Truth" for user identities.
* **Identity Provider (IdP):** Acts as the gateway for modern web applications using OpenID Connect (OIDC) or SAML.

| Host | Role | OS | IP Address |
| :--- | :--- | :--- | :--- |
| **DC01** | LDAP / Active Directory | Windows Server 2022 | `192.168.1.10` |
| **IAM01** | Keycloak / Docker | Ubuntu Server 22.04 | `192.168.1.11` |

###  Technical Stack
* **Identity:** Microsoft Active Directory
* **SSO Engine:** Keycloak 24.0 (Docker Container)
* **Networking:** Static Netplan Configuration (Linux)
* **Protocols:** LDAP, TCP/IP, OIDC

###  Key Configurations

#### 1. Networking (Netplan)
The Ubuntu VM was configured with a persistent static IP to ensure reliable service-to-service discovery within the internal virtual network.

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses: [192.168.1.11/24]
      nameservers:
        addresses: [192.168.1.10]
```
#### 2. LDAP User Federation
Established a secure, persistent handshake between Keycloak and the Windows Domain Controller. Integration was achieved using a dedicated service account, adhering to the **Principle of Least Privilege (PoLP)**.

* **Authentication Bind:** Utilized User Principal Name (UPN) format (`keycloak-svc@corp.net`) for a robust connection.
* **Attribute Mapping:** Synchronized the Windows `sAMAccountName` attribute to the Keycloak `username` field to ensure cross-platform identifier consistency.
* **Search Optimization:** Defined a specific search scope within targeted Organizational Units (OUs) to improve performance and minimize the exposed attack surface.

###  Verification & Results
* **Identity Synchronization:** Successfully imported Active Directory users (e.g., `jdoe`) into the specialized `Lab-Enterprise` realm.
* **End-to-End Authentication:** Validated the full login lifecycle. Keycloak successfully delegated credential verification to the Windows DC, confirming a functional federated identity bridge.

###  Lessons Learned & Troubleshooting
* **AD Account Policies:** Identified that the "User must change password at next logon" flag interrupts automated LDAP binds, requiring specific account provisioning steps for service users.
* **YAML Syntax Precision:** Resolved boot-time network failures by auditing Netplan indentation, reinforcing the importance of strict syntax in "Infrastructure as Code" configurations.
