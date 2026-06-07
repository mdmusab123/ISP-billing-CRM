<p align="center">
  <img src="https://img.shields.io/badge/Laravel-12.x-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" />
  <img src="https://img.shields.io/badge/PHP-8.2+-777BB4?style=for-the-badge&logo=php&logoColor=white" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/MongoDB-NoSQL-47A248?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Redis-Alpine-DC382D?style=for-the-badge&logo=redis&logoColor=white" />
  <img src="https://img.shields.io/badge/Flutter-Dart-02569B?style=for-the-badge&logo=flutter&logoColor=white" />
</p>

<h1 align="center">🌐 Amexcess ISP Portal</h1>

<p align="center">
  <strong>Enterprise-grade ISP Billing, CRM & Network Management Platform</strong><br/>
  Cryptographic licensing · Multi-vendor SNMP telemetry · Zero-config deployment in under 2 minutes
</p>

<p align="center">
  <a href="#-quick-install">Quick Install</a> •
  <a href="#-architecture">Architecture</a> •
  <a href="#-features">Features</a> •
  <a href="#-cicd-pipeline">CI/CD</a> •
  <a href="#-technology-stack">Tech Stack</a> •
  <a href="#-security--licensing">Security</a>
</p>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
  - [System Architecture](#system-architecture)
  - [Request Lifecycle](#request-lifecycle)
  - [Database Strategy](#database-strategy---polyglot-persistence)
  - [Scheduled Task Engine](#scheduled-task-engine)
- [Features](#-features)
  - [Customer Management](#-customer-management)
  - [Billing & Invoicing](#-billing--invoicing)
  - [Payment Gateways](#-payment-gateways)
  - [MikroTik Integration](#-mikrotik-router-integration)
  - [RADIUS / NAS](#-radius--nas-management)
  - [OLT & Fiber Monitoring](#-olt--fiber-network-monitoring)
  - [Real-Time Monitoring](#-real-time-bandwidth-monitoring)
  - [Ticketing System](#-support-ticketing-system)
  - [Staff & HR](#-staff--hr-management)
  - [Inventory](#-inventory-management)
  - [Network Visualization](#-network-visualization)
  - [Marketing & Messaging](#-marketing--messaging)
  - [Reports & Analytics](#-reports--analytics)
- [Security & Licensing](#-security--licensing)
  - [Triple Trap Mechanism](#the-triple-trap--rs256-jwt-payload)
- [Quick Install](#-quick-install)
- [Deployment](#-deployment)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Technology Stack](#-technology-stack)
- [Project Structure](#-project-structure)
- [Hardware Integrations](#-hardware-integrations)
- [Mobile Apps](#-mobile-companion-apps)
- [Configuration](#️-configuration)

---

## 🧭 Overview

Amexcess ISP Portal is a **production-grade, full-stack ISP management ecosystem** built for Internet Service Providers operating MikroTik routers and fiber OLT equipment. The platform handles the entire ISP operations lifecycle:

```
Customer Signup → Package Assignment → Router Provisioning → Billing → Payment → Monitoring → Support
```

**What makes it different:**

| Capability | Description |
|---|---|
| **Zero-Config Deployment** | Single `curl` command installs the entire stack in under 2 minutes |
| **Cryptographic Licensing** | App physically cannot function without valid license — no simple bypass |
| **Multi-Vendor Hardware** | Supports BDCOM, V.SOL, Raisecom, DN-OPTIC OLTs via SNMP |
| **Polyglot Database** | MySQL for billing (ACID), MongoDB for high-frequency telemetry |
| **Real-Time Dashboards** | WebSocket-powered live bandwidth, ONU status, and system metrics |
| **Mobile Apps** | Flutter-based companion apps for Admin and Customer portals |

---

## 🏗 Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          CLIENT VPS (Production)                        │
│                                                                         │
│  ┌─────────────────┐  ┌──────────────┐  ┌─────────┐  ┌──────────────┐ │
│  │   FrankenPHP     │  │   MySQL 8.0  │  │  Redis  │  │   MongoDB    │ │
│  │   + Caddy SSL    │  │              │  │  Alpine │  │   (NoSQL)    │ │
│  │   (Laravel 12)   │  │  Billing     │  │         │  │              │ │
│  │                   │  │  Customers   │  │ Queue   │  │  ONU Data    │ │
│  │  Worker Mode      │  │  Invoices    │  │ Cache   │  │  Bandwidth   │ │
│  │  (3-4x faster)    │  │  Staff       │  │ Session │  │  OLT Metrics │ │
│  └───────┬───────────┘  └──────────────┘  └─────────┘  └──────────────┘ │
│          │                     Docker Compose Network                    │
└──────────┼──────────────────────────────────────────────────────────────┘
           │
           │  ┌──────────────────────────────────────────────────────┐
           │  │              NETWORK INTEGRATIONS                     │
           │  │                                                       │
           ├──┤  MikroTik RouterOS API ──► PPPoE / Hotspot / VPN     │
           │  │  FreeRADIUS (MySQL)    ──► Centralized Auth + Pools  │
           │  │  SNMP v1/v2c           ──► OLT Health + ONU Polling  │
           │  └──────────────────────────────────────────────────────┘
           │
           │  RS256 JWT (every 24h)
           ▼
┌──────────────────────┐       ┌──────────────────────────────────────┐
│  Amexcess Central    │       │  CLIENT MOBILE APPS (Flutter/Dart)   │
│  Licensing Server    │       │                                      │
│  (amexcess.me)       │       │  📱 Admin Panel App (Android/iOS)    │
│                      │       │  📱 Customer Panel App (Android/iOS) │
│  ┌────────────────┐  │       └──────────────────────────────────────┘
│  │ License Keys   │  │
│  │ JWT Signing    │  │       ┌──────────────────────────────────────┐
│  │ DNS Provision  │  │       │  CI/CD (GitHub Actions)              │
│  │ Cloudflare API │  │       │                                      │
│  └────────────────┘  │       │  Lint → Test → Build → Docker Push   │
└──────────────────────┘       └──────────────────────────────────────┘
```

### Request Lifecycle

```
                    ┌─────────────────────────────────────────┐
                    │           INCOMING HTTP REQUEST          │
                    └────────────────┬────────────────────────┘
                                     │
                                     ▼
                    ┌─────────────────────────────────────────┐
                    │   Caddy (Reverse Proxy + Auto-SSL)      │
                    │   TLS 1.3 · Let's Encrypt / ZeroSSL     │
                    └────────────────┬────────────────────────┘
                                     │
                                     ▼
                    ┌─────────────────────────────────────────┐
                    │   FrankenPHP (Worker Mode)               │
                    │   Laravel boots ONCE, stays in memory    │
                    │   3-4x throughput vs PHP-FPM             │
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
                    ▼                ▼                ▼
            ┌──────────┐    ┌──────────┐    ┌──────────────┐
            │   Blade  │    │ Inertia  │    │   REST API   │
            │  (Admin) │    │ (React)  │    │  (Mobile +   │
            │          │    │          │    │   Webhooks)  │
            └──────────┘    └──────────┘    └──────────────┘
                    │                │                │
                    └────────────────┼────────────────┘
                                     │
                    ┌────────────────┼────────────────────┐
                    │                │                    │
                    ▼                ▼                    ▼
            ┌──────────┐    ┌──────────────┐    ┌──────────────┐
            │  MySQL   │    │   MongoDB    │    │    Redis     │
            │ (ACID)   │    │  (Telemetry) │    │   (Queue +   │
            │          │    │              │    │    Cache)    │
            └──────────┘    └──────────────┘    └──────────────┘
```

### Database Strategy — Polyglot Persistence

The system uses **two database engines** simultaneously, each optimized for its workload:

```
┌───────────────────────────────────────────────────────────────────┐
│                    POLYGLOT PERSISTENCE MODEL                     │
├───────────────────────────────┬───────────────────────────────────┤
│         MySQL 8.0             │           MongoDB                 │
│     (Relational / ACID)       │     (Document / Time-Series)      │
├───────────────────────────────┼───────────────────────────────────┤
│                               │                                   │
│  ✦ Customers                  │  ✦ ONU Status & Optical Signals   │
│  ✦ Invoices & Payments        │  ✦ ONU Signal History             │
│  ✦ Packages & Pricing         │  ✦ OLT Metrics (CPU/Temp/Mem)     │
│  ✦ Staff & Payroll            │  ✦ Bandwidth Usage Counters       │
│  ✦ Tickets & Replies          │  ✦ Upstream Traffic Samples        │
│  ✦ Zones & Sub-Zones          │  ✦ Upstream Interface Stats        │
│  ✦ MikroTik Router Records    │  ✦ Upstream Bandwidth Usage        │
│  ✦ RADIUS Auth Tables         │                                   │
│  ✦ Inventory & Products       │                                   │
│  ✦ Roles & Permissions        │                                   │
│                               │                                   │
│  WHY: Financial data needs    │  WHY: High-frequency writes from  │
│  strict ACID transactions,    │  SNMP polling (every 1-5 min),    │
│  foreign key constraints,     │  flexible schemas for varying     │
│  and relational JOINs.        │  OLT vendors, and fast document   │
│                               │  upserts without table locks.     │
└───────────────────────────────┴───────────────────────────────────┘
```

### Scheduled Task Engine

The platform runs **34 automated background commands** via Laravel's task scheduler and Redis-backed queues:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCHEDULED TASK ENGINE                         │
│                  (cron → artisan schedule:run)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ⏱ EVERY SECOND                                                 │
│  └── upstream:track              Track upstream interface Tx/Rx  │
│                                                                  │
│  ⏱ EVERY MINUTE                                                 │
│  ├── invoices:auto-due-expired   Auto-generate overdue invoices  │
│  ├── disconnect:due-customers    Suspend expired PPPoE/RADIUS    │
│  ├── reconnect:eligible          Re-enable paid customers        │
│  ├── bandwidth:track-all         Poll all active user bandwidth  │
│  ├── piprapay:verify-pending     Verify pending payments         │
│  └── queue:work                  Process background job queue    │
│                                                                  │
│  ⏱ EVERY 5 MINUTES                                              │
│  ├── mikrotik:sync-packages      Sync DB packages → Router       │
│  └── radius:sync-bandwidth       Sync RADIUS acct → MongoDB     │
│                                                                  │
│  ⏱ DAILY                                                        │
│  ├── mikrotik:cleanup-profiles   Remove orphaned router profiles │
│  └── backup:gdrive               Backup database to Google Drive │
│                                                                  │
│  ⏱ MONTHLY (1st of month)                                       │
│  └── billing:generate-monthly    Generate all customer invoices  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Features

### 👥 Customer Management

- Full lifecycle: registration, editing, identity verification, ban/unban
- Geographic organization via Zones and Sub-Zones
- Real-time online/offline status indicators
- Customer identity document upload and verification
- Connection type support: **PPPoE, PPTP, L2TP, SSTP, Hotspot, Static**
- Customer-facing self-service portal with billing and tickets
- Hotspot voucher generation and batch printing

### 💰 Billing & Invoicing

- **Automated monthly invoice generation** (1st of each month)
- Multi-month advance payment support
- Invoice verification workflow for manual payments
- Auto-disconnect on overdue → auto-reconnect on payment
- Due/Paid/Overdue/Expired status management
- Bulk invoice printing (standard + POS thermal)
- BTRC regulatory export reports

### 💳 Payment Gateways

| Gateway | Type | Integration |
|---|---|---|
| **bKash Tokenized** | Automated | Programmerhasan/bkash package |
| **PipraPay** | Automated | REST API with webhook verification |
| **Nagad** | Manual | Admin verification workflow |
| **Rocket** | Manual | Admin verification workflow |
| **Upay** | Manual | Admin verification workflow |
| **Bank Transfer** | Manual | Admin verification workflow |

### 📡 MikroTik Router Integration

```
┌──────────────────────────────────────────────────────────────┐
│                  MIKROTIK INTEGRATION LAYER                   │
│               (evilfreelancer/routeros-api-php)                │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────┐    ┌─────────────────┐                  │
│  │   PPP Users      │    │   Hotspot       │                  │
│  │   ─────────      │    │   ───────       │                  │
│  │   • PPPoE        │    │   • Setup Wizard│                  │
│  │   • PPTP         │    │   • User CRUD   │                  │
│  │   • L2TP         │    │   • Vouchers    │                  │
│  │   • SSTP         │    │                 │                  │
│  └─────────────────┘    └─────────────────┘                  │
│                                                               │
│  ┌─────────────────┐    ┌─────────────────┐                  │
│  │   IP Pools       │    │   DHCP Server   │                  │
│  │   ─────────      │    │   ───────────   │                  │
│  │   • Pool CRUD    │    │   • Server CRUD │                  │
│  │   • Sync to DB   │    │   • Fetch Info  │                  │
│  └─────────────────┘    └─────────────────┘                  │
│                                                               │
│  ┌─────────────────┐    ┌─────────────────┐                  │
│  │  Address Lists   │    │  Interface Mon. │                  │
│  │  ─────────────   │    │  ─────────────  │                  │
│  │  • Firewall ACL  │    │  • Live Rx/Tx   │                  │
│  │  • Block/Allow   │    │  • Upstream Mgr │                  │
│  └─────────────────┘    └─────────────────┘                  │
│                                                               │
│  🔐 Router passwords are AES-encrypted in MySQL.              │
│     Decryption key is injected via RS256 JWT at runtime.      │
│     Without a valid license → passwords cannot be decrypted.  │
└──────────────────────────────────────────────────────────────┘
```

### 🔑 RADIUS / NAS Management

- RADIUS server configuration and connection testing
- IP Pool synchronization between portal and RADIUS
- User authentication via `radcheck` / `radreply` tables
- Dynamic IP allocation with `Framed-Pool` attributes
- Rate limiting integration via `Mikrotik-Rate-Limit`
- Bandwidth accounting sync from RADIUS → MongoDB

### 🔬 OLT & Fiber Network Monitoring

```
┌──────────────────────────────────────────────────────────────────┐
│               OLT VENDOR ABSTRACTION LAYER                       │
│                  (Factory Pattern + SNMP)                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  OltServiceFactory ──┬──► VsolService     (.1.3.6.1.4.1.37950)   │
│                      ├──► BdcomService    (.1.3.6.1.4.1.3320)    │
│                      ├──► RaisecomService (.1.3.6.1.4.1.8886)    │
│                      └──► DnOpticService  (CDATA rebrand)        │
│                                                                   │
│  Each service provides:                                           │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  syncStatus()   → CPU, Temperature, Memory, Uptime         │ │
│  │  syncOnus()     → Discover & update all ONUs on OLT         │ │
│  │  rebootOnu()    → Remote ONU restart via SNMP SET           │ │
│  │  isOnline()     → Reachability check                        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ONU Data Points Collected:                                       │
│  • Rx Power (dBm)    • Tx Power (dBm)    • ONU Temperature       │
│  • Supply Voltage    • Bias Current      • Distance (meters)     │
│  • MAC Address       • Status            • Deregistration Reason │
│  • Model/Vendor      • PON Port Mapping  • Signal History        │
│                                                                   │
│  ⚠️ SNMP OIDs are injected via the licensing JWT.                │
│     Without a valid license → OID array is empty → sync fails.   │
└──────────────────────────────────────────────────────────────────┘
```

### 📊 Real-Time Bandwidth Monitoring

- **WebSocket-powered** live bandwidth charts (Laravel Reverb + Echo)
- Per-customer Tx/Rx monitoring with history
- Upstream interface traffic sampling (every second)
- Session usage tracking with daily/lifetime counters
- RADIUS accounting data aggregation into MongoDB
- Bulk bandwidth sync across all active users

### 🎫 Support Ticketing System

- Customer ticket creation with category selection
- Admin management with status workflow: `Open → In Progress → Resolved → Closed`
- Threaded reply/conversation system with file attachments
- Customer satisfaction ratings
- Auto-diagnostic integration (ONU signal checks on ticket creation)
- Unread ticket notifications for admins

### 👔 Staff & HR Management

- Staff accounts with granular **role-based access control** (Spatie/laravel-permission)
- 50+ individual permissions across all modules
- Salary, bonus, and deduction tracking
- Staff payment report generation with date range filters
- Zone/area assignment for field staff

### 📦 Inventory Management

- Product categories and stock tracking
- Purchase history with cost tracking
- Active/inactive product status management
- Expense aggregation in financial reports

### 🗺️ Network Visualization

- **Interactive Area Map** using Leaflet.js + Mapbox
  - Zone/Sub-Zone/Fiber Route geographic mapping
  - Cable length calculations for fiber routes
  - Customer location plotting on map
- **Network Diagram Builder**
  - Drag-and-drop visual topology editor
  - Save/load multiple diagram configurations

### 📣 Marketing & Messaging

- **Bulk SMS** via MassData API (Bangladesh)
- **WhatsApp messaging** via self-hosted OpenWA gateway
- SMS/WhatsApp template management with variable substitution
- Recipient filtering by zone, status, or custom selection
- Message history with delivery tracking
- **Firebase push notifications** to mobile apps

### 📈 Reports & Analytics

- **Revenue Reports**: Income tracking with date range filters and chart visualization
- **Expense Reports**: Staff payments + inventory purchases aggregation
- **BTRC Reports**: Regulatory compliance exports
- Customer growth metrics and payment method breakdowns
- Visual charts (Pie, Bar, Line) via ApexCharts and Recharts

---

## 🔒 Security & Licensing

### Three-Layer Hardening

```
┌──────────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  LAYER 1: BYTECODE COMPILATION                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  PHP source → Encrypted Zend OpCodes (ionCube/Swoole)      │  │
│  │  Variables, functions, control flows fully obfuscated       │  │
│  │  Reverse engineering: impractical                           │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  LAYER 2: CONTAINERIZED DISTRIBUTION                             │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  App ships as pre-built Docker image (read-only container) │  │
│  │  No source code exposed on client VPS                      │  │
│  │  Internal routes and logic never accessible on-disk        │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  LAYER 3: CRYPTOGRAPHIC DEPENDENCY INJECTION                     │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  App doesn't ask "Is this license valid?"                  │  │
│  │  App asks "Give me the keys I need to function."           │  │
│  │  Without valid JWT → app physically lacks components       │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### The Triple Trap — RS256 JWT Payload

The licensing server signs a JWT with an RS256 private key. The payload injects **four** critical runtime dependencies:

```
┌──────────────────────────────────────────────────────────────────┐
│                     JWT PAYLOAD (RS256)                           │
├──────────────┬───────────────────┬───────────────────────────────┤
│    Trap      │   Payload Key     │   Effect if Missing           │
├──────────────┼───────────────────┼───────────────────────────────┤
│  🔐 Trap 1   │  mikrotik_secret  │  Cannot decrypt router        │
│              │                   │  passwords → no provisioning  │
├──────────────┼───────────────────┼───────────────────────────────┤
│  💰 Trap 2   │  billing_multiplier│ Falls back to 0.0 →          │
│              │                   │  all invoices calculate as $0 │
├──────────────┼───────────────────┼───────────────────────────────┤
│  👥 Trap 3   │  max_users        │  Returns 0 → cannot create   │
│              │                   │  any new customers            │
├──────────────┼───────────────────┼───────────────────────────────┤
│  📡 Trap 4   │  olt_oids         │  Empty OID array → SNMP      │
│              │                   │  queries fail → no OLT sync   │
└──────────────┴───────────────────┴───────────────────────────────┘
```

**License Verification Flow:**

```
  INSTALLATION                              RUNTIME (every 24 hours)
  ────────────────────────                  ─────────────────────────────

  Client → POST /api/license/verify         Client → POST /api/customer/verify
           { license_key, vps_ip }                   { email }
                   │                                         │
           [Valid & IP matches]                    Server signs JWT (RS256)
                   │                                         │
           Lock license to VPS IP              Return signed JWT payload
                   │                                         │
           Allow installation              Client verifies with public key
                                                             │
                                            Inject: API keys, multipliers,
                                                    max_users, OIDs
                                                             │
                                            Hardware sync & billing ✓
```

> **If the runtime token is bypassed or invalid → hardware sync, billing, customer creation, and OLT monitoring ALL fail silently and completely.**

---

## ⚡ Quick Install

> **Prerequisites:** A fresh Ubuntu/Debian VPS with a domain pointed to its IP.

```bash
curl -sSL https://amexcess.me/install.sh | sudo bash
```

That's it. The installer handles everything in under **2 minutes**.

### What the Installer Does

The installer is a compiled **Go binary** (not a shell script) for reliability and cross-platform consistency:

```
 ✔  Root & swap check           Detects low-RAM VPS; provisions 2GB swap on-the-fly
 ✔  Docker installation         Auto-installs Docker if missing
 ✔  License validation          Validates key via API and binds to VPS IP
 ✔  DNS verification            Confirms domain → VPS IP via net.LookupIP
 ✔  Cloudflare DNS provisioning Auto-creates subdomain if using Amexcess DNS
 ✔  SSL scheduling              Caddy auto-provisions Let's Encrypt certificates
 ✔  Credential generation       Secure DB passwords + app keys via crypto/rand
 ✔  Environment generation      Writes .env, docker-compose.yml, Caddyfile
 ✔  Container orchestration     Pulls images, waits for MySQL, runs migrations
 ✔  Optional: OpenWA gateway    Self-hosted WhatsApp messaging (cloned & built)
 ✔  Cron job setup              Laravel scheduler + host update watcher
 ✔  Performance optimization    php artisan optimize + route/view/config cache
```

**Traditional manual setup: 1–2 hours → Automated: under 2 minutes.**

---

## 🚀 Deployment

### Docker Compose Stack

```yaml
services:
  app:        # FrankenPHP + Caddy + Laravel 12  (musabahmed2/isp-portal:latest)
  mysql:      # MySQL 8.0                        (Billing, Customers, Auth)
  redis:      # Redis Alpine                     (Queues, Cache, Sessions)
  mongo:      # MongoDB Latest                   (Telemetry, ONU, Bandwidth)
  openwa:     # OpenWA Gateway                   (WhatsApp messaging)
```

### Service Topology

```
┌──────────────────────────────────────────────────────────────┐
│                    DOCKER COMPOSE NETWORK                     │
│                                                               │
│   :80,:443 ──► [app] ──┬──► [mysql:3306]    Billing DB       │
│                        ├──► [redis:6379]    Queue + Cache     │
│                        ├──► [mongo:27017]   Telemetry DB      │
│                        └──► [openwa:2785]   WhatsApp API      │
│                                                               │
│   Volumes:                                                    │
│   ├── mysql_data      Persistent billing database             │
│   ├── redis_data      Queue state persistence                 │
│   ├── mongo_data      Telemetry retention                     │
│   ├── caddy_data      SSL certificates                        │
│   └── ./storage       Laravel storage (logs, uploads, keys)   │
└──────────────────────────────────────────────────────────────┘
```

### Performance Notes

- **FrankenPHP** runs PHP in worker mode (persistent in-memory execution) — **3–4× throughput** vs. traditional Nginx + PHP-FPM
- **Caddy** auto-provisions and renews Let's Encrypt / ZeroSSL certificates with zero configuration
- **Redis** handles job queues, session caching, and application cache
- **OOM Protection**: automatic swap provisioning prevents crashes on 1 GB RAM VPS instances

---

## 🔄 CI/CD Pipeline

Every push to `main` triggers the full automated pipeline:

```
┌──────────────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE (GitHub Actions)                │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Developer pushes to main                                        │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────────────────────────────┐                        │
│  │         CI — Build & Verify          │                        │
│  │                                      │                        │
│  │  ① Setup PHP 8.2                     │                        │
│  │  ② Composer install (cached)         │                        │
│  │  ③ Setup Node.js 20                  │                        │
│  │  ④ npm install --legacy-peer-deps    │                        │
│  │  ⑤ npm run build (Vite)              │                        │
│  │     └─ 6GB heap for large bundle     │                        │
│  └──────────────────┬──────────────────┘                        │
│                     │                                            │
│                     ▼                                            │
│  ┌─────────────────────────────────────┐                        │
│  │         CD — Containerize & Ship     │                        │
│  │                                      │                        │
│  │  ⑥ Docker Buildx setup              │                        │
│  │  ⑦ Login to Docker Hub              │                        │
│  │  ⑧ Build production image           │                        │
│  │     └─ FrankenPHP base              │                        │
│  │     └─ PHP extensions installed     │                        │
│  │     └─ Composer dependencies        │                        │
│  │     └─ Built frontend assets        │                        │
│  │  ⑨ Push → musabahmed2/isp-portal    │                        │
│  │           :latest on Docker Hub      │                        │
│  └──────────────────┬──────────────────┘                        │
│                     │                                            │
│          ┌──────────┴──────────┐                                │
│          ▼                     ▼                                 │
│    Host watcher cron      Admin dashboard                       │
│    detects update flag    one-click update                      │
│    → docker compose       for client VPS                        │
│      pull + up -d         nodes                                 │
│    (zero downtime)                                              │
└──────────────────────────────────────────────────────────────────┘
```

### Zero-Downtime Client Updates

```
Admin triggers update from Central Server
         │
         ▼
Central API → creates update_pending.flag on client VPS
         │
         ▼
Host watcher cron (runs every minute on client VPS)
         │
  ┌──────┴──────────────────────────────────────────┐
  │  1. Detect update_pending.flag                   │
  │  2. Move flag → update_in_progress.flag          │
  │  3. docker compose pull                          │
  │  4. docker compose up -d --remove-orphans        │
  │  5. Remove progress flag                         │
  │  6. Log result to storage/logs/update_watcher    │
  └─────────────────────────────────────────────────┘
```

---

## 🛠 Technology Stack

### Backend

| Layer | Technology | Purpose |
|---|---|---|
| **Runtime** | FrankenPHP 1.1 | PHP worker-mode, persistent in-memory execution |
| **Web Server** | Caddy | Automatic SSL provisioning, HTTP/3, reverse proxy |
| **Framework** | Laravel 12 | MVC application framework |
| **Database** | MySQL 8.0 | Relational data (billing, customers, auth) |
| **NoSQL** | MongoDB 5.5+ | Time-series telemetry (ONU, bandwidth, metrics) |
| **Cache / Queue** | Redis Alpine | Job queues, session cache, application cache |
| **Auth** | Multi-guard | Separate admin, customer, staff, reseller guards |
| **Permissions** | Spatie Permission | Granular RBAC with 50+ permissions |
| **Modules** | nwidart/laravel-modules | Modular architecture (BkashPG, Invoice) |
| **WebSocket** | Laravel Reverb | Real-time broadcasting (bandwidth, status) |
| **Security** | firebase/php-jwt | RS256 JWT licensing, token verification |

### Frontend

| Layer | Technology | Purpose |
|---|---|---|
| **Admin Views** | Blade Templates | Server-rendered admin pages |
| **Customer Portal** | Inertia.js + React 19 | SPA-like customer experience |
| **Styling** | Bootstrap 5.3 + Tailwind CSS | Responsive UI framework |
| **Charts** | ApexCharts + Recharts | Data visualization |
| **Tables** | DataTables | Advanced sorting, search, pagination |
| **Maps** | Leaflet.js + Mapbox GL | Network coverage mapping |
| **Build Tool** | Vite 6.3 | Fast HMR development, optimized production builds |
| **Icons** | FontAwesome + Iconify | 10k+ icon library |

### Infrastructure

| Layer | Technology | Purpose |
|---|---|---|
| **Containerization** | Docker + Compose | Immutable deployment, service orchestration |
| **CI/CD** | GitHub Actions | Automated build, test, and push pipeline |
| **Installer** | Go (compiled binary) | Zero-config VPS provisioning |
| **DNS** | Cloudflare API | Automated subdomain provisioning |
| **SSL** | Caddy + Let's Encrypt | Automatic certificate management |
| **Backups** | Google Drive API | Scheduled database backups |

### Network Protocols

| Protocol | Library/Tool | Purpose |
|---|---|---|
| **MikroTik API** | evilfreelancer/routeros-api-php | Router provisioning and management |
| **SNMP v1/v2c** | PHP snmp extension | OLT/ONU health polling |
| **RADIUS** | FreeRADIUS (MySQL backend) | Centralized PPPoE authentication |
| **WebSocket** | Pusher.js + Laravel Echo | Real-time client-side updates |

### Mobile

| Platform | Technology | Purpose |
|---|---|---|
| **Admin App** | Flutter (Dart) | Cross-platform admin companion |
| **Customer App** | Flutter (Dart) | Customer self-service on mobile |

---

## 📁 Project Structure

```
isp-portal/
├── app/
│   ├── Console/Commands/          # 34 Artisan commands (billing, sync, monitoring)
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Admin/             # 51 admin controllers
│   │   │   │   └── Marketing/     # SMS, WhatsApp, Push notifications
│   │   │   ├── Customer/          # Customer portal controllers
│   │   │   ├── Staff/             # Staff portal controllers
│   │   │   ├── Reseller/          # Reseller portal controllers
│   │   │   ├── Api/               # REST API controllers (mobile apps)
│   │   │   └── Payments/          # Payment gateway handlers
│   │   └── Middleware/
│   ├── Models/                    # 57 Eloquent models (MySQL + MongoDB)
│   │   └── Radius/                # RADIUS-specific models
│   ├── Services/
│   │   ├── MikrotikService.php    # 122KB — Full RouterOS API integration
│   │   ├── RadiusService.php      # RADIUS auth & accounting
│   │   ├── BillingService.php     # Invoice generation & multiplier logic
│   │   ├── LicensePayloadService.php  # JWT decoding & secret extraction
│   │   ├── FirebaseService.php    # Push notification delivery
│   │   ├── WhatsAppService.php    # OpenWA gateway integration
│   │   └── Olt/
│   │       ├── OltServiceFactory.php  # Vendor abstraction factory
│   │       ├── SnmpService.php        # Low-level SNMP wrapper
│   │       ├── VsolService.php        # V.SOL OLT driver
│   │       ├── BdcomService.php       # BDCOM OLT driver
│   │       ├── RaisecomService.php    # Raisecom OLT driver
│   │       └── DnOpticService.php     # DN-OPTIC OLT driver
│   └── Observers/
├── Modules/
│   ├── BkashPG/                   # bKash payment gateway module
│   └── Invoice/                   # Invoice generation module
├── admin-panel-app/               # Flutter Admin Mobile App
├── customer-panel-appfolder/      # Flutter Customer Mobile App
├── docker/
│   ├── Caddyfile                  # FrankenPHP + Caddy web server config
│   └── entrypoint.sh              # Container initialization script
├── .github/workflows/
│   └── docker-publish.yml         # CI/CD pipeline definition
├── database/
│   ├── migrations/                # MySQL schema migrations
│   └── seeders/                   # Permissions, Roles, OLT Models
├── routes/
│   ├── web.php                    # 933 lines — Full application routing
│   └── console.php                # Scheduled task definitions
├── install.go                     # Go-based zero-config installer (831 lines)
├── Dockerfile                     # Production container definition
├── docker-compose.yml             # Service orchestration
└── host_update_watcher.sh         # Zero-downtime update daemon
```

---

## 📡 Hardware Integrations

### MikroTik RouterOS

| Feature | Protocol | Details |
|---|---|---|
| PPPoE User Management | RouterOS API | Create, update, delete PPP secrets |
| Hotspot Management | RouterOS API | Setup wizard, user CRUD, vouchers |
| VPN Tunnels | RouterOS API | PPTP, L2TP, SSTP tunnel management |
| IP Pool Management | RouterOS API | Pool CRUD, sync with database |
| Address Lists | RouterOS API | Firewall ACLs, block/allow lists |
| DHCP Server | RouterOS API | Server configuration, lease management |
| Interface Monitoring | RouterOS API | Real-time Tx/Rx bandwidth stats |
| Profile Sync | RouterOS API | Auto-sync DB packages → router profiles |

### Fiber OLT Vendors (SNMP)

| Vendor | Enterprise OID | Capabilities |
|---|---|---|
| **V.SOL** | `.1.3.6.1.4.1.37950` | ONU discovery, Rx/Tx power, status, remote reboot |
| **BDCOM** | `.1.3.6.1.4.1.3320` | Full ONU telemetry, optical diagnostics, MAC |
| **Raisecom** | `.1.3.6.1.4.1.8886` | Multi-slot OLT, PON port mapping, ONU sync |
| **DN-OPTIC** | CDATA rebrand | SNMP-based health monitoring |

### SNMP Data Points Collected

```
Per-ONU Metrics:
├── Rx Power (dBm)          Optical receive power from OLT side
├── Tx Power (dBm)          Optical transmit power from ONU side
├── ONU Temperature (°C)    Internal ONU temperature sensor
├── Supply Voltage (V)       ONU power supply voltage
├── Bias Current (mA)       Laser bias current
├── Distance (meters)       Physical distance OLT ↔ ONU
├── MAC Address             Unique hardware identifier
├── Status                  Online / Offline / Dying Gasp / Wire Down
├── Deregistration Reason   Power Off / LOS / OLT Deregister / Unknown
└── Model / Vendor          ONU hardware identification

Per-OLT Metrics:
├── CPU Usage (%)
├── Memory Usage (%)
├── Temperature (°C)
├── System Uptime
└── System Description
```

---

## 📱 Mobile Companion Apps

The platform includes two **Flutter (Dart)** cross-platform mobile applications:

```
┌──────────────────────────────────────────────────────────┐
│                  MOBILE APP ECOSYSTEM                     │
├────────────────────────┬─────────────────────────────────┤
│    Admin Panel App     │     Customer Panel App           │
├────────────────────────┼─────────────────────────────────┤
│  📱 Android + iOS      │  📱 Android + iOS                │
│  🖥️ Windows + macOS    │  🖥️ Windows + macOS              │
│  🌐 Web                │  🌐 Web                          │
├────────────────────────┼─────────────────────────────────┤
│  • Dashboard overview  │  • Personal dashboard            │
│  • Customer management │  • Bill payment                  │
│  • Invoice management  │  • Usage reports                 │
│  • Network monitoring  │  • Support tickets               │
│  • Push notifications  │  • Payment history               │
│  • Quick actions       │  • Profile management            │
└────────────────────────┴─────────────────────────────────┘
```

---

## ⚙️ Configuration

The Go installer auto-generates all configuration files. For manual reference:

| File | Purpose |
|---|---|
| `.env` | Application environment variables (credentials, URLs, keys) |
| `docker-compose.yml` | Container orchestration and service topology |
| `Caddyfile` | Web server + SSL + FrankenPHP configuration |
| `storage/license_token.dat` | Persisted RS256 JWT from licensing server |
| `storage/oauth-public.key` | RSA public key for JWT verification |

Sensitive values (DB passwords, app keys, API credentials) are generated using `crypto/rand` during installation and are **unique per deployment**.

---

## 🔐 User Roles & Access

| Portal | URL Pattern | Access Level |
|---|---|---|
| **Admin** | `/admin/*` | Full platform management, 50+ granular permissions |
| **Customer** | `/customer/*` | Self-service billing, tickets, usage reports |
| **Staff** | `/staff/*` | Limited admin access based on assigned role |
| **Reseller** | `/reseller/*` | Customer management for assigned zones |

---

## 📄 License

This software is **commercially licensed**. Unauthorized redistribution, reverse engineering, or license bypass attempts are prohibited. All deployments are cryptographically bound to a specific VPS IP at installation time.

For licensing inquiries, visit [amexcess.me](https://amexcess.me).

---

<p align="center">
  <strong>Built with ❤️ for ISPs</strong><br/>
  <a href="https://amexcess.me">amexcess.me</a>
</p>
