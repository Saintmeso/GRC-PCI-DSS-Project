# PCI DSS Scope Determination Document

## Document Information
| Field | Value |
|-------|-------|
| Organization | MidTech Retail Solutions |
| Assessment Date | [5/26/2026] |
| Assessor | [Yousif Al Obaidi] |
| Version | 1.0 |

---

## 1. Executive Summary

### Scope Statement
The assessment evaluated the Cardholder Data Environment supporting MidTech Retail Solutions' e-commerce payment processing infrastructure as part of PCI DSS v4.0 scope determination activities. The environment includes web servers within the DMZ,payment processing systems, tokenization and encryption services, cardholder data storage systems as well as logging, patch management, authentication, backup, and administrative services. Although network segmentation does exist between the Corporate zone and the CDE, multiple shared services such as the AD, WSUS, Backup Database, and Centrailzed Logging/SIEM had mutliple shared services, trust relationships were also identified that may weaken segmentation effectiveness and expand their PCI DSS scope.

### Key Findings
- [ ] Shared AD, WSUS, Logging, and Backup infrastructure that weaken segmentation effectiveness between Corporate Zone and the CDE.
- [ ] Administrative trust relationships were identified via centralized authentication and jump host access creating indirect pathways into multiple systems handling CHD.
- [ ] Centralized logging and backup systems which expand PCI DSS scope due to potential exposure to payment related data.

### Overall Risk Rating
[ ] Low | [ ] Medium | [ ] High | [ ] Critical

---

## 2. Cardholder Data Environment Definition

### 2.1 Systems That STORE Cardholder Data

| System Name | Data Types Stored | Retention Period | Justification |
|-------------|------------------|------------------|---------------|
|Cardholder Database Server |PAN, transaction/payment related data |Unknown |Primary location for cardholder data used during payment processing. |
|Backup System |Bakcup copies of payment releated databases and stored CHD |Unknown |May store backup copies of systems containing CHD for recovery and retention purposes |

### 2.2 Systems That PROCESS Cardholder Data

| System Name | Processing Function | Data Elements | Justification |
|-------------|---------------------|---------------|---------------|
|Payment Application Server |Processes payment transaction |PAN, payment transation data |Receives and processes customer payment information during transaction handling. |
|Tokenization Service |Replaces PAN with tokens |PAN/tokenized payment data |Processes CHD to generate tokens that reduce exposure of cardholder data. |
|HSM Appliance | Performs encryption/cryptographic operations | Encrypted CHD,Cryptographic keys | Provides encryption and cryptographic services used to protect CHD during processing and storage |

### 2.3 Systems That TRANSMIT Cardholder Data

| System Name | Transmission Path | Encryption | Justification |
|-------------|-------------------|------------|---------------|
|Payment App Server | Payment systems <-> Tokenization/HSM/Database | Internal encryption machanisms likely used | Transmits CHD between payment-processing components, tokenization services, and storage systems. |
|Web Server |Customer -> Web Server -> Payment Application Server |Likely TLS/HTTPS |Recieves customer payment information and transmits CHD into the CDE for transaction processing. |
|Load Balancer |Web traffic distribution to payment systems |Likely TLS/HTTPS |Distributes incoming payment-related traffic to systems involved in processing CHD. |

---

## 3. Connected Systems Assessment

### 3.1 Direct Connections to CDE

| System | Connection Type | Business Purpose | In Scope? | Justification |
|--------|-----------------|------------------|-----------|---------------|
|Jump Host |Administrative Access |Allows admins to manage and maintain CDE systems | Yes |The jump host allows for direct admin access pathway to systems within the CDE. |
|WSUS |Patch/Update Management |Allows mass distribution of updates and security patches through out the CDE systems. | Yes |WSUS communicates directly with the CDE systems to release updates/patches and manage software throughout all of the CDE causing it to have a direct connection to the CDE systems.|

### 3.2 Indirect Connections (Via Intermediary)

| System | Connection Path | Business Purpose | In Scope? | Justification |
|--------|-----------------|------------------|-----------|---------------|
|Active Directory |Corporate AD -> Auth -> CDE Systens |Used for authentication and access management | Yes|Access to CDE systems may require AD authentication and or privileged account management which gives it an indirect connection to the CDE systems. |
|SIEM (Splunk) |Payment Systems -> Log Collector -> SIEM | Used for Centralized monitoring and security analysis | Yes |The SIEM receives logs generated from the Web server which handles CHD |

