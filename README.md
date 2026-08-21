# Secure Azure 3-Tier Web Application Architecture

## 🎯 Overview
This project showcases a hardened, enterprise-grade cloud architecture for a web application deployed on Microsoft Azure. The infrastructure is engineered using a "Defense-in-Depth" strategy to isolate sensitive application data, mitigate public-facing web threats, and manage secrets securely without code exposure.

## 🏗️ Architecture Design
![Azure Secure Architecture](azure.png)


## 🕵️‍♂️ Threat Modeling & Attack Scenario Simulation

To validate the security posture of this system architecture, the infrastructure was analyzed against realistic adversarial attack vectors. Below is the behavioral logic of how the system computationally mitigates active security breaches.

### Scenario A: Automated SQL Injection (SQLi) & Cross-Site Scripting (XSS)
* **The Threat Vector:** An external attacker deploys automated scanning tools (e.g., sqlmap) to inject malicious strings into application web forms, attempting to bypass authentication or dump database schemas.
* **System Deflection Logic:** The attack fails at the perimeter. Public HTTP traffic is inspected statefully by the **Azure WAF** before hitting computing servers. The WAF processes incoming headers against pre-configured OWASP Core Rule Sets (CRS). The malicious packets trigger deterministic anomaly thresholds, causing the system to automatically drop the TCP connection and return an HTTP `403 Forbidden` response code. The application logic layers are never touched.

### Scenario B: Exploitation of Compromised Server Logic (Lateral Movement)
* **The Threat Vector:** An attacker discovers a Zero-Day vulnerability in the web application's dependencies, achieving Remote Code Execution (RCE) on the underlying **Azure App Service** node. The attacker attempts to pivot and access the database.
* **System Deflection Logic:** The breach is isolated within a strict execution box. 
  1. **Network Disruption:** The attacker tries to run a port scan against the backend database. However, because the database is hosted inside an isolated **Private Data Subnet (Azure VNet)** with direct public routing disabled, the packets are discarded by default firewall Access Control Lists (ACLs).
  2. **Credential Obstruction:** The attacker searches server file directories for hardcoded database connection passwords. The search yields nothing because the system utilizes **Azure Entra Managed Identities**. The app requests temporary, short-lived tokens from **Azure Key Vault** via scoped API calls. The attacker cannot extract a static credential string to authenticate laterally into the data tier.

### Scenario C: Layer 7 Distributed Denial of Service (DDoS)
* **The Threat Vector:** A botnet coordinates thousands of concurrent dummy requests targeting application routes, attempting to exhaust the Web App Service’s CPU thread pool and crash the application.
* **System Deflection Logic:** The **Azure WAF** coupled with integrated rate-limiting algorithms drops traffic originating from malicious IP blocks at the edge. The system drops unauthenticated connection requests before they consume computational thread resources on the internal Web Server, preserving backend node availability.

## 🛡️ Computer Science & Cybersecurity Logic Implemented

### 1. Edge Security & Traffic Mitigation (Azure WAF)
* **CS Principle:** Request filtering and input validation.
* **Cyber Security Implementation:** All public HTTP/HTTPS traffic is forced through an Azure Web Application Firewall (WAF) before hitting backend computing nodes. This mitigates OWASP Top 10 vulnerabilities like SQL Injection (SQLi) and Cross-Site Scripting (XSS) at the network perimeter.

### 2. Network Segmentation & Zero-Trust (Azure VNet)
* **CS Principle:** Data isolation and decoupling system modules.
* **Cyber Security Implementation:** The system enforces strict network segmentation. The frontend Azure App Service lives in a public-facing VNet segment, while the Azure SQL Database is restricted within an isolated **Private Data Subnet**. The database has absolutely no public routing, denying direct internet access.

### 3. Least Privilege & Secrets Management (Azure Key Vault)
* **CS Principle:** Secure authentication and credential encapsulation.
* **Cyber Security Implementation:** Hardcoded passwords or connection strings are banned in the codebase. Database access tokens are dynamically pulled using Managed Identities from **Azure Key Vault**. This ensures that even if application servers are compromised, attacker lateral movement is blocked at the data tier.

## 🚀 Key Takeaways
This architecture achieves a robust security posture by aligning with the **NIST Cybersecurity Framework** and Microsoft Azure Security Benchmarks. It bridges core Computer Science platform scalability with professional, production-level system hardening.

