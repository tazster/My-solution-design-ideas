# Solution Design Blueprints: Event-Driven Feedback & Batch Pipeline

## Project Overview
This repository contains a conceptual solution design and reference architecture for a decoupled, event-driven customer feedback system integrated with asynchronous batch reconciliation layers. 

The goal of this design is to handle high-volume user interactions across multiple support channels, capture synchronous customer sentiment metadata into an ingestion buffer, and process data asynchronously to protect core transactional databases while syncing with external platforms.

---

## System Architecture Model (ArchiMate 3.1)

Below is the conceptual system topography mapped using the ArchiMate enterprise architecture framework. It isolates operations, real-time feedback flows, long-term data persistence, and scheduled integration loops.

![System Architecture](./docs/architecture_diagram.png)

### Key Architectural Layers

### 1. Operational Layer
* **Customer Interaction:** Captures bi-directional engagement across various communication interfaces (Email, Phone, Chat).
* **Case Management:** Handles operational ticketing workflows, updating state synchronously to an isolated Operational Data Store (ODS).

### 2. Feedback Ingestion & Integration
* **Asynchronous Event Trigger:** Closing an active operational ticket fires an immediate `Survey Triggered` lifecycle event.
* **Ingestion Buffer:** Survey payloads are written directly to a staging table layer rather than production analytical databases to maximize throughput and minimize database locking.
* **Outbound Gateway:** An automated publication worker drains the staging buffer and streams transactional payloads out via an external-facing API.

### 3. External Platform & Reconciliation Layer
* **Reconciliation Loop:** A Python-based ingestion worker executes a decoupled "pull" pattern to fetch processing status from the external 3rd-Party Platform (3PP).
* **Consumer Queue:** Events pass through a consumer message queue to manage throttling and write final states cleanly into the long-term Analytics Data Store.

### 4. Scheduled Multi-Tenant Batch Processing
To support multi-tenant SaaS data isolation guidelines, a dedicated Batch Orchestrator handles heavy analytical syncs at specified system intervals:
* **Enrichment Sync (12-Hour Interval):** Hydrates customer interaction data with deeper analytical context.
* **Metadata Syncs (Batch Intervals):** Dispatches specialized User and Tenant (Organization) profile documents via parallel dedicated APIs to ensure clean account segregation.

---