---

## 4. Security-Impacting Systems

### 4.1 Systems That Could Impact CDE Security

| System | Security Function | Impact if Compromised | In Scope? | Justification |
|--------|-------------------|----------------------|-----------|---------------|
|Active Directory |Main control on Authentication and priivilged access|The Impact of a compromised AD could grant an attacker the ability to impersonate users/admins, control devices, an or move laterally and these type of breaches are hard to detect as the seem very legitimate which in turn allows an attack to stay in the system for months | Yes|Since CDE systems may rely on AD authentication and account management, A compromise would deem extremely distructive as an attacker would effectivly have access to most of the CDE. |
|WSUS(Patch Management) |Centralized Patch/update deplotment |If compromised, attackers have the ability to mass distribute malicious updates and or software to the CDE systems which can lead to exposure of CHD | Yes|WSUS is used to directly manage and distribute updates to systems handling CHD, and it is solely because it can mass distribute updates/patches which makes it extremely damaging if compromised |

---

## 5. Trust Relationship Analysis

### 5.1 Authentication Dependencies

| CDE System | Authentication Source | Risk if Compromised | Mitigation |
|------------|----------------------|---------------------|------------|
|Payment Application Server |Active Directory |Attackers may gain control all connected accounts,devices and application which may lead to further breach into the CDE systems to obtain CHD  |Restrict privileged access via least privilege, The fewer privileged accounts, the smaller the attack surface is. Enforcing MFA which protects against stolen passwords and removes the risk of immediate access to administrative accounts |
|Card Database Server |Jump Host |Attackers may use the trust administrative pathways to gain unauthorized access to the Card Database Server |Restricting jump host access, implementing MFA and monitoring privileged sessions to insure security as well as isolating administrative access paths to lessen the attack surface. |

### 5.2 Shared Services

| Service | Used By CDE | Used By Non-CDE | Segmentation Effective? |
|---------|-------------|-----------------|------------------------|
| DNS |Yes |Yes | Yes |
| NTP |Yes |Yes | Yes |
| AD |Yes |Yes | No |
| Backup |Yes|Yes | No |
| Logging |Yes |Yes | No |
| Patching |Yes |Yes | No |

---

## 6. Segmentation Assessment

### 6.1 Segmentation Controls

| Control | Description | Effectiveness | Evidence |
|---------|-------------|---------------|----------|
| Firewall rules |The firewall seperates the Coroorate zone from the CDE as well as restrict network traffic between both environments. | Weak |Shared services like AD, WSUS, logging and Jumphost weaken the segmentation as those all have trust relationships that deem the firewall weak |
| VLAN separation |VThe vlan are used to seperate the CDE from the corporate zone | Weak |Services including AD authentication, WSUS, logging infrastrcture, backups, and jump host administrative access create trusted communication paths across segmented VLANS which deems the seperation weak.|
| Access controls |Administrative access to CDE systems is managed through AD and the jump host | Weak |Shared AD authentication and jump host administration create trusted access pathways between the corporate zone and systems that live within the CDE. |
| Monitoring |Centralized logging and SIEM monitoring are used for security visibilty across systems | Weak |Shared logging receievs logs from both CDE and non-CDE systems which reduces segmentation isolation by containing logs from both environments. |

### 6.2 Segmentation Gaps

| Gap ID | Description | Risk | Remediation |
|--------|-------------|------|-------------|
| GAP-001 |Shared AD between Corporate zone and CDE, meaning there is only one authentication system that controls both environments. | High |Implement seperate AD infrastrucure or specific authenication controls for CDE systems and restrict priviliged access. |
| GAP-002 |Shared WSUS (Patch Management), The same WSUS patches both Corporate and CDE systems, meaning one trusted management systems controls both environments. | High | Create a seperate WSUS for CDE and a seperate WSUS for the Corporate zone to correctly segment each other.|
| GAP-003 |Shared Logging/SIEM which logs from both Corporate systems and CDE systems. | High |Isolating CDE logging infrastructure from Corporate logging infracstrucure, restricting log access to only administrative/approved accounts, and ensuring CHD is not included in logs with the use of log filitering |

---

## 7. QSA Challenge Preparation

### 7.1 Anticipated Questions

