# Software Requirements Specification (SRS)

## Multi-Vendor Distribution & Sales Management System (MVDSMS)

**Version:** 2.0  
**Date:** February 10, 2026  
**Prepared for:** TecCoop (National Cooperative Society)  
**Developed by:** Temco Bank Web Services  

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Overall Description](#2-overall-description)
3. [System Architecture](#3-system-architecture)
4. [User Roles & Actors](#4-user-roles--actors)
5. [Functional Requirements](#5-functional-requirements)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [Database Design](#7-database-design)
8. [UI/UX Wireframe Descriptions](#8-uiux-wireframe-descriptions)
9. [API Specifications](#9-api-specifications)
10. [Appendices](#10-appendices)

---

## 1. Introduction

### 1.1 Purpose

This document defines the complete software requirements for the **Multi-Vendor Distribution & Sales Management System (MVDSMS)** — a mobile-first platform that digitizes the entire distribution supply chain from order collection at retail shops to final payment reconciliation. The system connects **Vendors (Manufacturers/Suppliers)**, **Distributors**, **Sales Representatives**, **Delivery Personnel**, and **Retail Shops** on a unified platform.

### 1.2 Scope

The system encompasses:

- **Mobile Application** (Android & iOS) for Sales Reps, Delivery Personnel, and Shop Owners
- **Web Applications:**
  - **AdminApp** — System administration and distributor management (port 3000 / backend 8080)
  - **FinanceApp** — Finance, accounting, invoicing, and payment management (port 3001 / backend 8087)
  - **CustomerPortal** — Self-service portal for shop owners and vendors (port 3002 / uses AdminApp backend)
- **Monolithic Backend** (Java / Jakarta EE / WildFly) with RESTful APIs
- **Offline-capable** mobile experience for field operations

### 1.3 Definitions, Acronyms & Abbreviations

| Term | Definition |
|------|-----------|
| **Vendor** | A manufacturer or supplier who provides products to distributors |
| **Distributor** | An entity that warehouses products and manages distribution to retail shops |
| **Sales Rep** | A field agent employed by a distributor who visits shops and collects orders |
| **Shop** | A retail outlet that purchases products from distributors |
| **SKU** | Stock Keeping Unit — a unique identifier for each product variant |
| **DN** | Delivery Note — a document accompanying goods during delivery |
| **PO** | Purchase Order — an order placed by a shop |
| **GRN** | Goods Received Note — acknowledgment of goods received |
| **MVDSMS** | Multi-Vendor Distribution & Sales Management System |
| **EJB** | Enterprise JavaBeans — server-side component architecture |
| **JAX-RS** | Java API for RESTful Web Services |
| **JPA** | Java Persistence API |
| **WAR** | Web Application Archive — Java deployment package |

### 1.4 References

- IEEE 830-1998 (SRS Standard)
- Jakarta EE 10 Specification
- OWASP Mobile Security Guidelines
- PCI-DSS (Payment Card Industry Data Security Standard)

### 1.5 Overview

The remainder of this document details the system's functional and non-functional requirements organized by user role and business process.

---

## 2. Overall Description

### 2.1 Product Perspective

MVDSMS replaces manual, paper-based distribution workflows with a digital system. It sits at the center of the distribution supply chain:

```
┌──────────┐      ┌──────────────┐      ┌───────────┐      ┌────────┐
│  VENDORS │─────▶│ DISTRIBUTORS │─────▶│ SALES REPS│─────▶│ SHOPS  │
│(Suppliers)│      │ (Warehouse)  │      │  (Field)  │      │(Retail)│
└──────────┘      └──────────────┘      └───────────┘      └────────┘
                        │                                       ▲
                        │         ┌──────────────┐              │
                        └────────▶│   DELIVERY   │──────────────┘
                                  │   VEHICLES   │
                                  └──────────────┘
```

### 2.2 Product Functions (Summary)

| # | Function | Description |
|---|----------|-------------|
| F1 | Multi-Vendor Catalog Management | Vendors manage product catalogs; distributors select which products to carry |
| F2 | Distributor Onboarding & Management | Distributors register, manage warehouses, vehicles, and staff |
| F3 | Sales Rep Field Operations | Route planning, shop visits, order collection, GPS tracking |
| F4 | Order Management | Full order lifecycle — creation, approval, fulfillment, delivery |
| F5 | Inventory Management | Real-time stock tracking across warehouses and vehicles |
| F6 | Vehicle & Delivery Management | Load planning, route optimization, delivery tracking |
| F7 | Delivery Note Generation | Automated DN creation when goods are loaded onto vehicles |
| F8 | Invoice Management | Invoice generation, tax calculation, credit management |
| F9 | Payment Collection | Multi-mode payment collection and reconciliation |
| F10 | Reporting & Analytics | Dashboards, sales reports, inventory reports, financial reports |

### 2.3 User Classes and Characteristics

| User Class | Platform | Technical Proficiency | Usage Frequency |
|-----------|----------|----------------------|-----------------|
| System Admin | Web (AdminApp) | High | Daily |
| Vendor | Web (CustomerPortal) | Medium | Daily |
| Distributor | Web (AdminApp + FinanceApp) + Mobile | Medium | Daily |
| Sales Rep | Mobile (primary) | Low–Medium | Daily (field) |
| Delivery Personnel | Mobile | Low | Daily (field) |
| Shop Owner | Mobile + Web (CustomerPortal) | Low | Periodic |

### 2.4 Operating Environment

- **Mobile App:** Android 8.0+ / iOS 14.0+
- **Web Dashboard:** Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- **Backend:** WildFly 31 Application Server + JDK 17
- **Database:** MariaDB 10.6 (shared schema)
- **Cache:** Redis 5.0.2 (Jedis client)
- **Offline Storage:** SQLite (mobile local DB)

### 2.5 Design and Implementation Constraints

- **Monolithic architecture** — single WAR deployment on WildFly application server
- Must support **offline-first** operations for Sales Reps and Delivery Personnel
- Must comply with local **tax regulations** (GST/VAT as applicable)
- Must support **multi-currency** and **multi-language**
- Payment processing must be **PCI-DSS compliant**
- GPS tracking must respect **user privacy** regulations
- Backend must use **Jakarta EE 10** standards (JAX-RS, EJB, JPA/Hibernate)
- All web frontends share a common React + TypeScript codebase pattern

### 2.6 Assumptions and Dependencies

- Users have smartphones with GPS and camera capabilities
- Intermittent internet connectivity in field areas
- Distributors have existing warehouse infrastructure
- Tax rates and rules are configurable per region
- WildFly application server is available in production environment
- MariaDB 10.6 database server is provisioned and accessible

---

## 3. System Architecture

### 3.1 Architecture Overview

The system follows a **monolithic architecture** pattern — a single deployable WAR application running on WildFly application server. Multiple frontend web applications connect to the monolith via RESTful APIs (JAX-RS).

| | AdminApp | FinanceApp | CustomerPortal |
|---|----------|-----------|----------------|
| **Purpose** | Distribution Management & Admin | Finance, Invoicing & Accounting | Vendor & Shop Self-Service |
| **Dev Port (Frontend)** | 3000 | 3001 | 3002 |
| **Backend Port** | 8080 (WildFly) | 8087 (Docker) | None (uses AdminApp backend) |

### 3.2 High-Level Architecture (Monolith)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                  │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  ┌────────────┐  │
│  │  AdminApp   │  │ FinanceApp  │  │CustomerPortal │  │ Mobile App │  │
│  │  React 18.2 │  │ React 18.2  │  │  React 18.2   │  │(React Nat.)│  │
│  │  :3000      │  │  :3001      │  │  :3002        │  │ Android/iOS│  │
│  └──────┬──────┘  └──────┬──────┘  └───────┬───────┘  └─────┬──────┘  │
└─────────┼────────────────┼─────────────────┼─────────────────┼──────────┘
          │                │                 │                 │
          ▼                ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    REVERSE PROXY (Nginx)                                │
│                 SSL Termination (port 443)                              │
└─────────────────────────────┬───────────────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────────────┐
│              MONOLITHIC APPLICATION SERVER (WildFly 31 + JDK 17)       │
│              Deployed as WAR on Jakarta EE 10 Container                │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    JAX-RS REST API Layer                        │    │
│  │              (MicroProfile OpenAPI 3.1 documented)             │    │
│  └─────────────────────────────┬───────────────────────────────────┘    │
│                                │                                        │
│  ┌─────────────────────────────▼───────────────────────────────────┐    │
│  │                    EJB Business Logic Layer                     │    │
│  │                                                                 │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────────┐ │    │
│  │  │ Auth &   │ │ Order    │ │Inventory │ │ Voucher / Invoice │ │    │
│  │  │ User Mgmt│ │ Mgmt     │ │ Mgmt     │ │ Module            │ │    │
│  │  ├──────────┤ ├──────────┤ ├──────────┤ ├───────────────────┤ │    │
│  │  │ Catalog  │ │ Delivery │ │ Vehicle  │ │ Payment           │ │    │
│  │  │ Mgmt     │ │ Mgmt     │ │ Mgmt     │ │ Collection        │ │    │
│  │  ├──────────┤ ├──────────┤ ├──────────┤ ├───────────────────┤ │    │
│  │  │ Org &    │ │ Route    │ │ Report   │ │ Notification      │ │    │
│  │  │ Shop Mgmt│ │ Planning │ │ Engine   │ │ (Email/WhatsApp)  │ │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └───────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │              JPA / Hibernate 6.2.x ORM Layer                   │    │
│  └─────────────────────────────┬───────────────────────────────────┘    │
│                                │                                        │
│  ┌──────────────┐  ┌──────────▼──────┐  ┌──────────────────────────┐   │
│  │ Redis Cache  │  │ MariaDB 10.6    │  │ JasperReports 6.20.6    │   │
│  │ (Jedis 5.0.2)│  │ (JDBC 3.x /    │  │ (PDF/Excel Generation)  │   │
│  │              │  │  MySQL Conn 8.0)│  │                          │   │
│  └──────────────┘  └─────────────────┘  └──────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Jakarta Mail 2.0.1  │  Meta WhatsApp Cloud API  │  FCM/APNs  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.3 Technology Stack

#### Backend

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Java | 17 |
| Platform | Jakarta EE | 10.0.0 |
| App Server | WildFly | 31.0.1 |
| ORM | Hibernate | 6.2.x |
| REST | JAX-RS | (Jakarta EE built-in) |
| EJB | Jakarta Enterprise Beans | (Jakarta EE built-in) |
| Auth | JWT (jjwt) | 0.11.5 |
| Password Hashing | BCrypt (jbcrypt) | 0.4 |
| JSON | Jackson | 2.15.x |
| Reports | JasperReports | 6.20.6 |
| Cache | Redis (Jedis) | 5.0.2 |
| Email | Jakarta Mail | 2.0.1 |
| API Docs | MicroProfile OpenAPI | 3.1 |
| Build | Maven | — |
| Packaging | WAR | — |
| Testing | JUnit 5 + Mockito | 5.10 / 5.7 |

#### Frontend (All 3 Web Apps)

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | React | 18.2 |
| Language | TypeScript | 5.3 |
| Build Tool | Vite | 5.0 |
| Routing | React Router DOM | 6.21 |
| HTTP Client | Axios | 1.6 |
| State Management | Zustand | 4.4 |
| Styling | Tailwind CSS | 3.4 |
| Icons | Lucide React | 0.300 |
| Charts | Recharts | 2.10 |
| Date Utils | date-fns | 3.0 |
| Forms | React Hook Form | 7.49 |
| Server State | TanStack React Query | 5.8 |
| Toasts | React Hot Toast | 2.4 |

#### Database

| Layer | Technology |
|-------|-----------|
| Engine | MariaDB 10.6 |
| Schema | `mvdsms_system` (shared) |
| Connector | MariaDB JDBC 3.x / MySQL Connector 8.0 |

#### Infrastructure

| Layer | Technology |
|-------|-----------|
| Containerization | Docker + Docker Compose |
| Reverse Proxy | Nginx (production) |
| SSL | Host Nginx (port 443) |
| App Server | WildFly 31 + JDK 17 |
| WhatsApp Integration | Meta WhatsApp Cloud API |
| Push Notifications | Firebase Cloud Messaging (FCM) / APNs |
| Maps & Routing | Google Maps API / Mapbox |

### 3.4 Offline-First Architecture

```
┌──────────────────────────────────────────────────┐
│                 MOBILE DEVICE                     │
│                                                    │
│  ┌──────────────────┐   ┌───────────────────┐    │
│  │  React Native    │   │  Local SQLite     │    │
│  │  App Logic       │──▶│  Database         │    │
│  │                  │   │  (Mirror Schema)  │    │
│  └────────┬─────────┘   └─────────┬─────────┘    │
│           │                       │               │
│  ┌────────▼───────────────────────▼────────────┐  │
│  │            Sync Engine                      │  │
│  │                                              │  │
│  │  ┌────────────────┐  ┌───────────────────┐  │  │
│  │  │ Outbound Queue │  │ Conflict Resolver │  │  │
│  │  │ (Orders, Pays) │  │ (Timestamp-based) │  │  │
│  │  └────────┬───────┘  └───────────────────┘  │  │
│  │           │                                  │  │
│  └───────────┼──────────────────────────────────┘  │
└──────────────┼──────────────────────────────────────┘
               │ (When online)
               ▼
┌──────────────────────────────────────────────────┐
│     WildFly Server (Monolith)                    │
│                                                    │
│  ┌──────────────────────────────────────────┐    │
│  │  /api/v1/sync/push   — Receive offline   │    │
│  │  /api/v1/sync/pull   — Send updates      │    │
│  │  /api/v1/sync/status — Check sync state  │    │
│  └──────────────────────────────────────────┘    │
│                                                    │
│  ┌──────────────────────────────────────────┐    │
│  │  Server-Side Validation & Merge          │    │
│  │  • Inventory re-check on order sync      │    │
│  │  • Duplicate detection (idempotency key) │    │
│  │  • Conflict log for manual resolution    │    │
│  └──────────────────────────────────────────┘    │
└──────────────────────────────────────────────────┘
```

**Offline Capabilities:**
- All field operations (order collection, delivery confirmation, payment recording) work **offline**
- Data syncs automatically when connectivity is restored
- Each offline record carries an **idempotency key** (UUID generated on device) to prevent duplicates
- Conflict resolution uses **timestamp-based last-write-wins** with **server-side inventory validation**
- Sync queue persists in SQLite across app restarts
- Offline product catalog and price list cached locally with periodic refresh
- GPS coordinates captured offline and uploaded during sync

---

## 4. User Roles & Actors

### 4.1 System Administrator

- **Description:** Manages the entire platform
- **Permissions:** Full CRUD on all entities, user management, system configuration
- **Key Activities:** Onboard vendors/distributors, manage subscriptions, monitor system health

### 4.2 Vendor (Supplier/Manufacturer)

- **Description:** Provides products to the distribution network
- **Permissions:** Manage own product catalog, view orders for own products, set pricing tiers
- **Key Activities:**
  - Create and maintain product catalog (name, SKU, images, pricing, units)
  - Define pricing tiers for different distributors
  - View sales analytics for own products
  - Manage promotions and discounts

### 4.3 Distributor

- **Description:** Warehouses products and manages distribution to retail shops
- **Permissions:** Manage own inventory, staff, vehicles, orders, invoices, payments
- **Key Activities:**
  - Select products from vendor catalogs to distribute
  - Manage warehouse inventory (stock in, stock out, adjustments)
  - Manage Sales Reps and Delivery Personnel
  - Approve/reject orders
  - Manage delivery vehicles and routes
  - Generate invoices and track payments
  - View comprehensive reports and dashboards

### 4.4 Sales Representative

- **Description:** Field agent who visits retail shops and collects orders
- **Permissions:** View assigned shops/routes, create orders, view product catalog, view own performance
- **Key Activities:**
  - Follow assigned daily route plan
  - Visit shops and collect purchase orders
  - Check product availability in real-time
  - Record shop visit details (GPS, timestamp, photos)
  - Submit orders to the distributor for approval
  - View order history for each shop
  - Collect payments (optional, based on distributor policy)

### 4.5 Delivery Personnel

- **Description:** Drives delivery vehicle and delivers goods to shops
- **Permissions:** View assigned deliveries, confirm delivery, collect payments, record returns
- **Key Activities:**
  - View delivery schedule and route
  - Confirm goods loaded onto vehicle
  - Navigate to shops
  - Hand over goods with delivery note
  - Collect payment or record credit
  - Record returns/rejections
  - Get digital signature from shop owner

### 4.6 Shop Owner (Retailer)

- **Description:** End customer who purchases products
- **Permissions:** View own orders, delivery status, invoices, payment history
- **Key Activities:**
  - Place orders via app (optional — orders can also be placed by Sales Rep)
  - Track delivery status
  - View and download invoices
  - View payment history and outstanding balance
  - Raise complaints/returns

---

## 5. Functional Requirements

### 5.1 Module: User Management & Authentication

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-1.1 | System shall support registration with email/phone + OTP verification | High | All |
| FR-1.2 | System shall support role-based access control (RBAC) with the 6 defined roles | High | Admin |
| FR-1.3 | System shall support JWT-based authentication with refresh tokens | High | All |
| FR-1.4 | System shall allow distributors to invite and manage their Sales Reps and Delivery Personnel | High | Distributor |
| FR-1.5 | System shall support profile management (name, photo, contact, address) | Medium | All |
| FR-1.6 | System shall enforce password policies (min 8 chars, complexity rules) | High | All |
| FR-1.7 | System shall support biometric login (fingerprint/face) on mobile | Medium | Mobile Users |
| FR-1.8 | System shall log all authentication events for audit | High | System |
| FR-1.9 | System shall support multi-tenant isolation — each distributor's data is isolated | High | System |

### 5.2 Module: Multi-Vendor Catalog Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-2.1 | Vendors shall create product catalogs with: name, SKU, description, images, category, unit of measure, weight, dimensions | High | Vendor |
| FR-2.2 | Vendors shall define multiple pricing tiers (MRP, wholesale, distributor price) | High | Vendor |
| FR-2.3 | Vendors shall organize products into hierarchical categories (Category > Sub-category > Product > Variant) | Medium | Vendor |
| FR-2.4 | Distributors shall browse and subscribe to vendor product catalogs | High | Distributor |
| FR-2.5 | Distributors shall set their own selling prices (markup) on subscribed products | High | Distributor |
| FR-2.6 | System shall support product search with filters (category, vendor, price range, availability) | High | All |
| FR-2.7 | System shall support barcode/QR code scanning for product lookup | Medium | Sales Rep, Delivery |
| FR-2.8 | Vendors shall manage product status (active, discontinued, seasonal) | Medium | Vendor |
| FR-2.9 | System shall maintain product image gallery with up to 5 images per product | Low | Vendor |
| FR-2.10 | System shall support bulk product import/export via CSV/Excel | Medium | Vendor, Distributor |

### 5.3 Module: Sales Rep Field Operations

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-3.1 | Distributor shall create and assign daily route plans to Sales Reps | High | Distributor |
| FR-3.2 | Sales Rep shall view assigned route with shop locations on a map | High | Sales Rep |
| FR-3.3 | Sales Rep shall check in at a shop (GPS + timestamp auto-captured) | High | Sales Rep |
| FR-3.4 | System shall enforce geo-fencing — check-in only allowed within configurable radius of shop | Medium | System |
| FR-3.5 | Sales Rep shall browse product catalog and check real-time stock availability while at a shop | High | Sales Rep |
| FR-3.6 | Sales Rep shall create a purchase order on behalf of the shop | High | Sales Rep |
| FR-3.7 | Order shall capture: shop ID, product list (SKU, qty, unit price), discounts, notes, expected delivery date | High | Sales Rep |
| FR-3.8 | Sales Rep shall view order history and outstanding balance for the current shop | High | Sales Rep |
| FR-3.9 | Sales Rep shall capture photos during shop visit (shelf display, promotions) | Low | Sales Rep |
| FR-3.10 | System shall track Sales Rep location periodically (configurable interval, e.g., every 5 min) | Medium | System |
| FR-3.11 | Sales Rep shall check out from shop (auto-captured timestamp) | High | Sales Rep |
| FR-3.12 | All order collection operations shall work **offline** and sync when online | High | Sales Rep |
| FR-3.13 | Sales Rep shall view daily summary (shops visited, orders placed, total value) | Medium | Sales Rep |

### 5.4 Module: Order Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-4.1 | System shall support order lifecycle: **Draft → Submitted → Approved → Processing → Loaded → In Transit → Delivered → Completed / Returned** | High | System |
| FR-4.2 | Sales Rep shall submit collected orders to the distributor | High | Sales Rep |
| FR-4.3 | Distributor shall review and approve/reject/modify submitted orders | High | Distributor |
| FR-4.4 | System shall validate order against available inventory before approval | High | System |
| FR-4.5 | System shall auto-assign order numbers in sequential format (e.g., ORD-2026-00001) | High | System |
| FR-4.6 | Distributor shall be able to partially fulfill an order (adjust quantities) | Medium | Distributor |
| FR-4.7 | System shall notify Sales Rep and Shop when order status changes | High | System |
| FR-4.8 | Shop Owner shall optionally place orders directly via the mobile app | Medium | Shop Owner |
| FR-4.9 | System shall support order cancellation with reason tracking | Medium | Distributor, Shop |
| FR-4.10 | System shall maintain complete order audit trail (who changed what, when) | High | System |
| FR-4.11 | Distributor shall set minimum order value and minimum order quantity rules | Medium | Distributor |
| FR-4.12 | System shall support recurring/repeat orders | Low | Shop Owner |

### 5.5 Module: Inventory Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-5.1 | System shall maintain real-time inventory per warehouse per product (SKU-level) | High | System |
| FR-5.2 | Distributor shall record **Stock In** (goods received from vendor) with GRN | High | Distributor |
| FR-5.3 | System shall auto-deduct stock on order approval (**Stock Reserved**) | High | System |
| FR-5.4 | System shall auto-deduct stock on vehicle loading (**Stock Out**) | High | System |
| FR-5.5 | Distributor shall perform manual stock adjustments with reason codes (damage, expiry, theft, correction) | High | Distributor |
| FR-5.6 | System shall support **batch/lot tracking** with expiry dates | Medium | Distributor |
| FR-5.7 | System shall generate **low stock alerts** when inventory falls below configurable reorder level | High | System |
| FR-5.8 | System shall support multiple warehouse locations per distributor | Medium | Distributor |
| FR-5.9 | System shall track **vehicle inventory** (goods loaded on each delivery vehicle) | High | System |
| FR-5.10 | System shall support stock transfer between warehouses | Low | Distributor |
| FR-5.11 | System shall provide inventory valuation reports (FIFO/weighted average) | Medium | Distributor |
| FR-5.12 | System shall support periodic stock-take / physical inventory count | Medium | Distributor |

### 5.6 Module: Vehicle & Delivery Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-6.1 | Distributor shall register delivery vehicles (registration number, type, capacity, driver assignment) | High | Distributor |
| FR-6.2 | Distributor shall create **delivery trips** by assigning approved orders to a vehicle | High | Distributor |
| FR-6.3 | System shall validate vehicle capacity against total order weight/volume before loading | Medium | System |
| FR-6.4 | System shall generate a **Loading Sheet** listing all items to be loaded per vehicle | High | System |
| FR-6.5 | Warehouse staff shall confirm loading completion (item-by-item or bulk) | High | Distributor |
| FR-6.6 | System shall update vehicle inventory upon loading confirmation | High | System |
| FR-6.7 | Delivery Personnel shall view assigned delivery route with navigation | High | Delivery |
| FR-6.8 | System shall optimize delivery route order for minimum travel distance | Medium | System |
| FR-6.9 | Delivery Personnel shall confirm delivery at each shop with: digital signature, photo proof, timestamp, GPS | High | Delivery |
| FR-6.10 | Delivery Personnel shall record partial deliveries and returns with reason codes | High | Delivery |
| FR-6.11 | System shall track vehicle location in real-time during delivery trips | Medium | System |
| FR-6.12 | Delivery Personnel shall close the trip and reconcile loaded vs delivered vs returned quantities | High | Delivery |

### 5.7 Module: Delivery Note Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-7.1 | System shall auto-generate a **Delivery Note (DN)** for each order loaded onto a vehicle | High | System |
| FR-7.2 | DN shall contain: DN number, date, distributor details, shop details, itemized product list (SKU, description, qty, unit), order reference | High | System |
| FR-7.3 | DN shall be available as a **printable PDF** and **digital format** in the app | High | System |
| FR-7.4 | Delivery Personnel shall present DN to shop owner at the time of delivery | High | Delivery |
| FR-7.5 | Shop Owner shall acknowledge receipt by digital signature on the mobile device | High | Shop Owner |
| FR-7.6 | If partial delivery, DN shall be updated to reflect actual delivered quantities | High | System |
| FR-7.7 | System shall maintain DN history accessible to Distributor, Delivery Personnel, and Shop Owner | Medium | System |
| FR-7.8 | DN shall have a unique sequential number format (e.g., DN-2026-00001) | High | System |

### 5.8 Module: Invoice Management

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-8.1 | System shall auto-generate an **Invoice** upon delivery confirmation | High | System |
| FR-8.2 | Invoice shall contain: invoice number, date, distributor details (with tax ID), shop details, itemized list (product, qty, unit price, tax, line total), subtotal, tax summary, grand total, payment terms | High | System |
| FR-8.3 | System shall calculate applicable taxes (GST/VAT) based on product category and region | High | System |
| FR-8.4 | System shall support configurable tax rules per product category and region | High | Distributor |
| FR-8.5 | Invoice shall be available as **printable PDF** and **digital format** | High | System |
| FR-8.6 | System shall support **credit terms** (e.g., Net 15, Net 30) per shop | High | Distributor |
| FR-8.7 | System shall track invoice status: **Generated → Sent → Partially Paid → Paid → Overdue** | High | System |
| FR-8.8 | System shall auto-flag overdue invoices and send reminders | Medium | System |
| FR-8.9 | Distributor shall issue **Credit Notes** for returns and adjustments | Medium | Distributor |
| FR-8.10 | System shall support invoice numbering in configurable sequential format | High | System |
| FR-8.11 | System shall maintain a complete ledger per shop (all invoices, payments, credits) | High | System |

### 5.9 Module: Payment Collection

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-9.1 | System shall support multiple payment methods: **Cash, Cheque, Bank Transfer, UPI/Mobile Payment, Credit (on account)** | High | System |
| FR-9.2 | Sales Rep or Delivery Personnel shall collect payment at the time of delivery or during shop visits | High | Sales Rep, Delivery |
| FR-9.3 | System shall record payment details: amount, method, reference number, date, collector | High | System |
| FR-9.4 | System shall support **partial payments** against an invoice | High | System |
| FR-9.5 | System shall auto-reconcile payments against invoices (oldest first / manual allocation) | High | System |
| FR-9.6 | System shall generate a **Payment Receipt** (printable + digital) | High | System |
| FR-9.7 | System shall track **outstanding balance** per shop in real-time | High | System |
| FR-9.8 | Distributor shall set **credit limits** per shop | High | Distributor |
| FR-9.9 | System shall block new orders if shop exceeds credit limit or has overdue payments (configurable) | Medium | System |
| FR-9.10 | Delivery Personnel / Sales Rep shall submit daily **cash collection summary** | High | Delivery, Sales Rep |
| FR-9.11 | Distributor shall reconcile collected cash against system records | High | Distributor |
| FR-9.12 | System shall support online payment links sent to shop owners via SMS/WhatsApp | Medium | System |

### 5.10 Module: Notifications & Alerts

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-10.1 | System shall send push notifications for: new orders, order status changes, delivery updates, payment confirmations | High | All |
| FR-10.2 | System shall send SMS notifications for critical events (configurable) | Medium | All |
| FR-10.3 | System shall send email notifications with attached documents (invoices, DNs) | Medium | All |
| FR-10.4 | System shall alert distributors on low stock levels | High | Distributor |
| FR-10.5 | System shall alert distributors on overdue payments | High | Distributor |
| FR-10.6 | System shall notify Sales Reps of their daily route assignments | High | Sales Rep |
| FR-10.7 | Users shall be able to configure notification preferences | Medium | All |

### 5.11 Module: Reporting & Analytics

| ID | Requirement | Priority | Actor |
|----|------------|----------|-------|
| FR-11.1 | **Sales Dashboard:** Total sales, order count, average order value — daily/weekly/monthly | High | Distributor |
| FR-11.2 | **Sales Rep Performance:** Shops visited, orders collected, total value, conversion rate per rep | High | Distributor |
| FR-11.3 | **Product Performance:** Top selling products, slow movers, category-wise sales | High | Distributor, Vendor |
| FR-11.4 | **Inventory Report:** Current stock levels, stock movement, aging analysis | High | Distributor |
| FR-11.5 | **Delivery Report:** On-time delivery rate, returns rate, vehicle utilization | Medium | Distributor |
| FR-11.6 | **Financial Report:** Revenue, outstanding receivables, aging analysis, collection efficiency | High | Distributor |
| FR-11.7 | **Shop Report:** Purchase history, payment history, credit utilization per shop | Medium | Distributor |
| FR-11.8 | **Vendor Report:** Purchase volume, product performance per vendor | Medium | Distributor |
| FR-11.9 | All reports shall support **date range filters** and **export to PDF/Excel** | High | All |
| FR-11.10 | Vendor shall view sales analytics for their products across all distributors | Medium | Vendor |

---

## 6. Non-Functional Requirements

### 6.1 Performance

| ID | Requirement | Target |
|----|------------|--------|
| NFR-1.1 | API response time (95th percentile) | < 500ms |
| NFR-1.2 | Mobile app launch time (cold start) | < 3 seconds |
| NFR-1.3 | Offline-to-online sync time (per order) | < 2 seconds |
| NFR-1.4 | Concurrent users supported | 10,000+ |
| NFR-1.5 | Report generation time | < 10 seconds |
| NFR-1.6 | Search results response time | < 1 second |

### 6.2 Security

| ID | Requirement |
|----|------------|
| NFR-2.1 | All data in transit shall be encrypted using TLS 1.3 |
| NFR-2.2 | All sensitive data at rest shall be encrypted (AES-256) |
| NFR-2.3 | Passwords shall be hashed using bcrypt with salt |
| NFR-2.4 | API shall implement rate limiting (100 requests/min per user) |
| NFR-2.5 | Session tokens shall expire after configurable inactivity period (default: 30 min for web, 7 days for mobile) |
| NFR-2.6 | System shall maintain audit logs for all sensitive operations |
| NFR-2.7 | Payment data handling shall comply with PCI-DSS standards |
| NFR-2.8 | System shall support IP whitelisting for admin access |
| NFR-2.9 | Mobile app shall support remote wipe of local data |

### 6.3 Reliability & Availability

| ID | Requirement | Target |
|----|------------|--------|
| NFR-3.1 | System uptime | 99.9% (excluding planned maintenance) |
| NFR-3.2 | Data backup frequency | Every 6 hours (incremental), daily (full) |
| NFR-3.3 | Recovery Point Objective (RPO) | < 1 hour |
| NFR-3.4 | Recovery Time Objective (RTO) | < 4 hours |
| NFR-3.5 | Zero data loss for financial transactions | Mandatory |

### 6.4 Scalability

| ID | Requirement |
|----|------------|
| NFR-4.1 | WildFly application server shall support vertical scaling and clustered deployment via Docker Compose |
| NFR-4.2 | MariaDB shall support read replicas for reporting queries (JasperReports) |
| NFR-4.3 | System shall support 100+ distributors, 10,000+ shops, 1,000+ sales reps |
| NFR-4.4 | File storage shall scale independently (local filesystem or object storage) |
| NFR-4.5 | Redis cache shall be used for session management and frequently accessed data to reduce DB load |

### 6.5 Usability

| ID | Requirement |
|----|------------|
| NFR-5.1 | Mobile app shall follow platform-specific design guidelines (Material Design / HIG) |
| NFR-5.2 | Order creation flow shall be completable in < 5 taps (after shop check-in) |
| NFR-5.3 | App shall support multi-language (English + configurable local languages) |
| NFR-5.4 | App shall support dark mode |
| NFR-5.5 | All forms shall have input validation with clear error messages |
| NFR-5.6 | App shall display clear offline/online status indicator |

### 6.6 Maintainability

| ID | Requirement |
|----|------------|
| NFR-6.1 | Codebase shall follow layered monolithic architecture (JAX-RS → EJB → JPA) with clear separation of concerns |
| NFR-6.2 | API shall be versioned (e.g., /api/v1/) and documented via MicroProfile OpenAPI 3.1 |
| NFR-6.3 | System shall support feature flags for gradual rollouts via `system_configuration` table |
| NFR-6.4 | Minimum 80% unit test coverage for EJB business logic (JUnit 5 + Mockito) |
| NFR-6.5 | CI/CD pipeline for automated testing, Maven build, WAR packaging, and Docker deployment |
| NFR-6.6 | All JPA entities shall use Hibernate annotations with proper lazy/eager loading strategies |
| NFR-6.7 | Database migrations shall be managed via versioned SQL scripts |

---

## 7. Database Design

**Database:** MariaDB 10.6  
**Schema:** `mvdsms_system` (shared)  
**ORM:** Hibernate 6.2.x (JPA)  
**Naming Convention:** `snake_case` for all table and column names  
**Primary Keys:** `BIGINT AUTO_INCREMENT` (all tables)  
**Audit Columns:** All tables include `created_by`, `updated_by`, `created_at`, `updated_at`

### 7.1 Entity Relationship Overview

```
                    ┌─────────────────────┐
                    │ general_user_profile │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
┌─────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ user_role       │ │ user_address     │ │ user_document    │
└─────────────────┘ └──────────────────┘ └──────────────────┘
          │
          ▼
┌──────────────────────────┐       ┌──────────────────────────┐
│general_organization_     │──────▶│ org_address              │
│profile                   │       ├──────────────────────────┤
└──────────┬───────────────┘       │ org_contact_person       │
           │                       ├──────────────────────────┤
           │                       │ org_bank_account         │
           │                       ├──────────────────────────┤
           │                       │ org_document             │
           │                       └──────────────────────────┘
           │
     ┌─────┼──────────┬──────────────┬───────────────┐
     │     │          │              │               │
     ▼     ▼          ▼              ▼               ▼
┌────────┐┌────────┐┌──────────┐┌─────────┐┌──────────────┐
│Vendor  ││Distri- ││ Shop     ││Warehouse││ Vehicle      │
│        ││butor   ││          ││         ││              │
└───┬────┘└───┬────┘└────┬─────┘└────┬────┘└──────┬───────┘
    │         │          │           │            │
    ▼         │          │           ▼            ▼
┌────────┐   │     ┌────▼─────┐┌─────────┐┌──────────────┐
│Product │   │     │  Order   ││Inventory ││Delivery Trip │
│Category│   │     └────┬─────┘└─────────┘└──────┬───────┘
│Product │   │          │                         │
└────────┘   │     ┌────▼─────┐            ┌─────▼────────┐
             │     │Order Item│            │Delivery Note │
             │     └──────────┘            └──────┬───────┘
             │                                    │
             │     ┌──────────────────────────────┘
             │     │
             ▼     ▼
      ┌──────────────────┐     ┌──────────────────┐
      │  Voucher (Header)│────▶│  Voucher Line    │
      │  (INV/REC/DN/CN) │     │  (Detail Items)  │
      └────────┬─────────┘     └──────────────────┘
               │
               ▼
      ┌──────────────────┐
      │  Payment         │
      └──────────────────┘
```

---

### 7.2 Group A: User & Authentication Tables

#### `general_user_profile`
Central user identity table for all system actors.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `uuid` | VARCHAR(36) | UNIQUE, NOT NULL | Public-facing identifier |
| `username` | VARCHAR(100) | UNIQUE, NOT NULL | Login username |
| `email` | VARCHAR(255) | UNIQUE | Email address |
| `phone` | VARCHAR(20) | UNIQUE | Phone number |
| `password_hash` | VARCHAR(255) | NOT NULL | BCrypt hashed password |
| `first_name` | VARCHAR(100) | NOT NULL | |
| `last_name` | VARCHAR(100) | | |
| `display_name` | VARCHAR(200) | | Full display name |
| `avatar_url` | VARCHAR(500) | | Profile photo URL |
| `gender` | ENUM('MALE','FEMALE','OTHER') | | |
| `date_of_birth` | DATE | | |
| `preferred_language` | VARCHAR(10) | DEFAULT 'en' | e.g., en, hi, ta |
| `timezone` | VARCHAR(50) | DEFAULT 'UTC' | |
| `is_active` | TINYINT(1) | DEFAULT 1 | Account active flag |
| `is_email_verified` | TINYINT(1) | DEFAULT 0 | |
| `is_phone_verified` | TINYINT(1) | DEFAULT 0 | |
| `last_login_at` | DATETIME | | |
| `last_login_ip` | VARCHAR(45) | | IPv4/IPv6 |
| `failed_login_count` | INT | DEFAULT 0 | For account lockout |
| `locked_until` | DATETIME | | Account lock expiry |
| `password_changed_at` | DATETIME | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `general_role`
System roles definition.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `role_code` | VARCHAR(50) | UNIQUE, NOT NULL | e.g., ADMIN, VENDOR, DISTRIBUTOR, SALES_REP, DELIVERY, SHOP_OWNER |
| `role_name` | VARCHAR(100) | NOT NULL | Display name |
| `description` | VARCHAR(500) | | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `user_role`
Many-to-many: users can have multiple roles.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `role_id` | BIGINT | FK → general_role, NOT NULL | |
| `organization_id` | BIGINT | FK → general_organization_profile, NULLABLE | Role scoped to org |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `assigned_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `assigned_by` | BIGINT | FK → general_user_profile | |
| UNIQUE | | (user_id, role_id, organization_id) | |

#### `general_permission`
Granular permissions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `permission_code` | VARCHAR(100) | UNIQUE, NOT NULL | e.g., ORDER_CREATE, INVENTORY_VIEW |
| `permission_name` | VARCHAR(200) | NOT NULL | |
| `module` | VARCHAR(50) | NOT NULL | e.g., ORDER, INVENTORY, PAYMENT |
| `description` | VARCHAR(500) | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `role_permission`
Many-to-many: roles to permissions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `role_id` | BIGINT | FK → general_role, NOT NULL | |
| `permission_id` | BIGINT | FK → general_permission, NOT NULL | |
| UNIQUE | | (role_id, permission_id) | |

#### `user_address`
Multiple addresses per user.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `address_type` | ENUM('HOME','WORK','OTHER') | NOT NULL | |
| `address_line_1` | VARCHAR(255) | NOT NULL | |
| `address_line_2` | VARCHAR(255) | | |
| `city` | VARCHAR(100) | | |
| `state_province` | VARCHAR(100) | | |
| `postal_code` | VARCHAR(20) | | |
| `country` | VARCHAR(100) | | |
| `latitude` | DECIMAL(10,8) | | GPS latitude |
| `longitude` | DECIMAL(11,8) | | GPS longitude |
| `is_default` | TINYINT(1) | DEFAULT 0 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `user_document`
KYC and identity documents.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `document_type` | VARCHAR(50) | NOT NULL | e.g., NIC, PASSPORT, DRIVING_LICENSE |
| `document_number` | VARCHAR(100) | | |
| `document_url` | VARCHAR(500) | | Scanned copy URL |
| `issue_date` | DATE | | |
| `expiry_date` | DATE | | |
| `is_verified` | TINYINT(1) | DEFAULT 0 | |
| `verified_by` | BIGINT | FK → general_user_profile | |
| `verified_at` | DATETIME | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `user_session`
Active sessions and tokens.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `token_hash` | VARCHAR(255) | NOT NULL | Hashed JWT refresh token |
| `device_type` | ENUM('WEB','ANDROID','IOS') | | |
| `device_id` | VARCHAR(255) | | Unique device identifier |
| `device_name` | VARCHAR(255) | | e.g., "Samsung Galaxy S24" |
| `ip_address` | VARCHAR(45) | | |
| `user_agent` | VARCHAR(500) | | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `expires_at` | DATETIME | NOT NULL | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `user_otp`
OTP verification for registration and password reset.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile | |
| `otp_code` | VARCHAR(10) | NOT NULL | |
| `otp_type` | ENUM('REGISTRATION','PASSWORD_RESET','PHONE_VERIFY','EMAIL_VERIFY') | NOT NULL | |
| `target` | VARCHAR(255) | NOT NULL | Email or phone |
| `is_used` | TINYINT(1) | DEFAULT 0 | |
| `expires_at` | DATETIME | NOT NULL | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.3 Group B: Organization & Entity Tables

#### `general_organization_profile`
Central organization entity — vendors, distributors, and shops are all organizations.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `uuid` | VARCHAR(36) | UNIQUE, NOT NULL | Public-facing identifier |
| `org_code` | VARCHAR(50) | UNIQUE, NOT NULL | System-generated code (e.g., DST-0001) |
| `org_type` | ENUM('VENDOR','DISTRIBUTOR','SHOP') | NOT NULL | Organization type |
| `legal_name` | VARCHAR(255) | NOT NULL | Registered legal name |
| `trade_name` | VARCHAR(255) | | Trading / display name |
| `tax_id` | VARCHAR(100) | | Tax identification number |
| `business_registration_no` | VARCHAR(100) | | Business registration |
| `logo_url` | VARCHAR(500) | | Organization logo |
| `website` | VARCHAR(255) | | |
| `industry` | VARCHAR(100) | | e.g., FMCG, Pharma, Electronics |
| `description` | TEXT | | |
| `owner_user_id` | BIGINT | FK → general_user_profile, NOT NULL | Primary owner |
| `parent_org_id` | BIGINT | FK → general_organization_profile, NULLABLE | For hierarchy (e.g., vendor → distributor relationship) |
| `credit_limit` | DECIMAL(15,2) | DEFAULT 0.00 | Credit limit (for shops) |
| `payment_terms_days` | INT | DEFAULT 0 | Net payment terms |
| `currency_code` | VARCHAR(3) | DEFAULT 'LKR' | Default currency |
| `is_verified` | TINYINT(1) | DEFAULT 0 | Admin-verified flag |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `verified_by` | BIGINT | FK → general_user_profile | |
| `verified_at` | DATETIME | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `org_address`
Multiple addresses per organization.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `address_type` | ENUM('REGISTERED','WAREHOUSE','BRANCH','SHOP') | NOT NULL | |
| `address_line_1` | VARCHAR(255) | NOT NULL | |
| `address_line_2` | VARCHAR(255) | | |
| `city` | VARCHAR(100) | | |
| `state_province` | VARCHAR(100) | | |
| `postal_code` | VARCHAR(20) | | |
| `country` | VARCHAR(100) | | |
| `latitude` | DECIMAL(10,8) | | GPS latitude |
| `longitude` | DECIMAL(11,8) | | GPS longitude |
| `geo_fence_radius_m` | INT | DEFAULT 100 | Geo-fence radius in meters |
| `is_default` | TINYINT(1) | DEFAULT 0 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `org_contact_person`
Contact persons per organization.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `user_id` | BIGINT | FK → general_user_profile, NULLABLE | Linked user if registered |
| `contact_name` | VARCHAR(200) | NOT NULL | |
| `designation` | VARCHAR(100) | | e.g., Owner, Manager, Accountant |
| `phone` | VARCHAR(20) | | |
| `email` | VARCHAR(255) | | |
| `is_primary` | TINYINT(1) | DEFAULT 0 | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `org_bank_account`
Bank account details per organization.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `bank_name` | VARCHAR(200) | NOT NULL | |
| `branch_name` | VARCHAR(200) | | |
| `account_number` | VARCHAR(50) | NOT NULL | |
| `account_name` | VARCHAR(200) | | |
| `swift_code` | VARCHAR(20) | | |
| `iban` | VARCHAR(50) | | |
| `account_type` | ENUM('CURRENT','SAVINGS') | DEFAULT 'CURRENT' | |
| `is_default` | TINYINT(1) | DEFAULT 0 | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `org_document`
Organization documents (licenses, certificates, tax registrations).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `document_type` | VARCHAR(50) | NOT NULL | e.g., TAX_CERT, BUSINESS_LICENSE, TRADE_LICENSE |
| `document_number` | VARCHAR(100) | | |
| `document_url` | VARCHAR(500) | | |
| `issue_date` | DATE | | |
| `expiry_date` | DATE | | |
| `is_verified` | TINYINT(1) | DEFAULT 0 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `org_user_assignment`
Links users to organizations with specific roles (e.g., sales rep assigned to distributor).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `role_in_org` | VARCHAR(50) | NOT NULL | e.g., OWNER, SALES_REP, DRIVER, WAREHOUSE_STAFF |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `assigned_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `assigned_by` | BIGINT | FK → general_user_profile | |
| UNIQUE | | (organization_id, user_id, role_in_org) | |

#### `distributor_shop`
Many-to-many relationship between distributors and shops.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | Org of type DISTRIBUTOR |
| `shop_id` | BIGINT | FK → general_organization_profile, NOT NULL | Org of type SHOP |
| `credit_limit_override` | DECIMAL(15,2) | | Distributor-specific credit limit |
| `payment_terms_override` | INT | | Distributor-specific terms |
| `price_tier` | VARCHAR(50) | DEFAULT 'STANDARD' | e.g., STANDARD, PREMIUM, WHOLESALE |
| `assigned_sales_rep_id` | BIGINT | FK → general_user_profile | Default sales rep for this shop |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (distributor_id, shop_id) | |

---

### 7.4 Group C: Product & Catalog Tables

#### `product_category`
Hierarchical product categories.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `parent_id` | BIGINT | FK → product_category, NULLABLE | Self-referencing for hierarchy |
| `category_code` | VARCHAR(50) | UNIQUE, NOT NULL | |
| `category_name` | VARCHAR(200) | NOT NULL | |
| `description` | VARCHAR(500) | | |
| `image_url` | VARCHAR(500) | | |
| `sort_order` | INT | DEFAULT 0 | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `product`
Master product catalog (owned by vendors).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `vendor_id` | BIGINT | FK → general_organization_profile, NOT NULL | Org of type VENDOR |
| `category_id` | BIGINT | FK → product_category, NOT NULL | |
| `sku` | VARCHAR(50) | UNIQUE, NOT NULL | Stock Keeping Unit |
| `product_name` | VARCHAR(255) | NOT NULL | |
| `description` | TEXT | | |
| `unit_of_measure` | VARCHAR(20) | NOT NULL | e.g., PCS, KG, LTR, BOX, PACK |
| `pieces_per_unit` | INT | DEFAULT 1 | e.g., 12 pieces per box |
| `weight_kg` | DECIMAL(10,3) | | Weight per unit |
| `volume_cbm` | DECIMAL(10,6) | | Volume per unit (cubic meters) |
| `mrp` | DECIMAL(15,2) | | Maximum retail price |
| `wholesale_price` | DECIMAL(15,2) | | Default wholesale price |
| `cost_price` | DECIMAL(15,2) | | Vendor cost price |
| `barcode` | VARCHAR(50) | | EAN/UPC barcode |
| `qr_code` | VARCHAR(255) | | QR code data |
| `tax_category` | VARCHAR(50) | | For tax calculation |
| `is_taxable` | TINYINT(1) | DEFAULT 1 | |
| `min_order_qty` | DECIMAL(10,2) | DEFAULT 1 | |
| `reorder_level` | DECIMAL(10,2) | DEFAULT 0 | |
| `shelf_life_days` | INT | | |
| `status` | ENUM('ACTIVE','INACTIVE','DISCONTINUED','SEASONAL') | DEFAULT 'ACTIVE' | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `product_image`
Multiple images per product.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `image_url` | VARCHAR(500) | NOT NULL | |
| `thumbnail_url` | VARCHAR(500) | | |
| `sort_order` | INT | DEFAULT 0 | |
| `is_primary` | TINYINT(1) | DEFAULT 0 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `product_price_tier`
Pricing tiers per product (vendor-defined).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `tier_name` | VARCHAR(50) | NOT NULL | e.g., WHOLESALE, RETAIL, PREMIUM |
| `min_quantity` | DECIMAL(10,2) | DEFAULT 1 | Minimum qty for this tier |
| `unit_price` | DECIMAL(15,2) | NOT NULL | |
| `effective_from` | DATE | | |
| `effective_to` | DATE | | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `distributor_product`
Products a distributor has subscribed to from vendor catalogs, with distributor-specific pricing.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `selling_price` | DECIMAL(15,2) | NOT NULL | Distributor's selling price |
| `distributor_cost` | DECIMAL(15,2) | | What distributor pays vendor |
| `markup_percentage` | DECIMAL(5,2) | | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (distributor_id, product_id) | |

---

### 7.5 Group D: Warehouse & Inventory Tables

#### `warehouse`
Distributor warehouse locations.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `warehouse_code` | VARCHAR(50) | NOT NULL | |
| `warehouse_name` | VARCHAR(200) | NOT NULL | |
| `address_id` | BIGINT | FK → org_address | |
| `manager_user_id` | BIGINT | FK → general_user_profile | |
| `is_default` | TINYINT(1) | DEFAULT 0 | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (distributor_id, warehouse_code) | |

#### `inventory`
Current stock levels per warehouse per product.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `warehouse_id` | BIGINT | FK → warehouse, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `quantity_on_hand` | DECIMAL(15,3) | DEFAULT 0 | Physical stock |
| `quantity_reserved` | DECIMAL(15,3) | DEFAULT 0 | Reserved for approved orders |
| `quantity_available` | DECIMAL(15,3) | GENERATED (on_hand - reserved) | Available for new orders |
| `reorder_level` | DECIMAL(15,3) | DEFAULT 0 | Low stock threshold |
| `reorder_quantity` | DECIMAL(15,3) | DEFAULT 0 | Suggested reorder qty |
| `last_stock_take_at` | DATETIME | | Last physical count date |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (warehouse_id, product_id) | |

#### `inventory_batch`
Batch/lot tracking with expiry.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `inventory_id` | BIGINT | FK → inventory, NOT NULL | |
| `batch_number` | VARCHAR(100) | NOT NULL | |
| `manufacturing_date` | DATE | | |
| `expiry_date` | DATE | | |
| `quantity` | DECIMAL(15,3) | NOT NULL | Qty in this batch |
| `cost_price` | DECIMAL(15,2) | | Cost per unit for this batch |
| `status` | ENUM('AVAILABLE','EXPIRED','DAMAGED','QUARANTINE') | DEFAULT 'AVAILABLE' | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `inventory_movement`
Audit trail of all stock movements.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `inventory_id` | BIGINT | FK → inventory, NOT NULL | |
| `movement_type` | ENUM('STOCK_IN','STOCK_OUT','ADJUSTMENT','TRANSFER_IN','TRANSFER_OUT','RETURN','LOADING','UNLOADING') | NOT NULL | |
| `reference_type` | VARCHAR(50) | | e.g., GRN, ORDER, ADJUSTMENT, DELIVERY_TRIP |
| `reference_id` | BIGINT | | ID of the referenced entity |
| `quantity_before` | DECIMAL(15,3) | NOT NULL | |
| `quantity_change` | DECIMAL(15,3) | NOT NULL | Positive = in, Negative = out |
| `quantity_after` | DECIMAL(15,3) | NOT NULL | |
| `batch_id` | BIGINT | FK → inventory_batch, NULLABLE | |
| `reason_code` | VARCHAR(50) | | e.g., DAMAGE, EXPIRY, THEFT, CORRECTION, SALE |
| `notes` | TEXT | | |
| `performed_by` | BIGINT | FK → general_user_profile, NOT NULL | |
| `performed_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `goods_received_note`
GRN for stock received from vendors.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `grn_number` | VARCHAR(50) | UNIQUE, NOT NULL | e.g., GRN-2026-00001 |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `vendor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `warehouse_id` | BIGINT | FK → warehouse, NOT NULL | |
| `purchase_order_ref` | VARCHAR(100) | | Vendor PO reference |
| `received_date` | DATE | NOT NULL | |
| `total_items` | INT | | |
| `total_quantity` | DECIMAL(15,3) | | |
| `status` | ENUM('DRAFT','RECEIVED','VERIFIED','CANCELLED') | DEFAULT 'DRAFT' | |
| `notes` | TEXT | | |
| `received_by` | BIGINT | FK → general_user_profile | |
| `verified_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `goods_received_note_item`
Line items for GRN.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `grn_id` | BIGINT | FK → goods_received_note, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `quantity_expected` | DECIMAL(15,3) | | |
| `quantity_received` | DECIMAL(15,3) | NOT NULL | |
| `quantity_rejected` | DECIMAL(15,3) | DEFAULT 0 | |
| `unit_cost` | DECIMAL(15,2) | | |
| `batch_number` | VARCHAR(100) | | |
| `expiry_date` | DATE | | |
| `rejection_reason` | VARCHAR(255) | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.6 Group E: Order Management Tables

#### `sales_order`
Purchase orders collected from shops.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `order_number` | VARCHAR(50) | UNIQUE, NOT NULL | e.g., ORD-2026-00001 |
| `idempotency_key` | VARCHAR(36) | UNIQUE | UUID from mobile for offline dedup |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `shop_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `sales_rep_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `warehouse_id` | BIGINT | FK → warehouse | Fulfillment warehouse |
| `order_date` | DATE | NOT NULL | |
| `expected_delivery_date` | DATE | | |
| `status` | ENUM('DRAFT','SUBMITTED','APPROVED','PROCESSING','LOADED','IN_TRANSIT','DELIVERED','COMPLETED','CANCELLED','RETURNED') | DEFAULT 'DRAFT' | |
| `subtotal` | DECIMAL(15,2) | DEFAULT 0 | |
| `discount_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `tax_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `grand_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `notes` | TEXT | | |
| `cancellation_reason` | VARCHAR(500) | | |
| `approved_by` | BIGINT | FK → general_user_profile | |
| `approved_at` | DATETIME | | |
| `created_offline` | TINYINT(1) | DEFAULT 0 | Created in offline mode |
| `synced_at` | DATETIME | | When synced to server |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `sales_order_item`
Line items for each order.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `order_id` | BIGINT | FK → sales_order, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `quantity_ordered` | DECIMAL(15,3) | NOT NULL | Qty requested by shop |
| `quantity_approved` | DECIMAL(15,3) | | Qty approved by distributor |
| `quantity_loaded` | DECIMAL(15,3) | | Qty loaded onto vehicle |
| `quantity_delivered` | DECIMAL(15,3) | | Qty actually delivered |
| `quantity_returned` | DECIMAL(15,3) | DEFAULT 0 | Qty returned |
| `unit_price` | DECIMAL(15,2) | NOT NULL | |
| `discount_amount` | DECIMAL(15,2) | DEFAULT 0 | |
| `tax_rate` | DECIMAL(5,2) | DEFAULT 0 | Tax percentage |
| `tax_amount` | DECIMAL(15,2) | DEFAULT 0 | |
| `line_total` | DECIMAL(15,2) | NOT NULL | (qty × price) - discount + tax |
| `notes` | VARCHAR(500) | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `order_status_history`
Complete audit trail of order status changes.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `order_id` | BIGINT | FK → sales_order, NOT NULL | |
| `from_status` | VARCHAR(20) | | Previous status |
| `to_status` | VARCHAR(20) | NOT NULL | New status |
| `changed_by` | BIGINT | FK → general_user_profile, NOT NULL | |
| `change_reason` | VARCHAR(500) | | |
| `changed_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.7 Group F: Vehicle & Delivery Tables

#### `vehicle`
Delivery vehicles registered by distributors.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `registration_number` | VARCHAR(50) | NOT NULL | |
| `vehicle_type` | VARCHAR(50) | | e.g., VAN, TRUCK, MOTORCYCLE, THREE_WHEELER |
| `make_model` | VARCHAR(100) | | |
| `max_weight_capacity_kg` | DECIMAL(10,2) | | |
| `max_volume_capacity_cbm` | DECIMAL(10,4) | | |
| `fuel_type` | VARCHAR(20) | | |
| `default_driver_id` | BIGINT | FK → general_user_profile | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (distributor_id, registration_number) | |

#### `delivery_trip`
A delivery trip groups multiple orders for a single vehicle run.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `trip_number` | VARCHAR(50) | UNIQUE, NOT NULL | e.g., TRIP-2026-00001 |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `vehicle_id` | BIGINT | FK → vehicle, NOT NULL | |
| `driver_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `warehouse_id` | BIGINT | FK → warehouse, NOT NULL | Loading warehouse |
| `planned_date` | DATE | NOT NULL | |
| `status` | ENUM('PLANNED','LOADING','LOADED','IN_TRANSIT','COMPLETED','CANCELLED') | DEFAULT 'PLANNED' | |
| `total_orders` | INT | DEFAULT 0 | |
| `total_weight_kg` | DECIMAL(10,2) | DEFAULT 0 | |
| `total_volume_cbm` | DECIMAL(10,4) | DEFAULT 0 | |
| `loading_started_at` | DATETIME | | |
| `loading_completed_at` | DATETIME | | |
| `departed_at` | DATETIME | | |
| `completed_at` | DATETIME | | |
| `odometer_start` | DECIMAL(10,1) | | |
| `odometer_end` | DECIMAL(10,1) | | |
| `notes` | TEXT | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `delivery_trip_order`
Orders assigned to a delivery trip.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `trip_id` | BIGINT | FK → delivery_trip, NOT NULL | |
| `order_id` | BIGINT | FK → sales_order, NOT NULL | |
| `delivery_sequence` | INT | NOT NULL | Stop order in route |
| `estimated_arrival` | DATETIME | | |
| `actual_arrival` | DATETIME | | |
| `status` | ENUM('PENDING','DELIVERED','PARTIAL','FAILED','SKIPPED') | DEFAULT 'PENDING' | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (trip_id, order_id) | |

#### `vehicle_inventory`
Tracks what is currently loaded on each vehicle.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `trip_id` | BIGINT | FK → delivery_trip, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `quantity_loaded` | DECIMAL(15,3) | NOT NULL | |
| `quantity_delivered` | DECIMAL(15,3) | DEFAULT 0 | |
| `quantity_returned` | DECIMAL(15,3) | DEFAULT 0 | |
| `quantity_remaining` | DECIMAL(15,3) | GENERATED (loaded - delivered - returned) | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (trip_id, product_id) | |

#### `delivery_confirmation`
Proof of delivery per order.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `trip_order_id` | BIGINT | FK → delivery_trip_order, NOT NULL | |
| `order_id` | BIGINT | FK → sales_order, NOT NULL | |
| `delivered_at` | DATETIME | NOT NULL | |
| `receiver_name` | VARCHAR(200) | | |
| `receiver_phone` | VARCHAR(20) | | |
| `signature_url` | VARCHAR(500) | | Digital signature image |
| `photo_url` | VARCHAR(500) | | Delivery proof photo |
| `latitude` | DECIMAL(10,8) | | GPS at delivery |
| `longitude` | DECIMAL(11,8) | | |
| `notes` | TEXT | | |
| `confirmed_by` | BIGINT | FK → general_user_profile, NOT NULL | Delivery person |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `delivery_return`
Items returned during delivery.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `delivery_confirmation_id` | BIGINT | FK → delivery_confirmation, NOT NULL | |
| `product_id` | BIGINT | FK → product, NOT NULL | |
| `quantity_returned` | DECIMAL(15,3) | NOT NULL | |
| `reason_code` | ENUM('DAMAGED','WRONG_ITEM','EXCESS','EXPIRED','REJECTED','OTHER') | NOT NULL | |
| `reason_notes` | VARCHAR(500) | | |
| `photo_url` | VARCHAR(500) | | Photo of returned goods |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.8 Group G: Voucher Module (Invoices, Receipts, Delivery Notes, Credit Notes)

The voucher module uses a **unified header + line items** pattern for all financial and logistics documents.

#### `voucher_type`
Defines the types of vouchers in the system.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `type_code` | VARCHAR(20) | UNIQUE, NOT NULL | e.g., INV, REC, DN, CN, DN_LOAD, GRN |
| `type_name` | VARCHAR(100) | NOT NULL | e.g., Invoice, Receipt, Delivery Note, Credit Note, Loading Note |
| `description` | VARCHAR(500) | | |
| `prefix` | VARCHAR(10) | NOT NULL | Number prefix, e.g., INV-, REC-, DN-, CN- |
| `affects_balance` | ENUM('DEBIT','CREDIT','NONE') | NOT NULL | How it affects shop ledger |
| `requires_approval` | TINYINT(1) | DEFAULT 0 | |
| `is_printable` | TINYINT(1) | DEFAULT 1 | |
| `jasper_template` | VARCHAR(200) | | JasperReports template file name |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `voucher_number_sequence`
Auto-incrementing number sequences per voucher type per organization per fiscal year.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `voucher_type_id` | BIGINT | FK → voucher_type, NOT NULL | |
| `organization_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `fiscal_year` | VARCHAR(10) | NOT NULL | e.g., 2026 |
| `current_number` | BIGINT | DEFAULT 0 | Last used number |
| `format_pattern` | VARCHAR(100) | | e.g., {PREFIX}{YEAR}-{SEQ:5} → INV-2026-00001 |
| UNIQUE | | (voucher_type_id, organization_id, fiscal_year) | |

#### `voucher`
Unified header for all document types (Invoice, Receipt, Delivery Note, Credit Note, Loading Note).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `voucher_number` | VARCHAR(50) | UNIQUE, NOT NULL | Auto-generated from sequence |
| `voucher_type_id` | BIGINT | FK → voucher_type, NOT NULL | |
| `order_id` | BIGINT | FK → sales_order, NULLABLE | Related order |
| `trip_id` | BIGINT | FK → delivery_trip, NULLABLE | Related delivery trip |
| `parent_voucher_id` | BIGINT | FK → voucher, NULLABLE | e.g., Credit Note references original Invoice |
| `issuer_org_id` | BIGINT | FK → general_organization_profile, NOT NULL | Distributor issuing the voucher |
| `counterparty_org_id` | BIGINT | FK → general_organization_profile, NOT NULL | Shop / Vendor receiving |
| `voucher_date` | DATE | NOT NULL | |
| `due_date` | DATE | | Payment due date (for invoices) |
| `reference_number` | VARCHAR(100) | | External reference |
| `currency_code` | VARCHAR(3) | DEFAULT 'LKR' | |
| `subtotal` | DECIMAL(15,2) | DEFAULT 0 | |
| `discount_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `tax_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `grand_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `amount_paid` | DECIMAL(15,2) | DEFAULT 0 | Running total of payments (for invoices) |
| `balance_due` | DECIMAL(15,2) | DEFAULT 0 | grand_total - amount_paid |
| `status` | ENUM('DRAFT','GENERATED','SENT','PARTIALLY_PAID','PAID','OVERDUE','CANCELLED','VOID') | DEFAULT 'DRAFT' | |
| `notes` | TEXT | | |
| `internal_notes` | TEXT | | Internal-only notes |
| `terms_and_conditions` | TEXT | | Printed on document |
| `pdf_url` | VARCHAR(500) | | Generated PDF file URL |
| `approved_by` | BIGINT | FK → general_user_profile | |
| `approved_at` | DATETIME | | |
| `sent_at` | DATETIME | | When sent to counterparty |
| `sent_via` | ENUM('EMAIL','WHATSAPP','SMS','PRINT','APP') | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `voucher_line`
Line items for any voucher type.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `voucher_id` | BIGINT | FK → voucher, NOT NULL | |
| `line_number` | INT | NOT NULL | Sequence within voucher |
| `product_id` | BIGINT | FK → product, NULLABLE | NULL for service/misc lines |
| `description` | VARCHAR(500) | | Line description (auto-filled from product or manual) |
| `quantity` | DECIMAL(15,3) | DEFAULT 0 | |
| `unit_of_measure` | VARCHAR(20) | | |
| `unit_price` | DECIMAL(15,2) | DEFAULT 0 | |
| `discount_percentage` | DECIMAL(5,2) | DEFAULT 0 | |
| `discount_amount` | DECIMAL(15,2) | DEFAULT 0 | |
| `tax_rate` | DECIMAL(5,2) | DEFAULT 0 | |
| `tax_amount` | DECIMAL(15,2) | DEFAULT 0 | |
| `line_total` | DECIMAL(15,2) | DEFAULT 0 | |
| `batch_id` | BIGINT | FK → inventory_batch, NULLABLE | |
| `notes` | VARCHAR(500) | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `voucher_tax_summary`
Tax breakdown per voucher (for multi-tax scenarios).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `voucher_id` | BIGINT | FK → voucher, NOT NULL | |
| `tax_name` | VARCHAR(100) | NOT NULL | e.g., VAT, GST, CGST, SGST |
| `tax_rate` | DECIMAL(5,2) | NOT NULL | |
| `taxable_amount` | DECIMAL(15,2) | NOT NULL | |
| `tax_amount` | DECIMAL(15,2) | NOT NULL | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `voucher_attachment`
File attachments on vouchers (photos, signed copies, etc.).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `voucher_id` | BIGINT | FK → voucher, NOT NULL | |
| `file_name` | VARCHAR(255) | NOT NULL | |
| `file_url` | VARCHAR(500) | NOT NULL | |
| `file_type` | VARCHAR(50) | | MIME type |
| `file_size_bytes` | BIGINT | | |
| `description` | VARCHAR(255) | | |
| `uploaded_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### Voucher Type Usage Summary

| Voucher Type Code | Name | When Created | Affects Balance | Key Fields |
|---|---|---|---|---|
| `INV` | Invoice | After delivery confirmation | DEBIT (shop owes) | due_date, tax details, payment terms |
| `REC` | Receipt | When payment is collected | CREDIT (shop pays) | payment_method, reference_number |
| `DN` | Delivery Note | When goods loaded onto vehicle | NONE | delivery details, receiver signature |
| `CN` | Credit Note | For returns/adjustments | CREDIT (reduces shop debt) | parent_voucher_id → original INV |
| `DN_LOAD` | Loading Note | When vehicle is loaded | NONE | vehicle details, loading checklist |
| `GRN_V` | GRN Voucher | When goods received from vendor | NONE | vendor reference, received quantities |

---

### 7.9 Group H: Payment Tables

#### `payment`
Individual payment transactions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `payment_number` | VARCHAR(50) | UNIQUE, NOT NULL | e.g., PAY-2026-00001 |
| `receipt_voucher_id` | BIGINT | FK → voucher, NULLABLE | Linked receipt voucher |
| `invoice_voucher_id` | BIGINT | FK → voucher, NULLABLE | Invoice being paid |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `shop_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `amount` | DECIMAL(15,2) | NOT NULL | |
| `payment_method` | ENUM('CASH','CHEQUE','BANK_TRANSFER','UPI','MOBILE_PAYMENT','CREDIT') | NOT NULL | |
| `reference_number` | VARCHAR(100) | | Transaction/transfer reference |
| `cheque_number` | VARCHAR(50) | | |
| `cheque_date` | DATE | | |
| `cheque_bank` | VARCHAR(200) | | |
| `status` | ENUM('COLLECTED','VERIFIED','DEPOSITED','CLEARED','BOUNCED','CANCELLED') | DEFAULT 'COLLECTED' | |
| `collected_by` | BIGINT | FK → general_user_profile, NOT NULL | Sales rep or delivery person |
| `collected_at` | DATETIME | NOT NULL | |
| `verified_by` | BIGINT | FK → general_user_profile | |
| `verified_at` | DATETIME | | |
| `deposited_at` | DATETIME | | |
| `cleared_at` | DATETIME | | |
| `latitude` | DECIMAL(10,8) | | GPS at collection |
| `longitude` | DECIMAL(11,8) | | |
| `notes` | TEXT | | |
| `created_offline` | TINYINT(1) | DEFAULT 0 | |
| `idempotency_key` | VARCHAR(36) | UNIQUE | For offline dedup |
| `created_by` | BIGINT | FK → general_user_profile | |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `payment_allocation`
Allocates payments to specific invoices (supports partial payments and multi-invoice payments).

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `payment_id` | BIGINT | FK → payment, NOT NULL | |
| `invoice_voucher_id` | BIGINT | FK → voucher, NOT NULL | Invoice being paid |
| `allocated_amount` | DECIMAL(15,2) | NOT NULL | |
| `allocated_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `allocated_by` | BIGINT | FK → general_user_profile | |

#### `daily_collection_summary`
Daily cash/cheque collection summary submitted by field staff.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `collector_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `collection_date` | DATE | NOT NULL | |
| `total_cash_collected` | DECIMAL(15,2) | DEFAULT 0 | |
| `total_cheques_collected` | DECIMAL(15,2) | DEFAULT 0 | |
| `total_digital_collected` | DECIMAL(15,2) | DEFAULT 0 | |
| `total_amount` | DECIMAL(15,2) | DEFAULT 0 | |
| `cash_handed_over` | DECIMAL(15,2) | DEFAULT 0 | Cash physically handed to office |
| `variance` | DECIMAL(15,2) | DEFAULT 0 | Difference (shortage/excess) |
| `status` | ENUM('SUBMITTED','VERIFIED','DISCREPANCY','CLOSED') | DEFAULT 'SUBMITTED' | |
| `verified_by` | BIGINT | FK → general_user_profile | |
| `verified_at` | DATETIME | | |
| `notes` | TEXT | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (collector_id, collection_date) | |

#### `shop_ledger`
Running ledger per shop per distributor — every financial transaction posted here.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `shop_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `transaction_date` | DATE | NOT NULL | |
| `voucher_id` | BIGINT | FK → voucher, NULLABLE | Related voucher |
| `payment_id` | BIGINT | FK → payment, NULLABLE | Related payment |
| `transaction_type` | ENUM('INVOICE','PAYMENT','CREDIT_NOTE','ADJUSTMENT','OPENING_BALANCE') | NOT NULL | |
| `debit_amount` | DECIMAL(15,2) | DEFAULT 0 | Increases balance owed |
| `credit_amount` | DECIMAL(15,2) | DEFAULT 0 | Decreases balance owed |
| `running_balance` | DECIMAL(15,2) | NOT NULL | Balance after this transaction |
| `description` | VARCHAR(500) | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.10 Group I: Route & Field Operations Tables

#### `route_plan`
Daily route plans for sales reps.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `distributor_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `sales_rep_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `route_date` | DATE | NOT NULL | |
| `route_name` | VARCHAR(200) | | |
| `total_shops` | INT | DEFAULT 0 | |
| `shops_visited` | INT | DEFAULT 0 | |
| `status` | ENUM('PLANNED','IN_PROGRESS','COMPLETED','CANCELLED') | DEFAULT 'PLANNED' | |
| `started_at` | DATETIME | | |
| `completed_at` | DATETIME | | |
| `created_by` | BIGINT | FK → general_user_profile | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |
| UNIQUE | | (sales_rep_id, route_date) | |

#### `route_plan_stop`
Individual shop stops in a route plan.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `route_plan_id` | BIGINT | FK → route_plan, NOT NULL | |
| `shop_id` | BIGINT | FK → general_organization_profile, NOT NULL | |
| `visit_sequence` | INT | NOT NULL | Order of visit |
| `status` | ENUM('PENDING','CHECKED_IN','CHECKED_OUT','SKIPPED') | DEFAULT 'PENDING' | |
| `check_in_at` | DATETIME | | |
| `check_out_at` | DATETIME | | |
| `check_in_latitude` | DECIMAL(10,8) | | |
| `check_in_longitude` | DECIMAL(11,8) | | |
| `check_out_latitude` | DECIMAL(10,8) | | |
| `check_out_longitude` | DECIMAL(11,8) | | |
| `visit_duration_minutes` | INT | | Computed from check-in/out |
| `order_placed` | TINYINT(1) | DEFAULT 0 | Was an order placed? |
| `payment_collected` | TINYINT(1) | DEFAULT 0 | Was payment collected? |
| `skip_reason` | VARCHAR(255) | | If skipped |
| `notes` | TEXT | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `shop_visit_photo`
Photos captured during shop visits.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `route_stop_id` | BIGINT | FK → route_plan_stop, NOT NULL | |
| `photo_type` | ENUM('SHELF','PROMOTION','STOREFRONT','OTHER') | | |
| `photo_url` | VARCHAR(500) | NOT NULL | |
| `caption` | VARCHAR(255) | | |
| `latitude` | DECIMAL(10,8) | | |
| `longitude` | DECIMAL(11,8) | | |
| `captured_at` | DATETIME | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `gps_tracking_log`
Periodic GPS tracking of field staff.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `latitude` | DECIMAL(10,8) | NOT NULL | |
| `longitude` | DECIMAL(11,8) | NOT NULL | |
| `accuracy_meters` | DECIMAL(8,2) | | GPS accuracy |
| `speed_kmh` | DECIMAL(6,2) | | |
| `battery_level` | INT | | Device battery % |
| `is_online` | TINYINT(1) | | Was device online when captured |
| `recorded_at` | DATETIME | NOT NULL | Timestamp on device |
| `synced_at` | DATETIME | | When synced to server |

---

### 7.11 Group J: Tax & Configuration Tables

#### `tax_rate`
Configurable tax rates.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `tax_name` | VARCHAR(100) | NOT NULL | e.g., VAT, GST, Service Tax |
| `tax_code` | VARCHAR(20) | UNIQUE, NOT NULL | |
| `rate_percentage` | DECIMAL(5,2) | NOT NULL | |
| `applicable_to` | VARCHAR(50) | | Product category or ALL |
| `region` | VARCHAR(100) | | Geographic applicability |
| `effective_from` | DATE | NOT NULL | |
| `effective_to` | DATE | | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `system_configuration`
Key-value system settings.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `config_key` | VARCHAR(100) | UNIQUE, NOT NULL | |
| `config_value` | TEXT | | |
| `config_type` | ENUM('STRING','INTEGER','BOOLEAN','JSON','DECIMAL') | DEFAULT 'STRING' | |
| `description` | VARCHAR(500) | | |
| `organization_id` | BIGINT | FK → general_organization_profile, NULLABLE | NULL = global |
| `updated_by` | BIGINT | FK → general_user_profile | |
| `updated_at` | DATETIME | ON UPDATE CURRENT_TIMESTAMP | |

#### `currency`
Supported currencies.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `currency_code` | VARCHAR(3) | UNIQUE, NOT NULL | ISO 4217 |
| `currency_name` | VARCHAR(100) | NOT NULL | |
| `symbol` | VARCHAR(10) | | |
| `decimal_places` | INT | DEFAULT 2 | |
| `is_active` | TINYINT(1) | DEFAULT 1 | |

---

### 7.12 Group K: Notification & Audit Tables

#### `notification`
System notifications.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | Recipient |
| `title` | VARCHAR(255) | NOT NULL | |
| `message` | TEXT | NOT NULL | |
| `notification_type` | VARCHAR(50) | | e.g., ORDER, DELIVERY, PAYMENT, STOCK, SYSTEM |
| `reference_type` | VARCHAR(50) | | Entity type |
| `reference_id` | BIGINT | | Entity ID |
| `channel` | ENUM('PUSH','SMS','EMAIL','WHATSAPP','IN_APP') | DEFAULT 'IN_APP' | |
| `is_read` | TINYINT(1) | DEFAULT 0 | |
| `read_at` | DATETIME | | |
| `sent_at` | DATETIME | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `notification_preference`
User notification preferences.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `notification_type` | VARCHAR(50) | NOT NULL | |
| `push_enabled` | TINYINT(1) | DEFAULT 1 | |
| `sms_enabled` | TINYINT(1) | DEFAULT 0 | |
| `email_enabled` | TINYINT(1) | DEFAULT 1 | |
| `whatsapp_enabled` | TINYINT(1) | DEFAULT 0 | |
| UNIQUE | | (user_id, notification_type) | |

#### `audit_log`
System-wide audit trail.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile | |
| `action` | VARCHAR(50) | NOT NULL | e.g., CREATE, UPDATE, DELETE, LOGIN, APPROVE |
| `entity_type` | VARCHAR(100) | NOT NULL | Table/entity name |
| `entity_id` | BIGINT | | |
| `old_values` | JSON | | Previous state (JSON) |
| `new_values` | JSON | | New state (JSON) |
| `ip_address` | VARCHAR(45) | | |
| `user_agent` | VARCHAR(500) | | |
| `performed_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

#### `sync_log`
Mobile offline sync tracking.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGINT | PK, AUTO_INCREMENT | |
| `user_id` | BIGINT | FK → general_user_profile, NOT NULL | |
| `device_id` | VARCHAR(255) | | |
| `sync_direction` | ENUM('PUSH','PULL') | NOT NULL | |
| `entity_type` | VARCHAR(50) | | e.g., ORDER, PAYMENT, GPS |
| `records_synced` | INT | DEFAULT 0 | |
| `records_failed` | INT | DEFAULT 0 | |
| `conflicts_detected` | INT | DEFAULT 0 | |
| `sync_started_at` | DATETIME | | |
| `sync_completed_at` | DATETIME | | |
| `error_details` | TEXT | | |
| `created_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |

---

### 7.13 Table Count Summary

| Group | Tables | Count |
|-------|--------|-------|
| A: User & Auth | general_user_profile, general_role, user_role, general_permission, role_permission, user_address, user_document, user_session, user_otp | 9 |
| B: Organization | general_organization_profile, org_address, org_contact_person, org_bank_account, org_document, org_user_assignment, distributor_shop | 7 |
| C: Product & Catalog | product_category, product, product_image, product_price_tier, distributor_product | 5 |
| D: Warehouse & Inventory | warehouse, inventory, inventory_batch, inventory_movement, goods_received_note, goods_received_note_item | 6 |
| E: Order Management | sales_order, sales_order_item, order_status_history | 3 |
| F: Vehicle & Delivery | vehicle, delivery_trip, delivery_trip_order, vehicle_inventory, delivery_confirmation, delivery_return | 6 |
| G: Voucher Module | voucher_type, voucher_number_sequence, voucher, voucher_line, voucher_tax_summary, voucher_attachment | 6 |
| H: Payment | payment, payment_allocation, daily_collection_summary, shop_ledger | 4 |
| I: Route & Field Ops | route_plan, route_plan_stop, shop_visit_photo, gps_tracking_log | 4 |
| J: Tax & Config | tax_rate, system_configuration, currency | 3 |
| K: Notification & Audit | notification, notification_preference, audit_log, sync_log | 4 |
| **Total** | | **57 tables** |

---

## 8. UI/UX Wireframe Descriptions

All web applications use **React 18.2 + TypeScript + Tailwind CSS 3.4** with **Lucide React** icons, **Recharts** for data visualization, **React Hook Form** for form handling, and **TanStack React Query** for server state management.

### 8.1 Mobile App — Sales Rep (React Native)

#### 8.1.1 Home / Dashboard Screen
- **Top Bar:** Profile avatar, notification bell (with badge count), sync status indicator (green=online, amber=syncing, red=offline with pending count)
- **Summary Cards Row:** Today's stats — Shops to visit (with progress ring), Orders placed, Total order value, Payments collected
- **Route Map (Mapbox/Google Maps):** Interactive map showing today's assigned shops with pins (color-coded: visited=green, pending=blue, skipped=red, current=pulsing orange)
- **Shop List (FlatList):** Scrollable list below map — shop trade_name, address, last visit date, outstanding balance badge (red if overdue), distance from current location
- **Offline Banner:** Persistent banner when offline — "Offline Mode — {n} items pending sync"
- **Bottom Navigation:** Home | Orders | Products | Payments | More

#### 8.1.2 Shop Visit Screen
- **Shop Header Card:** Shop trade_name, org_code, address (from org_address), contact person (from org_contact_person), outstanding balance badge (from shop_ledger running_balance)
- **Credit Status Bar:** Visual bar showing credit utilization (used / credit_limit from distributor_shop)
- **Action Buttons Grid (2×2):** New Order, View History, Collect Payment, View Ledger
- **Quick Stats Row:** Last order date, average order value, total orders this month, days since last visit
- **Visit Log:** Auto-captured GPS coordinates, timestamp, check-in button (validates geo-fence from org_address.geo_fence_radius_m)
- **Photo Capture:** Camera button to capture shelf/storefront photos (stored in shop_visit_photo)

#### 8.1.3 Order Creation Screen
- **Search Bar:** Search products by name, SKU, or barcode scan (camera icon) — queries distributor_product catalog
- **Category Tabs:** Horizontal scrollable tabs from product_category hierarchy
- **Product Grid/List Toggle:** Product thumbnail (from product_image), product_name, SKU, available stock (from inventory.quantity_available), selling_price (from distributor_product), unit_of_measure
- **Add to Cart Modal:** Quantity stepper (respects min_order_qty), unit price (editable with discount), tax display, line total preview, notes field
- **Cart Summary (Floating Bottom Sheet):** Item count, subtotal, tax total, discount total, grand total, "Review Order" button
- **Order Review Screen:** Full itemized list, shop details, expected delivery date picker, order notes, "Submit Order" button — creates sales_order + sales_order_item records

#### 8.1.4 Order History Screen
- **Filter Bar:** Date range picker (date-fns), status dropdown (ENUM values from sales_order.status), shop search
- **Order Cards (FlatList):** Order number, order_date, shop trade_name, grand_total, status badge (color-coded chip)
- **Order Detail (Full Screen):** Itemized list (from sales_order_item), order timeline (from order_status_history), related vouchers (DN, Invoice from voucher table)

#### 8.1.5 Payment Collection Screen
- **Shop Selector:** Search or auto-filled from current shop visit
- **Outstanding Invoices List:** From voucher table (type=INV, status=SENT/PARTIALLY_PAID/OVERDUE) — invoice_number, voucher_date, grand_total, balance_due, days overdue (red highlight)
- **Payment Form:** Amount input (pre-filled with balance_due), payment method selector (CASH/CHEQUE/UPI/BANK_TRANSFER), reference number, cheque details (conditional), notes
- **Receipt Preview:** Auto-generated receipt summary before confirmation
- **GPS auto-capture** on payment submission

#### 8.1.6 Daily Summary Screen
- **Stats Cards:** Total shops visited, total orders placed, total order value, total payments collected
- **Shop Visit Timeline:** Chronological list of check-in/check-out times per shop
- **Cash Summary:** Total cash collected, total cheques, total digital — "Submit Daily Summary" button (creates daily_collection_summary)

### 8.2 Mobile App — Delivery Personnel (React Native)

#### 8.2.1 Home / Trip Screen
- **Active Trip Card:** trip_number, vehicle registration_number, driver name, planned_date, status badge
- **Trip Stats Row:** Total deliveries, completed, pending, total loaded weight vs capacity (progress bar)
- **Delivery List (FlatList):** Ordered by delivery_sequence — shop trade_name, address, order grand_total, voucher DN number, status indicator (PENDING=gray, DELIVERED=green, PARTIAL=amber, FAILED=red)
- **Map View Tab:** Route with numbered delivery stops, current location, turn-by-turn navigation button (opens Google Maps/Waze)
- **Loading Checklist Button:** View loading sheet (vehicle_inventory items)

#### 8.2.2 Loading Confirmation Screen
- **Vehicle Info:** registration_number, vehicle_type, driver name
- **Items Checklist:** Product name, SKU, quantity to load (from vehicle_inventory.quantity_loaded), checkbox per item
- **Scan Mode:** Barcode scanner to verify items during loading
- **Weight/Volume Summary:** Total loaded vs vehicle capacity (max_weight_capacity_kg, max_volume_capacity_cbm)
- **Confirm Loading Button:** Updates delivery_trip status to LOADED

#### 8.2.3 Delivery Confirmation Screen
- **Shop Details:** trade_name, address, contact person, phone (tap to call)
- **Items Checklist:** Product name, expected qty (from sales_order_item.quantity_loaded), actual delivered qty (editable stepper), return qty with reason code dropdown (DAMAGED/WRONG_ITEM/EXCESS/EXPIRED/REJECTED/OTHER)
- **Return Photo:** Camera capture for returned items (stored in delivery_return.photo_url)
- **Payment Section:** Invoice amount due, payment method selector, amount collected input
- **Signature Pad:** Full-width digital signature capture area (saved to delivery_confirmation.signature_url)
- **Delivery Photo:** Camera capture for proof of delivery (saved to delivery_confirmation.photo_url)
- **GPS Auto-Capture:** Latitude/longitude recorded automatically
- **Confirm Button:** "Complete Delivery" with summary modal — creates delivery_confirmation, delivery_return records, updates voucher (DN) status

#### 8.2.4 Trip Summary Screen
- **Completion Stats:** Delivered / Total, on-time %, returns count
- **Reconciliation:** Loaded vs Delivered vs Returned per product (from vehicle_inventory)
- **Cash Collection Summary:** Total cash, cheques, digital payments collected during trip
- **Close Trip Button:** Finalizes delivery_trip, submits daily_collection_summary

### 8.3 Web — AdminApp (React 18.2, port 3000)

Distributor administration and operations management.

#### 8.3.1 Sidebar Navigation
- **Logo + Org Name** (from general_organization_profile.trade_name)
- **Menu Items:** Dashboard, Organizations (Vendors/Shops), Users & Roles, Products & Catalog, Orders, Inventory, Vehicles & Delivery, Route Planning, Reports, Settings
- **Collapsible** with icon-only mode
- **User Profile Dropdown** (bottom): Profile, Preferences, Logout

#### 8.3.2 Main Dashboard
- **KPI Cards (Recharts):** Today's orders (count + value), pending approvals, active deliveries, outstanding receivables, low stock alerts
- **Sales Trend Chart (Line):** Daily/weekly/monthly sales with comparison to previous period
- **Top Products Chart (Horizontal Bar):** Top 10 products by revenue
- **Sales Rep Performance Chart (Bar):** Orders collected, value, shops visited per rep
- **Payment Collection Chart (Pie):** Cash vs Cheque vs Digital vs Credit breakdown
- **Recent Activity Feed:** Latest orders, deliveries, payments — with timestamps and status badges
- **Alerts Panel (Right Sidebar):** Low stock items (from inventory where quantity_available < reorder_level), overdue invoices (from voucher where status=OVERDUE), pending order approvals

#### 8.3.3 Organization Management Screen
- **Tabs:** Vendors | Shops | All Organizations
- **Data Table (TanStack Table):** org_code, trade_name, org_type, contact person, phone, credit_limit, outstanding balance, is_active, is_verified — sortable, filterable, searchable
- **Create/Edit Slide-over Panel:** Multi-tab form — Basic Info (legal_name, trade_name, tax_id), Addresses (org_address CRUD), Contact Persons (org_contact_person CRUD), Bank Accounts (org_bank_account CRUD), Documents (org_document upload), Settings (credit_limit, payment_terms_days, price_tier)
- **Shop Detail View:** Full profile, assigned sales rep, order history, ledger (shop_ledger), outstanding balance, credit utilization chart

#### 8.3.4 Order Management Screen
- **Data Table:** order_number, order_date, shop trade_name, sales_rep display_name, grand_total, status — with status filter tabs (All | Pending Approval | Approved | In Transit | Delivered)
- **Bulk Actions Toolbar:** Approve selected, Reject selected, Assign to trip, Export to Excel
- **Order Detail Slide-over:** Full order details (sales_order + sales_order_item), status timeline (order_status_history), related vouchers (DN, Invoice), actions (Approve/Reject/Modify quantities)
- **Modify Order Modal:** Adjust quantity_approved per item, add notes, recalculate totals

#### 8.3.5 Inventory Management Screen
- **Stock Overview Table:** Product SKU, product_name, warehouse, quantity_on_hand, quantity_reserved, quantity_available, reorder_level — with conditional formatting (red row if available < reorder_level)
- **Stock Alerts Tab:** Filtered view of items below reorder level
- **Stock In (GRN) Form:** Select vendor, warehouse, add products with quantities, batch numbers, expiry dates — creates goods_received_note + goods_received_note_item + inventory_movement
- **Stock Adjustment Form:** Select product + warehouse, adjustment type (DAMAGE/EXPIRY/THEFT/CORRECTION), quantity change, reason notes — creates inventory_movement
- **Movement History:** Filterable log from inventory_movement — date, type, product, quantity change, performed_by
- **Batch Tracking Tab:** View inventory_batch records — batch_number, manufacturing_date, expiry_date, quantity, status

#### 8.3.6 Vehicle & Delivery Management Screen
- **Vehicles Tab:** Data table of vehicles — registration_number, vehicle_type, capacity, default driver, is_active. CRUD form for vehicle management.
- **Delivery Trips Tab:**
  - **Trip Planner:** Create new trip — select vehicle, driver, planned_date, warehouse
  - **Order Assignment:** Drag-and-drop approved orders onto trip, auto-calculate delivery_sequence
  - **Capacity Indicator:** Visual progress bar — total_weight_kg / max_weight_capacity_kg, total_volume_cbm / max_volume_capacity_cbm
  - **Loading Sheet (Print):** JasperReports PDF — consolidated product list per vehicle (from vehicle_inventory)
  - **Trip Tracking:** Real-time map with vehicle location (from gps_tracking_log), delivery status per stop
  - **Trip Reconciliation:** Loaded vs Delivered vs Returned summary after trip completion

#### 8.3.7 Route Planning Screen
- **Calendar View:** Monthly calendar showing route plans per sales rep
- **Route Builder:** Select sales_rep, date, drag shops from available list to route — set visit_sequence
- **Map Preview:** Route visualization with numbered stops and estimated travel time
- **Route Templates:** Save and reuse common route patterns
- **Route Performance:** Actual vs planned visits, visit duration, orders placed per stop

#### 8.3.8 Settings Screen
- **Organization Profile:** Edit general_organization_profile fields
- **Tax Configuration:** CRUD for tax_rate records
- **Voucher Number Sequences:** Configure format_pattern per voucher_type
- **System Configuration:** Key-value settings from system_configuration
- **User Management:** Invite users, assign roles (user_role), manage org_user_assignment

### 8.4 Web — FinanceApp (React 18.2, port 3001)

Finance, accounting, invoicing, and payment management.

#### 8.4.1 Finance Dashboard
- **KPI Cards:** Total revenue (MTD/YTD), outstanding receivables, overdue amount, collection rate %
- **Receivables Aging Chart (Stacked Bar):** Current, 1-30 days, 31-60 days, 61-90 days, 90+ days
- **Daily Collection Trend (Line Chart):** Cash + Cheque + Digital collections over time
- **Top Debtors Table:** Shops with highest outstanding balances

#### 8.4.2 Voucher Management Screen
- **Tabs per Voucher Type:** Invoices (INV) | Receipts (REC) | Delivery Notes (DN) | Credit Notes (CN) | Loading Notes (DN_LOAD)
- **Data Table:** voucher_number, voucher_date, counterparty trade_name, grand_total, balance_due, status, due_date
- **Voucher Detail View:** Header info, line items (voucher_line), tax summary (voucher_tax_summary), attachments (voucher_attachment), payment history (payment_allocation), PDF preview/download (JasperReports)
- **Create Credit Note:** Select parent invoice, specify return items/amounts, auto-post to shop_ledger
- **Send Voucher:** Email (Jakarta Mail) / WhatsApp (Meta API) / Print (JasperReports PDF)

#### 8.4.3 Payment Management Screen
- **Payments Table:** payment_number, collected_at, shop trade_name, amount, payment_method, status, collected_by
- **Payment Verification Workflow:** List of COLLECTED payments → Verify (mark as VERIFIED) → Deposit (mark as DEPOSITED)
- **Cheque Management:** Filter cheques, track cheque_date, mark as CLEARED or BOUNCED
- **Daily Collection Reconciliation:** View daily_collection_summary per collector — total_cash_collected vs cash_handed_over, variance highlighting
- **Payment Allocation:** Allocate payments to specific invoices (payment_allocation), auto-allocate (oldest first) or manual

#### 8.4.4 Shop Ledger Screen
- **Shop Selector:** Search by trade_name or org_code
- **Ledger Table:** Transaction date, voucher/payment reference, description, debit, credit, running_balance — from shop_ledger
- **Statement Export:** Generate PDF/Excel statement for date range (JasperReports)
- **Balance Summary:** Current balance, credit limit, available credit, overdue amount

#### 8.4.5 Financial Reports (JasperReports)
- **Sales Report:** Revenue by period, product, category, shop, sales rep — exportable PDF/Excel
- **Receivables Aging Report:** Outstanding by shop with aging buckets
- **Collection Report:** Collections by method, collector, period
- **Tax Report:** Tax collected by category, rate, period
- **Profit & Loss Summary:** Revenue - Cost of Goods - Discounts = Gross Profit

### 8.5 Web — CustomerPortal (React 18.2, port 3002)

Self-service portal for shop owners and vendors.

#### 8.5.1 Shop Owner View
- **Dashboard:** Outstanding balance, recent orders, recent deliveries, recent payments
- **Order History:** View all orders with status tracking timeline
- **Invoice & Statement:** View/download invoices (PDF), view ledger statement
- **Place Order (Optional):** Browse distributor_product catalog, create order
- **Payment History:** View all payments, download receipts
- **Profile Management:** Update org details, contact persons, addresses

#### 8.5.2 Vendor View
- **Dashboard:** Total products listed, orders containing vendor products, revenue from products
- **Product Catalog Management:** CRUD on product table — add/edit products, manage images, set pricing tiers
- **Sales Analytics:** Product performance across distributors, top-selling products, trend charts
- **Distributor Network:** View which distributors carry vendor products (from distributor_product)

---

## 9. API Specifications

### 9.1 API Design Principles

- **RESTful** architecture implemented via **JAX-RS** (Jakarta EE built-in)
- **JSON** request/response format via **Jackson 2.15.x**
- **Versioned:** `/api/v1/`
- **Authenticated:** Bearer token (JWT via jjwt 0.11.5)
- **Paginated:** All list endpoints support `?page=0&size=20&sort=createdAt,desc`
- **Filterable:** Query parameters for filtering and sorting
- **Documented:** MicroProfile OpenAPI 3.1 annotations on all endpoints
- **Error Format:** Standardized error response `{ "status": 400, "error": "Bad Request", "message": "...", "timestamp": "..." }`

### 9.2 Core API Endpoints (70+)

#### 9.2.1 Authentication & Session (6 endpoints)
```
POST   /api/v1/auth/register                    Register new user (general_user_profile)
POST   /api/v1/auth/login                       Login → returns JWT access + refresh token
POST   /api/v1/auth/refresh-token               Refresh expired access token
POST   /api/v1/auth/forgot-password             Send OTP to email/phone (user_otp)
POST   /api/v1/auth/verify-otp                  Verify OTP and reset password
POST   /api/v1/auth/logout                      Invalidate session (user_session)
```

#### 9.2.2 User Profile & Management (10 endpoints)
```
GET    /api/v1/users/me                          Get current user profile
PUT    /api/v1/users/me                          Update current user profile
PUT    /api/v1/users/me/password                 Change password
GET    /api/v1/users/me/sessions                 List active sessions (user_session)
DELETE /api/v1/users/me/sessions/{id}            Revoke a session
GET    /api/v1/users/{id}                        Get user by ID (admin)
PUT    /api/v1/users/{id}/status                 Activate/deactivate user (admin)
GET    /api/v1/users/{id}/addresses              Get user addresses (user_address)
POST   /api/v1/users/{id}/addresses              Add user address
GET    /api/v1/users/{id}/documents              Get user documents (user_document)
```

#### 9.2.3 Roles & Permissions (6 endpoints)
```
GET    /api/v1/roles                             List all roles (general_role)
GET    /api/v1/roles/{id}/permissions            Get permissions for a role
POST   /api/v1/users/{id}/roles                  Assign role to user (user_role)
DELETE /api/v1/users/{id}/roles/{roleId}         Remove role from user
GET    /api/v1/permissions                       List all permissions (general_permission)
PUT    /api/v1/roles/{id}/permissions            Update role permissions (role_permission)
```

#### 9.2.4 Organization Management (14 endpoints)
```
GET    /api/v1/organizations                     List organizations (filterable by org_type)
GET    /api/v1/organizations/{id}                Get organization detail
POST   /api/v1/organizations                     Create organization (general_organization_profile)
PUT    /api/v1/organizations/{id}                Update organization
PUT    /api/v1/organizations/{id}/verify         Verify organization (admin)
GET    /api/v1/organizations/{id}/addresses      List org addresses (org_address)
POST   /api/v1/organizations/{id}/addresses      Add org address
PUT    /api/v1/organizations/{id}/addresses/{aid} Update org address
GET    /api/v1/organizations/{id}/contacts       List contact persons (org_contact_person)
POST   /api/v1/organizations/{id}/contacts       Add contact person
GET    /api/v1/organizations/{id}/bank-accounts  List bank accounts (org_bank_account)
POST   /api/v1/organizations/{id}/bank-accounts  Add bank account
GET    /api/v1/organizations/{id}/documents      List org documents (org_document)
POST   /api/v1/organizations/{id}/documents      Upload org document
```

#### 9.2.5 Distributor-Shop Relationships (5 endpoints)
```
GET    /api/v1/distributors/{id}/shops           List shops for distributor (distributor_shop)
POST   /api/v1/distributors/{id}/shops           Link shop to distributor
PUT    /api/v1/distributors/{id}/shops/{shopId}  Update relationship (credit, terms, price tier)
GET    /api/v1/distributors/{id}/staff           List staff (org_user_assignment)
POST   /api/v1/distributors/{id}/staff           Assign user to distributor
```

#### 9.2.6 Product & Catalog (12 endpoints)
```
GET    /api/v1/categories                        List product categories (product_category)
POST   /api/v1/categories                        Create category
PUT    /api/v1/categories/{id}                   Update category
GET    /api/v1/products                          List products (filterable by vendor, category, status)
GET    /api/v1/products/{id}                     Get product detail with images & price tiers
POST   /api/v1/products                          Create product (vendor)
PUT    /api/v1/products/{id}                     Update product (vendor)
POST   /api/v1/products/{id}/images              Upload product image (product_image)
DELETE /api/v1/products/{id}/images/{imgId}      Delete product image
GET    /api/v1/products/{id}/price-tiers         Get price tiers (product_price_tier)
POST   /api/v1/products/{id}/price-tiers         Create price tier
GET    /api/v1/distributors/{id}/catalog         Get distributor's subscribed products (distributor_product)
POST   /api/v1/distributors/{id}/catalog         Subscribe to vendor product with selling price
PUT    /api/v1/distributors/{id}/catalog/{pid}   Update distributor selling price
```

#### 9.2.7 Inventory & Warehouse (12 endpoints)
```
GET    /api/v1/warehouses                        List warehouses for distributor
POST   /api/v1/warehouses                        Create warehouse
PUT    /api/v1/warehouses/{id}                   Update warehouse
GET    /api/v1/inventory                         List inventory (filterable by warehouse, product)
GET    /api/v1/inventory/{id}                    Get inventory detail with batches
GET    /api/v1/inventory/low-stock               Get items below reorder level
GET    /api/v1/inventory/movements               List inventory movements (inventory_movement)
POST   /api/v1/inventory/stock-adjustment        Create stock adjustment
GET    /api/v1/grn                               List GRNs (goods_received_note)
POST   /api/v1/grn                               Create GRN with line items
GET    /api/v1/grn/{id}                          Get GRN detail
PUT    /api/v1/grn/{id}/verify                   Verify GRN and update inventory
```

#### 9.2.8 Order Management (10 endpoints)
```
GET    /api/v1/orders                            List orders (filterable by status, shop, date, rep)
GET    /api/v1/orders/{id}                       Get order detail with items & status history
POST   /api/v1/orders                            Create order (sales_order + sales_order_item)
PUT    /api/v1/orders/{id}                       Update order (draft only)
POST   /api/v1/orders/{id}/submit                Submit order (DRAFT → SUBMITTED)
POST   /api/v1/orders/{id}/approve               Approve order, reserve inventory (SUBMITTED → APPROVED)
POST   /api/v1/orders/{id}/reject                Reject order with reason
POST   /api/v1/orders/{id}/cancel                Cancel order with reason
GET    /api/v1/orders/{id}/history               Get order status history (order_status_history)
GET    /api/v1/orders/{id}/vouchers              Get related vouchers (DN, Invoice, CN)
```

#### 9.2.9 Vehicle & Delivery Trip (12 endpoints)
```
GET    /api/v1/vehicles                          List vehicles for distributor
POST   /api/v1/vehicles                          Register vehicle
PUT    /api/v1/vehicles/{id}                     Update vehicle
GET    /api/v1/delivery-trips                    List delivery trips
POST   /api/v1/delivery-trips                    Create delivery trip
GET    /api/v1/delivery-trips/{id}               Get trip detail with orders & vehicle inventory
POST   /api/v1/delivery-trips/{id}/orders        Assign orders to trip (delivery_trip_order)
DELETE /api/v1/delivery-trips/{id}/orders/{oid}  Remove order from trip
POST   /api/v1/delivery-trips/{id}/confirm-loading  Confirm loading, generate DN vouchers
PATCH  /api/v1/delivery-trips/{id}/status        Update trip status (depart, complete, cancel)
POST   /api/v1/delivery-trips/{id}/deliver/{oid} Confirm delivery for an order (delivery_confirmation)
GET    /api/v1/delivery-trips/{id}/reconciliation  Get loaded vs delivered vs returned summary
```

#### 9.2.10 Voucher Module (10 endpoints)
```
GET    /api/v1/vouchers                          List vouchers (filterable by type, status, org, date)
GET    /api/v1/vouchers/{id}                     Get voucher detail with lines, tax summary, attachments
POST   /api/v1/vouchers                          Create voucher (manual — e.g., credit note)
PUT    /api/v1/vouchers/{id}                     Update voucher (draft only)
POST   /api/v1/vouchers/{id}/approve             Approve voucher
POST   /api/v1/vouchers/{id}/send                Send voucher via email/WhatsApp/SMS
GET    /api/v1/vouchers/{id}/pdf                 Generate/download PDF (JasperReports)
POST   /api/v1/vouchers/{id}/attachments         Upload attachment (voucher_attachment)
GET    /api/v1/vouchers/overdue                  List overdue invoices
GET    /api/v1/voucher-types                     List voucher types (voucher_type)
```

#### 9.2.11 Payment (8 endpoints)
```
GET    /api/v1/payments                          List payments (filterable by shop, method, status, date)
POST   /api/v1/payments                          Record payment (creates payment + receipt voucher)
GET    /api/v1/payments/{id}                     Get payment detail
GET    /api/v1/payments/{id}/receipt              Generate payment receipt PDF
POST   /api/v1/payments/{id}/verify              Verify payment (COLLECTED → VERIFIED)
POST   /api/v1/payments/{id}/deposit             Mark as deposited
POST   /api/v1/payments/{id}/bounce              Mark cheque as bounced
POST   /api/v1/payments/{id}/allocate            Allocate payment to invoices (payment_allocation)
```

#### 9.2.12 Shop Ledger & Collections (4 endpoints)
```
GET    /api/v1/shops/{id}/ledger                 Get shop ledger (shop_ledger)
GET    /api/v1/shops/{id}/balance                Get current outstanding balance
GET    /api/v1/daily-collections                 List daily collection summaries
POST   /api/v1/daily-collections                 Submit daily collection summary
```

#### 9.2.13 Route Planning & Field Ops (8 endpoints)
```
GET    /api/v1/routes                            List route plans (route_plan)
POST   /api/v1/routes                            Create route plan with stops
GET    /api/v1/routes/{id}                       Get route plan detail with stops
GET    /api/v1/routes/today                      Get today's route for current sales rep
POST   /api/v1/routes/{id}/stops/{stopId}/checkin   Check in at shop (route_plan_stop)
POST   /api/v1/routes/{id}/stops/{stopId}/checkout  Check out from shop
POST   /api/v1/routes/{id}/stops/{stopId}/photos    Upload shop visit photo (shop_visit_photo)
POST   /api/v1/gps-tracking                      Submit GPS tracking batch (gps_tracking_log)
```

#### 9.2.14 Reports (8 endpoints)
```
GET    /api/v1/reports/sales-summary             Sales summary (period, product, shop, rep)
GET    /api/v1/reports/sales-rep-performance     Sales rep performance metrics
GET    /api/v1/reports/product-performance       Product performance analytics
GET    /api/v1/reports/inventory                 Inventory status & movement report
GET    /api/v1/reports/delivery                  Delivery performance report
GET    /api/v1/reports/financial                 Financial summary (revenue, receivables, aging)
GET    /api/v1/reports/shop/{id}                 Shop-specific report (orders, payments, ledger)
GET    /api/v1/reports/export/{reportId}         Download report as PDF/Excel (JasperReports)
```

#### 9.2.15 Sync — Mobile Offline (3 endpoints)
```
POST   /api/v1/sync/push                        Upload offline data (orders, payments, GPS, visits)
GET    /api/v1/sync/pull                         Download updates since last sync timestamp
GET    /api/v1/sync/status                       Check sync status & pending conflicts
```

#### 9.2.16 Notifications (4 endpoints)
```
GET    /api/v1/notifications                     List notifications for current user
PATCH  /api/v1/notifications/{id}/read           Mark notification as read
POST   /api/v1/notifications/read-all            Mark all as read
GET    /api/v1/notifications/preferences         Get/update notification preferences
PUT    /api/v1/notifications/preferences         Update notification preferences
```

### 9.3 Endpoint Count Summary

| Group | Count |
|-------|-------|
| Authentication & Session | 6 |
| User Profile & Management | 10 |
| Roles & Permissions | 6 |
| Organization Management | 14 |
| Distributor-Shop Relationships | 5 |
| Product & Catalog | 14 |
| Inventory & Warehouse | 12 |
| Order Management | 10 |
| Vehicle & Delivery Trip | 12 |
| Voucher Module | 10 |
| Payment | 8 |
| Shop Ledger & Collections | 4 |
| Route Planning & Field Ops | 8 |
| Reports | 8 |
| Sync (Mobile) | 3 |
| Notifications | 5 |
| **Total** | **135 endpoints** |

---

## 10. Appendices

### Appendix A: Business Process Flows

#### A.1 Order-to-Delivery Flow (with Table References)

```
Sales Rep checks in at shop (route_plan_stop)
        │
        ▼
Browses distributor_product catalog, checks inventory.quantity_available
        │
        ▼
Collects order → creates sales_order + sales_order_item (offline capable, idempotency_key)
        │
        ▼
Submits order → status: DRAFT → SUBMITTED (order_status_history logged)
        │
        ▼
Distributor reviews in AdminApp
        │
    ┌───┴───┐
    ▼       ▼
 Approve  Reject ──▶ Notify Sales Rep & Shop (notification)
    │                  order_status_history: SUBMITTED → CANCELLED
    ▼
System reserves inventory (inventory.quantity_reserved += qty)
order_status_history: SUBMITTED → APPROVED
        │
        ▼
Distributor creates delivery_trip, assigns order (delivery_trip_order)
        │
        ▼
Warehouse loads goods → vehicle_inventory created
inventory_movement: LOADING, quantity_on_hand decreases
        │
        ▼
System generates Delivery Note voucher (voucher type=DN, voucher_line items)
Loading Note voucher (voucher type=DN_LOAD)
delivery_trip status: LOADING → LOADED
        │
        ▼
Vehicle departs → delivery_trip status: LOADED → IN_TRANSIT
        │
        ▼
Delivery Personnel arrives at shop
delivery_confirmation created (signature, photo, GPS)
        │
    ┌───┴───┐
    ▼       ▼
 Full    Partial ──▶ delivery_return records created
 Delivery           vehicle_inventory.quantity_returned updated
    │                inventory_movement: RETURN
    ▼
voucher (DN) status updated → DELIVERED or PARTIAL
vehicle_inventory.quantity_delivered updated
sales_order status: IN_TRANSIT → DELIVERED
        │
        ▼
System auto-generates Invoice voucher (voucher type=INV)
voucher_line items from delivered quantities
voucher_tax_summary calculated
shop_ledger: DEBIT entry (running_balance increases)
        │
        ▼
Payment collected → payment record created
Receipt voucher (voucher type=REC) generated
payment_allocation links payment to invoice
shop_ledger: CREDIT entry (running_balance decreases)
voucher (INV) amount_paid updated, status → PARTIALLY_PAID or PAID
        │
        ▼
daily_collection_summary submitted by field staff
Distributor verifies & reconciles
        │
        ▼
sales_order status: DELIVERED → COMPLETED
```

#### A.2 Inventory Flow (with Table References)

```
Vendor ships goods to Distributor
        │
        ▼
Distributor creates goods_received_note + goods_received_note_item
        │
        ▼
GRN verified → inventory.quantity_on_hand increases
inventory_movement: STOCK_IN
inventory_batch created (batch_number, expiry_date)
        │
        ▼
Order approved → inventory.quantity_reserved increases
inventory_movement: (reserved, no physical movement)
        │
        ▼
Goods loaded onto vehicle → inventory.quantity_on_hand decreases
vehicle_inventory.quantity_loaded set
inventory_movement: LOADING
        │
        ▼
Goods delivered → vehicle_inventory.quantity_delivered increases
        │
        ▼
Returns (if any) → delivery_return created
vehicle_inventory.quantity_returned increases
inventory.quantity_on_hand increases (returned to warehouse)
inventory_movement: RETURN
        │
        ▼
Manual adjustments → inventory_movement: ADJUSTMENT
reason_code: DAMAGE, EXPIRY, THEFT, CORRECTION
```

#### A.3 Payment & Ledger Flow (with Table References)

```
Invoice voucher generated (after delivery)
shop_ledger: DEBIT entry posted
        │
        ▼
Payment due (immediate or per payment_terms_days)
        │
        ├──▶ Cash collected by Delivery/Sales Rep
        │         │
        │         ▼
        │    payment created (method=CASH, status=COLLECTED)
        │    Receipt voucher (type=REC) generated
        │    payment_allocation → links to invoice
        │    shop_ledger: CREDIT entry posted
        │    voucher (INV) balance_due recalculated
        │         │
        │         ▼
        │    daily_collection_summary submitted
        │    Distributor verifies → payment status: COLLECTED → VERIFIED → DEPOSITED
        │
        ├──▶ Cheque collected
        │         │
        │         ▼
        │    payment created (method=CHEQUE, cheque_number, cheque_date, cheque_bank)
        │    payment status: COLLECTED → VERIFIED → DEPOSITED
        │         │
        │         ▼
        │    Clearance: status → CLEARED
        │    OR Bounce: status → BOUNCED
        │       → shop_ledger: DEBIT reversal entry
        │       → voucher (INV) balance_due restored
        │
        ├──▶ Bank Transfer / UPI / Mobile Payment
        │         │
        │         ▼
        │    payment created (method=BANK_TRANSFER/UPI, reference_number)
        │    Auto-verified → status: COLLECTED → CLEARED
        │
        └──▶ Credit (on account)
                  │
                  ▼
             shop_ledger running_balance tracked
             System checks against credit_limit (from distributor_shop)
                  │
                  ▼
             If overdue → voucher (INV) status: SENT → OVERDUE
             notification sent to shop & distributor
             New orders blocked if exceeds credit_limit (configurable)
```

#### A.4 Voucher Lifecycle Flow

```
Trigger Event (delivery, payment, return, loading)
        │
        ▼
voucher_number_sequence: next number generated
Format: {PREFIX}{YEAR}-{SEQ:5} (e.g., INV-2026-00001)
        │
        ▼
voucher record created (status=DRAFT or GENERATED)
voucher_line items populated
voucher_tax_summary calculated
        │
        ▼
PDF generated via JasperReports (jasper_template from voucher_type)
Stored at pdf_url
        │
        ▼
Sent to counterparty:
├── Email (Jakarta Mail 2.0.1) → sent_via=EMAIL
├── WhatsApp (Meta Cloud API) → sent_via=WHATSAPP
├── SMS → sent_via=SMS
├── Print → sent_via=PRINT
└── In-App → sent_via=APP
voucher status: GENERATED → SENT
        │
        ▼
For Invoices (type=INV):
├── Payments received → status: SENT → PARTIALLY_PAID → PAID
├── Past due_date → status: → OVERDUE
└── Credit Note issued → parent_voucher_id links to original
    shop_ledger: CREDIT entry reduces balance
```

### Appendix B: Glossary of Status Codes

#### Sales Order Statuses (`sales_order.status`)
| Status | Description | Triggered By |
|--------|-------------|-------------|
| `DRAFT` | Order created but not yet submitted | Sales Rep (mobile) |
| `SUBMITTED` | Order submitted, awaiting distributor review | Sales Rep |
| `APPROVED` | Order approved, inventory reserved | Distributor (AdminApp) |
| `PROCESSING` | Order being prepared for delivery | System |
| `LOADED` | Order loaded onto delivery vehicle | Warehouse staff |
| `IN_TRANSIT` | Vehicle is en route to the shop | Delivery personnel |
| `DELIVERED` | Goods delivered and acknowledged by shop | Delivery personnel |
| `COMPLETED` | Delivery confirmed, invoice generated, payment settled | System |
| `CANCELLED` | Order cancelled (before delivery) | Distributor / Shop |
| `RETURNED` | Goods fully returned by shop (after delivery) | Delivery personnel |

#### Voucher Statuses (`voucher.status`)
| Status | Description | Applicable Types |
|--------|-------------|-----------------|
| `DRAFT` | Voucher created, not yet finalized | All |
| `GENERATED` | Voucher finalized, PDF generated | All |
| `SENT` | Sent to counterparty | INV, DN, CN |
| `PARTIALLY_PAID` | Partial payment received | INV |
| `PAID` | Full payment received | INV |
| `OVERDUE` | Past due date, unpaid | INV |
| `CANCELLED` | Voided | All |
| `VOID` | Superseded by correction | All |

#### Payment Statuses (`payment.status`)
| Status | Description |
|--------|-------------|
| `COLLECTED` | Payment collected by field staff, pending verification |
| `VERIFIED` | Payment verified by distributor office |
| `DEPOSITED` | Cash/cheque deposited to bank |
| `CLEARED` | Bank transfer/cheque cleared |
| `BOUNCED` | Cheque bounced — balance restored |
| `CANCELLED` | Payment entry cancelled/reversed |

#### Delivery Trip Statuses (`delivery_trip.status`)
| Status | Description |
|--------|-------------|
| `PLANNED` | Trip created, orders assigned |
| `LOADING` | Warehouse loading in progress |
| `LOADED` | All items loaded, DN vouchers generated |
| `IN_TRANSIT` | Vehicle departed warehouse |
| `COMPLETED` | All deliveries done, trip reconciled |
| `CANCELLED` | Trip cancelled before departure |

### Appendix C: Notification Templates

| Event | Channel | Template |
|-------|---------|----------|
| New Order | Push | "New order #{order_number} received from {shop_name} — {currency}{amount}" |
| Order Approved | Push + SMS | "Your order #{order_number} has been approved. Expected delivery: {date}" |
| Order Rejected | Push + SMS | "Your order #{order_number} was not approved. Reason: {reason}" |
| Out for Delivery | Push + WhatsApp | "Your order #{order_number} is out for delivery. Track: {link}" |
| Delivery Completed | Push + Email | "Order #{order_number} delivered. Invoice #{invoice_number} attached." |
| Payment Received | Push | "Payment of {currency}{amount} received for Invoice #{invoice_number}" |
| Payment Overdue | Push + SMS + WhatsApp | "Payment of {currency}{amount} for Invoice #{invoice_number} is overdue by {days} days" |
| Cheque Bounced | Push + SMS | "Cheque #{cheque_number} for {currency}{amount} has bounced. Please arrange alternative payment." |
| Low Stock Alert | Push + Email | "Low stock alert: {product_name} — only {qty} {uom} remaining in {warehouse_name}" |
| Daily Route Assigned | Push | "Your route for {date} is ready. {count} shops to visit." |
| Sync Conflict | Push | "Sync conflict detected for order #{order_number}. Please review in app." |
| Credit Limit Exceeded | Push + Email | "Shop {shop_name} has exceeded credit limit. Outstanding: {currency}{balance} / Limit: {currency}{limit}" |

### Appendix D: Project Milestones (6 Phases — ~8.5 Months)

| Phase | Milestone | Duration | Backend Deliverables | Frontend Deliverables |
|-------|-----------|----------|---------------------|----------------------|
| **Phase 1** | Foundation | 6 weeks | JPA entities (Groups A, B, J), Auth EJBs (JWT, BCrypt, OTP), User/Org/Role JAX-RS endpoints, MariaDB schema setup, Docker Compose config | AdminApp: Login, User Management, Org Management screens; React project setup (Vite + Tailwind + Zustand) |
| **Phase 2** | Core Operations | 8 weeks | Product/Catalog EJBs & endpoints (Group C), Order Management EJBs & endpoints (Group E), Inventory EJBs & endpoints (Group D), Redis cache integration | AdminApp: Product Catalog, Order Management, Inventory screens; Mobile App: Sales Rep — Home, Shop Visit, Order Creation, Order History (React Native + offline SQLite) |
| **Phase 3** | Delivery & Logistics | 6 weeks | Vehicle/Delivery EJBs & endpoints (Group F), Route Planning EJBs & endpoints (Group I), GPS tracking endpoint, Voucher module — DN & Loading Note (Group G partial) | AdminApp: Vehicle Management, Trip Planner, Route Planning, Loading Sheet; Mobile App: Delivery Personnel — Trip, Loading, Delivery Confirmation screens |
| **Phase 4** | Financial | 6 weeks | Voucher module — Invoice, Receipt, Credit Note (Group G complete), Payment EJBs & endpoints (Group H), Shop Ledger logic, JasperReports templates (Invoice, DN, Receipt, Loading Note, Statement) | FinanceApp: Full setup — Dashboard, Voucher Management, Payment Management, Shop Ledger, Financial Reports; CustomerPortal: Shop Owner & Vendor views |
| **Phase 5** | Analytics & Polish | 4 weeks | Report EJBs (JasperReports integration), Notification EJBs (FCM, Jakarta Mail, WhatsApp API), Sync engine (push/pull/conflict resolution), Audit logging | AdminApp: Reports & Dashboards (Recharts); Mobile App: Offline sync engine, Daily Summary, Payment Collection, Notifications; All apps: Dark mode, multi-language |
| **Phase 6** | Testing & Launch | 4 weeks | JUnit 5 + Mockito test suites (80% coverage), Performance testing (JMeter), Security audit (OWASP), Production deployment (Docker + Nginx) | UAT with field users, Bug fixes, App Store / Play Store submission, User training materials |
| **Total** | | **34 weeks (~8.5 months)** | | |

### Appendix E: Risk Assessment Matrix

| # | Risk | Probability | Impact | Severity | Mitigation Strategy |
|---|------|------------|--------|----------|-------------------|
| R1 | Poor network connectivity in field areas | High | High | **Critical** | Offline-first architecture with SQLite local DB, sync engine with idempotency keys, conflict resolution via server-side validation |
| R2 | User adoption resistance (Sales Reps, Delivery) | Medium | High | **High** | Intuitive mobile UI (< 5 taps for order), training programs, phased rollout starting with tech-savvy reps, feedback loops |
| R3 | Data loss during offline sync conflicts | Medium | High | **High** | Timestamp-based conflict resolution, idempotency_key dedup, server-side inventory re-validation, sync_log for audit, manual conflict resolution UI |
| R4 | Payment fraud or misappropriation | Low | High | **High** | GPS-tagged payments, daily_collection_summary with variance tracking, dual verification workflow, audit_log on all financial operations |
| R5 | WildFly monolith performance bottleneck | Medium | Medium | **Medium** | Redis caching (Jedis) for hot data, MariaDB read replicas for reports, JPA query optimization, connection pooling, Docker Compose scaling |
| R6 | Scope creep during development | High | Medium | **High** | Phased delivery with clear milestones, strict change management, feature flags via system_configuration, MVP-first approach |
| R7 | JasperReports PDF generation latency | Medium | Low | **Medium** | Async PDF generation via EJB @Asynchronous, pre-compiled .jasper templates, Redis caching of generated PDFs |
| R8 | Mobile app compatibility issues | Medium | Medium | **Medium** | React Native cross-platform, minimum OS versions (Android 8.0+, iOS 14.0+), device testing matrix, OTA updates |
| R9 | MariaDB data growth / query performance | Low | Medium | **Medium** | Table partitioning for gps_tracking_log and audit_log, archival strategy for old data, proper indexing, query profiling |
| R10 | WhatsApp API rate limits / downtime | Medium | Low | **Low** | Fallback to SMS/Email, message queue for retry, Meta WhatsApp Cloud API rate limit monitoring |
| R11 | Security breach / unauthorized access | Low | High | **High** | JWT with short expiry + refresh tokens, BCrypt password hashing, RBAC with granular permissions, TLS 1.3, audit_log, IP whitelisting for admin |
| R12 | Key personnel dependency | Medium | Medium | **Medium** | Documentation, code reviews, knowledge sharing sessions, modular codebase with clear separation |

### Appendix F: Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PRODUCTION SERVER                         │
│                    (e.g., 109.123.227.166)                  │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Host Nginx (port 443 — SSL Termination)               │ │
│  │  ├── /admin/*     → localhost:3000 (AdminApp)          │ │
│  │  ├── /finance/*   → localhost:3001 (FinanceApp)        │ │
│  │  ├── /portal/*    → localhost:3002 (CustomerPortal)    │ │
│  │  ├── /api/*       → localhost:8080 (WildFly)           │ │
│  │  └── /mobile-api/*→ localhost:8080 (WildFly)           │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  ┌──── Docker Compose ────────────────────────────────────┐ │
│  │                                                         │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  │ │
│  │  │ WildFly 31  │  │ MariaDB 10.6│  │ Redis 5.0.2   │  │ │
│  │  │ + JDK 17    │  │             │  │ (Jedis)       │  │ │
│  │  │ :8080       │  │ :3306       │  │ :6379         │  │ │
│  │  │ (WAR deploy)│  │ (mvdsms_    │  │               │  │ │
│  │  │             │  │  system)    │  │               │  │ │
│  │  └─────────────┘  └─────────────┘  └───────────────┘  │ │
│  │                                                         │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────┐  │ │
│  │  │ AdminApp    │  │ FinanceApp  │  │CustomerPortal │  │ │
│  │  │ (React)     │  │ (React)     │  │ (React)       │  │ │
│  │  │ :3000       │  │ :3001       │  │ :3002         │  │ │
│  │  └─────────────┘  └─────────────┘  └───────────────┘  │ │
│  │                                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Appendix G: Maven Project Structure (Backend)

```
mvdsms/
├── pom.xml                          (Parent POM — Java 17, Jakarta EE 10, WildFly BOM)
├── src/
│   ├── main/
│   │   ├── java/com/teccoop/mvdsms/
│   │   │   ├── config/              (Application config, CORS, JWT filter)
│   │   │   ├── entity/              (JPA @Entity classes — maps to all 57 tables)
│   │   │   │   ├── user/            (GeneralUserProfile, GeneralRole, UserRole, etc.)
│   │   │   │   ├── organization/    (GeneralOrganizationProfile, OrgAddress, etc.)
│   │   │   │   ├── product/         (Product, ProductCategory, ProductImage, etc.)
│   │   │   │   ├── inventory/       (Inventory, InventoryBatch, InventoryMovement, etc.)
│   │   │   │   ├── order/           (SalesOrder, SalesOrderItem, OrderStatusHistory)
│   │   │   │   ├── delivery/        (Vehicle, DeliveryTrip, DeliveryConfirmation, etc.)
│   │   │   │   ├── voucher/         (Voucher, VoucherLine, VoucherType, etc.)
│   │   │   │   ├── payment/         (Payment, PaymentAllocation, ShopLedger, etc.)
│   │   │   │   ├── route/           (RoutePlan, RoutePlanStop, GpsTrackingLog, etc.)
│   │   │   │   └── system/          (TaxRate, SystemConfiguration, Notification, etc.)
│   │   │   ├── repository/          (JPA @Repository / DAO classes)
│   │   │   ├── service/             (EJB @Stateless business logic)
│   │   │   ├── rest/                (JAX-RS @Path resource classes)
│   │   │   │   ├── AuthResource.java
│   │   │   │   ├── UserResource.java
│   │   │   │   ├── OrganizationResource.java
│   │   │   │   ├── ProductResource.java
│   │   │   │   ├── OrderResource.java
│   │   │   │   ├── InventoryResource.java
│   │   │   │   ├── DeliveryResource.java
│   │   │   │   ├── VoucherResource.java
│   │   │   │   ├── PaymentResource.java
│   │   │   │   ├── RouteResource.java
│   │   │   │   ├── ReportResource.java
│   │   │   │   ├── SyncResource.java
│   │   │   │   └── NotificationResource.java
│   │   │   ├── dto/                 (Request/Response DTOs)
│   │   │   ├── mapper/              (Entity ↔ DTO mappers)
│   │   │   ├── security/            (JWT provider, auth filter, RBAC interceptor)
│   │   │   ├── exception/           (Custom exceptions, JAX-RS ExceptionMapper)
│   │   │   └── util/                (Helpers, constants, validators)
│   │   ├── resources/
│   │   │   ├── META-INF/persistence.xml   (JPA/Hibernate config for MariaDB)
│   │   │   └── reports/                    (JasperReports .jrxml templates)
│   │   └── webapp/
│   │       └── WEB-INF/web.xml
│   └── test/
│       └── java/com/teccoop/mvdsms/        (JUnit 5 + Mockito tests)
├── Dockerfile                       (WildFly 31 + WAR deployment)
├── docker-compose.yml               (WildFly + MariaDB + Redis + 3 React apps)
└── README.md
```

---

## Document Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Project Manager | | | |
| Technical Lead | | | |
| Business Analyst | | | |
| Client Representative | | | |

---

*End of Document — MVDSMS SRS v2.0*
