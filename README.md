# Hybrid Security Architecture for Odoo ERP: Hardening and Threat Monitoring

[![Odoo](https://img.shields.io/badge/Odoo-17-714B67?style=flat&logo=odoo&logoColor=white)](https://www.odoo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Nginx](https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?style=flat&logo=nginx&logoColor=white)](https://nginx.org/)
[![ModSecurity](https://img.shields.io/badge/ModSecurity-OWASP%20CRS-red?style=flat)](https://modsecurity.org/)
[![Wazuh](https://img.shields.io/badge/Wazuh-4.14.5-00A3E0?style=flat)](https://wazuh.com/)
[![License](https://img.shields.io/badge/License-Academic-blue.svg)](#)

**Final Year Project**  
Department of Cybersecurity — Mehran University of Engineering & Technology (MUET)

**Author:** [Usman-543](https://github.com/Usman-543)  
**Host OS:** Pop!_OS 22.04 LTS

---

## Abstract

This project designs and implements a **hybrid security architecture** for Odoo ERP that combines:

- Containerized application stack (Odoo 17 + PostgreSQL 15)
- Nginx reverse proxy with TLS 1.3
- ModSecurity Web Application Firewall (OWASP CRS v3.3.5)
- Wazuh SIEM for real-time threat monitoring and Active Response

The system was validated through controlled attack simulations (SQL Injection, XSS, Path Traversal, and Brute Force) from a Kali Linux VM. All attacks were successfully detected and blocked, with automated firewall responses triggered via custom Wazuh rules.

---

## Architecture Overview

```
┌─────────────────┐     TLS 1.3      ┌──────────────────────┐
│   Client /      │ ───────────────► │  Nginx + ModSecurity │
│   Attacker      │                  │  (WAF + Reverse Proxy)│
└─────────────────┘                  └────────────┬───────────┘
                                               │
                                               ▼
                                     ┌──────────────────────┐
                                     │   Odoo 17 Container │
                                     └───────────┬───────────┘
                                               │
                                               ▼
                                     ┌──────────────────────┐
                                     │ PostgreSQL 15       │
                                     └──────────────────────┘
                                               │
                     ┌────────────────────────┬───────────────────────┐
                     ▼                         ▼                         ▼
              Wazuh Agent               Wazuh Manager              Active Response
              (Host)                    (192.168.56.103)           (firewall-drop)
```

---

## Key Components

| Component              | Version / Details                  | Role                                      |
|------------------------|------------------------------------|-------------------------------------------|
| Odoo                   | 17 (Docker)                        | ERP Application                           |
| PostgreSQL             | 15 (Docker)                        | Database                                  |
| Nginx                  | Reverse Proxy + TLS 1.3            | Secure entry point                        |
| ModSecurity + CRS      | OWASP CRS v3.3.5                   | WAF (SQLi, XSS, Path Traversal, etc.)     |
| Wazuh SIEM             | 4.14.5 (Manager on VM)             | Log collection, detection, Active Response|
| Kali Linux             | Attack simulation VM               | Penetration testing                       |

---

## Features Implemented

- [x] Hardened Docker deployment of Odoo + PostgreSQL
- [x] Nginx reverse proxy with modern TLS configuration
- [x] ModSecurity WAF with OWASP Core Rule Set
- [x] Wazuh agent on host + Manager on dedicated VM
- [x] Custom Wazuh rules (Rule 100001) + Active Response (`firewall-drop`)
- [x] Controlled attack simulation & evidence collection
- [x] Sanitized configuration files (no secrets in repository)

---

## Repository Structure

```
├── docker/                 # Docker Compose (sanitized)
├── configs/
│   ├── nginx/              # Nginx configuration
│   ├── odoo/               # Odoo config example
│   ├── modsecurity/        # ModSecurity overrides
│   └── wazuh/              # Custom local_rules.xml
├── scripts/                # Automation & helper scripts
├── evidence/               # Screenshots of tests & dashboards
├── docs/                   # Additional documentation
├── .env.example            # Environment variable template
└── README.md
```

---

## Quick Start (Local Lab)

> **Note:** This is an academic lab environment. Do not expose it to the public internet without additional hardening.

1. Clone the repository
2. Copy `.env.example` → `configs/postgres.env` and set strong passwords
3. Create the external Docker network:
   ```bash
   docker network create odoo-network
   ```
4. Start the stack:
   ```bash
   cd docker
   docker compose up -d
   ```
5. Configure Nginx + ModSecurity and point to the Odoo container
6. Install/register Wazuh agent and import `configs/wazuh/local_rules.xml`

---

## Attack Simulation Results

| Attack Type              | Tool / Method          | WAF Result     | Wazuh Detection          | Active Response |
|--------------------------|------------------------|----------------|--------------------------|-----------------|
| SQL Injection            | Manual payload         | Blocked (403)  | Yes (ModSecurity rules)  | -               |
| Cross-Site Scripting (XSS)| Manual payload        | Blocked (403)  | Yes                      | -               |
| Path Traversal / LFI     | `../../../../etc/passwd` | Blocked (403)| Yes (CRS 930/932 rules) | -               |
| Brute Force Login        | Hydra (rockyou.txt)    | 403 responses  | Yes (Rule **100001**)    | firewall-drop   |

Evidence screenshots are available in the [`evidence/`](evidence/) folder.

---

## Security Considerations

- All secrets are excluded via `.gitignore`
- Real certificates, private keys, and production passwords are **not** present in this repository
- Active Response uses `firewall-drop` to temporarily block attacking IPs
- Services are bound to localhost where possible
- Self-signed TLS certificate used for lab environment (`erp.local`)

---

## Author & Academic Context

**Usman-543**  
Final Year Project — Department of Cybersecurity  
Mehran University of Engineering & Technology (MUET)

---

## License

This project is released for **academic and educational purposes** only.
