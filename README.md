# NexERP — by Nexora

> **Integrated Enterprise Resource Planning for Modern Indonesian Businesses**
> *Powered by ERPNext v16 | Part of CBQA Global Group | Innovating With Purpose*

[![ERPNext](https://img.shields.io/badge/ERPNext-v16.14-blue)](https://erpnext.com)
[![Frappe HRMS](https://img.shields.io/badge/HRMS-v16.5-orange)](https://github.com/frappe/hrms)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue)](https://hub.docker.com/r/frappe/erpnext)
[![License](https://img.shields.io/badge/License-GPL--3.0-lightgrey)](LICENSE)

---

## Overview

**NexERP** is Nexora's enterprise ERP platform built on [ERPNext](https://erpnext.com), customized for Indonesian businesses with full HR & Payroll localization (BPJS, PPh 21).

**Company:** Nexora | Part of CBQA Global Group
**URL:** https://cbqaglobal.co.id

---

## Key Features

### Core ERP Modules
- **Accounting & Finance** — GL, AR/AP, Bank Reconciliation, PPN 11%, PPh
- **HR & Payroll** — Employee Management, BPJS, PPh 21, Leave, Attendance, Recruitment
- **Procurement** — Purchase Orders, Supplier Management, 3-way matching
- **Sales & CRM** — Lead → Quotation → Sales Order → Invoice
- **Inventory & Stock** — Multi-warehouse, Batch/Serial, FIFO/LIFO
- **Manufacturing** — BOM, Work Orders, Production Planning
- **Projects** — Task Management, Timesheet, Resource Allocation
- **Assets** — Asset Register, Depreciation, Maintenance

### Indonesian Localizations
- **BPJS Ketenagakerjaan**: JKK, JKM, JHT (2% karyawan / 3.7% perusahaan), JP (1% / 2%)
- **BPJS Kesehatan**: 1% karyawan / 4% perusahaan
- **PPh 21**: Auto-calculation pada komponen gaji
- **Kalender Libur**: Hari Libur Nasional Indonesia 2026
- **Mata Uang**: IDR default

---

## Quick Start

### Prerequisites
- Docker 20+ and Docker Compose v2+
- Minimum: 4GB RAM, 20GB disk

### Run with Docker

```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
docker compose -f pwd.yml up -d
# Open: http://localhost:8080
# Login: Administrator / admin
```

### Custom NexERP Image (with HR module)

```dockerfile
FROM frappe/erpnext:v16.14.0
USER frappe
RUN cd /home/frappe/frappe-bench && \
    bench get-app --branch version-16 --skip-assets hrms
```

```bash
docker build -t nexora-erp:latest .
docker compose -f pwd-nexora.yml up -d

# Install HR on site
docker exec <backend-container> bench --site frontend install-app hrms
docker exec <backend-container> bench build --app hrms
```

---

## HR Payroll Structure (Staff - Nexora)

| Component | Type | Nilai |
|-----------|------|-------|
| Gaji Pokok | Earning | `base` |
| Tunjangan Jabatan | Earning | `base * 0.2` |
| Tunjangan Transport | Earning | Rp 500.000 |
| Tunjangan Makan | Earning | Rp 400.000 |
| BPJS JHT | Deduction | `base * 0.02` |
| BPJS JP | Deduction | `base * 0.01` |
| BPJS Kesehatan | Deduction | `base * 0.01` |
| PPh 21 | Deduction | (calculated) |

---

## Architecture

```
nginx :8080 → gunicorn (backend) → MariaDB 11.8
                                 → Redis (cache + queue)
websocket (Socket.IO) + scheduler + queue workers
```

---

## Useful Commands

```bash
# Bench CLI
docker exec -it <backend> bash
source /home/frappe/frappe-bench/env/bin/activate

bench --site frontend list-apps
bench --site frontend clear-cache
bench --site frontend migrate
bench --site frontend backup
```

---

## License

GNU General Public License v3.0 — same as upstream ERPNext.

---

*NexERP v1.0 | April 2026 | Nexora | Part of CBQA Global Group*
