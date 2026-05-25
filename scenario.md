# PCI DSS v4.0 Scope Determination & Segmentation Assessment

## Scenario Overview
This project simulates a PCI DSS v4.0 scope determination and segmentation assessment for **MidTech Retail Solutions**, a fictional e-commerce company processing approximately 2.4 million credit card transactions annually. The organization implemented network segmentation between its Corporate Zone and Cardholder Data Environment (CDE) and believed the segmentation significantly reduced PCI scope.

The purpose of this assessment was to determine whether the segmentation was truly effective or whether shared services, trust relationships, and centralized infrastructure weakened isolation and expanded PCI DSS scope.

---

## Environment Overview
The simulated environment included:
- Internet-facing Web Servers within a DMZ
- Payment Application Server
- Cardholder Database Server
- Tokenization Service
- HSM (Hardware Security Module)
- Active Directory
- WSUS Patch Management
- SIEM / Centralized Logging
- Backup Infrastructure
- Administrative Jump Host

---

## Project Objectives
During this assessment, I:
- Identified systems that store, process, and transmit Cardholder Data (CHD)
- Mapped CHD flow throughout the environment using a Data Flow Diagram (DFD)
- Assessed connected and security-impacting systems
- Evaluated shared services and trust relationships crossing segmentation boundaries
- Performed PCI DSS v4.0 segmentation validation
- Identified segmentation gaps and PCI scope expansion risks
- Developed remediation recommendations and residual risk analysis