| Area | Likely Question | Your Response | Evidence |
|------|-----------------|---------------|----------|
| Shared Services |How do shared services between the Corporate Zone and CDE affect segmentation effectivenes? |Although network segmentation exists, multiple services such as AD, WSUS, logging, and backups all create trust relationships that weaken and completely bypass current isolation measures between environments. |Mulitple shared services are used by both the Corporate Zone and the CDE, weakening segmentation effectiveness. Active Directory is used for authentication across both environements, creating a shared trust relationship between Corporate and CDE systems. Similarly, WSUS centrally manages and distributes updates to systems across both environments, creating a shared management dependency. In addition, centralized logging infrastructure collects logs from both Corporate and CDE systems into a shared SIEM environment, reducing operational isolation between segmented networks. |
| Admin Access |How is administrative access into CDE managed and restricted? |Administrative access is managed through a Jump host and shared authentication infrastructure, which may weaken segmentation effectiveness if compromised | The Jump host provides centralized administrative access used by authorized administrators to perform maintenance, patching, and management of systems within the CDE, including the Payment Application Server and Card Database Server. This creates a trusted administrative pathway between shared infrastructure and CDE systems.  |
| Logging |Do centralized logs contain CHD or payment-releated information? |Logs created from payment processing systems may contain payment relataed information depending on application logging configuration |Centralized logging infrastructure collects logs from both corporate and CDE systems into a shared SIEM environment. Pyament processing systems, including web and application servers all forward logs to the same monitoring infrastructure, reducing the effectiveness of isolation between segmented environments. |
| Backups |Do backup systems contain CHD or copies of CDE systems? |Backup infracstructure may contain copies of systems storing CHD, including payment databases and releated system data. |Current backup database is used to store copies of systems within the CDE which includes payment related databases and system data. Shared backup services between corporate and CDE environments reduce segmentation isolation and may possbily extend PCI scope to backup infrastructure. |

---

## 8. Recommendations

### 8.1 Immediate Actions (0-30 days)
1. [Restrict and review privliged administrative access to CDE systems such as the Jump host and Shared AD accounts to reduce immediate admin risk. ]
2. [Review logging configurations to ensure CHD and authentication data is not visible or accessable in logs to reduce hidden PCI scope exposure. ]
3. [Implement MFA for all administrative access into systems within the CDE to protect against credential compromise.  ]

### 8.2 Short-Term Actions (30-90 days)
1. [Seperating WSUS for CDE systems and corporate systems to remove segmentation weakness. ]
2. [Isolate logging and monitoring for systhems that habdle CHD in order to reduce visiblity crossover and support operational separation ]
3. [Harden the firewall and VLAN segmentation rules between the corporate zone and the CDE based on the trust relationships between the AD, WSUS, Jump Host, SIEM and Backup Database. ]

### 8.3 Long-Term Actions (90+ days)
1. [Implement a dedicated AD infrastructure for CDE systems to reduce against shared authentication trust relationships ]
2. [Deploy a dedicated backup database for CDE systems and ensure backup data is encrypted and access is restricted via Least privilege]
3. [Conduct formal PCI DSS segmentation testing and scope validatation to ensure continued segmentation effectiveness ]

---

## 9. Residual Risk Statement

After implementing recommendations, the following risks remain:

| Risk | Likelihood | Impact | Acceptance Required? |
|------|------------|--------|---------------------|
|Shared Auth and Admin trust relationship may still create indirect pathways into the CDE despite enforced segmentation controls |Medium |High | Yes |
|Centralized operational services such as logging,backups, and montioring may continue to expose payment information across shared infrastructure |Low |Medium | Yes |

---

## Appendices

### A. Network Diagram
<img width="762" height="797" alt="image" src="https://github.com/user-attachments/assets/1947fb8e-fa10-4645-8902-b396db68653e" />
<img width="768" height="782" alt="image" src="https://github.com/user-attachments/assets/03101849-f4db-4cad-89d2-d5172a9596b8" />



### B. Data Flow Diagram
<img width="522" height="661" alt="PCI-DSS drawio" src="https://github.com/user-attachments/assets/76cc87d9-ecad-47cd-ad83-8fc49740ccbb" />


### C. Evidence Index
| Evidence ID | Description | Location |
|-------------|-------------|----------|
|EVID-001 |Network architecture and CHD flow analysis identifying shared services, trust relationships and segmentation weaknesses between the Corporate Zone and the CDE |Appenix B - Data Flow Diagram Figure 1 |

---

## Approval

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Assessor |Yousif Al Obaidi |5/26/2026 |YA |
| Reviewer | | | |
| Approver | | | |
