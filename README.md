# ServiceNow MCP-Gateway Public

*(Reference Architecture & Personal Technical Exploration)*

This repository documents a **personal, non-commercial technical exploration** of governance patterns for AI systems using the **Model Context Protocol (MCP)** when interacting with enterprise platforms such as **ServiceNow**.

The focus of this work is architectural:
how to introduce **policy enforcement, auditability, and execution control** between AI intent and downstream system actions.


<img width="auto" height="500" alt="image" src="https://github.com/user-attachments/assets/d9718130-7c7c-4d10-a392-9e7cd75a1cf7" />

> **Important**
> This repository intentionally contains **documentation and reference materials only**.
> It does **not** provide a public implementation, product, or deployable solution.

---
![SCR-20251230-mwnr](https://github.com/user-attachments/assets/112dd858-e59f-4f80-ab7c-5d371f1b3ea6)



## What This Is

* A **reference architecture** for MCP-based AI integrations
* A **learning exercise** focused on AI governance and control-plane design
* A set of **patterns and diagrams** illustrating how AI intent can be safely translated into system execution

---

## What This Is Not

* A product, platform, or SaaS offering
* An official ServiceNow integration
* A replacement for the ServiceNow SDK, APIs, or tooling
* A production-ready or supported deployment

---

## Architectural Overview

At a high level, the exploration applies a familiar **Control Plane / Data Plane** mental model:

* **Control Plane**
  Defines what actions are allowed, under which conditions, and with what constraints.

* **Data Plane**
  Handles the execution of approved actions and emits audit-ready signals.

This separation helps make AI-initiated actions:

* explicit
* reviewable
* bounded

without embedding governance logic directly into every integration.

---

![SCR-20251230-mwjy](https://github.com/user-attachments/assets/d96da915-a0ca-462d-86ec-62cd03cb39cd)



## Conceptual Request Flow

1. An AI system expresses **intent** using MCP
2. The request is evaluated against **declared capabilities and policies**
3. Approved actions are executed via standard ServiceNow APIs or SDK-based endpoints
4. Execution context and outcomes are recorded for traceability

All system names and flows are shown **as examples** only.

---

## Visual References

This repository includes architecture diagrams and walkthroughs that illustrate these patterns:

* **Conceptual Request Lifecycle** — from AI intent to ServiceNow execution
* **Control Plane vs Data Plane Separation** — governance vs execution
* **Point-to-Point vs Gateway Pattern** — centralized policy enforcement

All visuals are labeled as **reference architecture** and are provided for learning purposes.

---
<img width="auto" height="500" alt="image" src="https://github.com/user-attachments/assets/1af46a9b-54d0-4d71-9eb7-13d858488228" />

## Status

This work is ongoing and intentionally scoped as a **personal technical exploration**.
There is no public roadmap, release plan, or commercialization intent.

---
<img width="1508" height="746" alt="image" src="https://github.com/user-attachments/assets/32e4781f-befc-4dcf-bf7e-cfd02f42a396" />
<img width="1470" height="646" alt="image" src="https://github.com/user-attachments/assets/2c632b9a-ff6c-454c-a082-d67b7ea12454" />
<img width="1255" height="2821" alt="image" src="https://github.com/user-attachments/assets/b5aeaf69-e01a-4c62-ad5a-42e936a2d98b" />


# 🔄 MCP Gateway Flow Cycle

This document outlines the complete request-response lifecycle within the MCP Gateway architecture, detailing how user requests are processed, secured, and executed against ServiceNow.


## ⚡ The 8-Step Flow

The lifecycle of a single request travels through four distinct zones: **User**, **Client**, **Gateway**, and **ServiceNow**.

### 1. User Makes Request
> **Actor:** User
- The user interacts with their AI assistant (Claude, Gemini, etc.) using natural language.
- **Example:** *"Create a P1 incident for the database outage."*

### 2. Request to MCP Client
> **Actor:** AI Agent (MCP Client)
- The AI agent receives the natural language prompt.
- It identifies that external tools are needed to fulfill the request.
- It prepares to communicate with the configured MCP Gateway.

### 3. Client Requests Tools
> **Actor:** AI Agent → Gateway
- The client connects to the Gateway (via stdio or HTTP) and requests the list of available capabilities (tools).
- **Protocol:** `tools/list`

### 4. Gateway Returns Tools
> **Actor:** Gateway → AI Agent
- The **Policy Engine** intercepts the request.
- It evaluates the user's identity and permissions.
- It filters the global capability catalog, returning *only* the tools this specific user is allowed to see.

### 5. Tool Invocation
> **Actor:** AI Agent → Gateway
- The AI Agent selects the appropriate tool (e.g., `create_incident`) and extracts parameters from the user's prompt.
- It sends a JSON-RPC request to the Gateway.
- **Protocol:** `tools/call`

### 6. Gateway → ServiceNow
> **Actor:** Gateway → ServiceNow API
- **Pre-Execution Check:** The Gateway validates the action against the active policy (checking for read-only modes, required approvals, etc.).
- The **ServiceNow Connector** translates the request into a specific ServiceNow API call (REST or SDK).
- It executes the authenticated request against the ServiceNow instance.

### 7. ServiceNow Response
> **Actor:** ServiceNow → Gateway
- ServiceNow returns the raw result (e.g., the created incident record).
- **Post-Execution:** The Gateway receives the data and applies **Field Filtering** policies to redact sensitive information before passing it back.

### 8. Result to User
> **Actor:** AI Agent → User
- The Gateway returns the sanitized JSON result to the AI Agent.
- The AI Agent interprets the structured data and generates a natural language response for the user.
- **Result:** *"I've successfully created Incident INC12345. You can view it here..."*

---

## 🔐 Security Layers Throughout

The Gateway architecture enforces security at every stage of the lifecycle, ensuring a "Defense in Depth" strategy.

| Layer | Location | Description |
| :--- | :--- | :--- |
| **Authentication** | Client ↔ Gateway | Verifies the identity of the calling AI Agent (OAuth/API Key). |
| **Authentication** | Gateway ↔ ServiceNow | Manages secure, scoped credentials for backend access. |
| **Policy Enforcement** | Capability Discovery | Hides unauthorized tools from the user entirely. |
| **Policy Enforcement** | Execution Request | Validates specific actions (e.g., "Can User A create P1 incidents?"). |
| **Audit Logging** | Decision Points | Records every ALLOW/DENY decision, input payload, and outcome for compliance. |
| **Safety Mechanisms** | Global Guardrails | Includes **Kill Switch**, **Read-Only Mode**, **Rate Limiting**, and **Approval Workflows**. |



## Disclaimer

This repository reflects a personal learning exercise and does not represent the views of any employer, client, or vendor.
ServiceNow is referenced solely as an example of a downstream enterprise system.


