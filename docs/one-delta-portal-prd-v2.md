# One Delta Portal — Product Requirements Document
**Unified Replacement for GRC-NEXT**

| Field | Value |
|-------|-------|
| Last Updated | June 5, 2026 |
| Status | In Progress |
| Phase | Commercial (Phase 0 complete) |
| Primary Audiences | UX · Engineering · GFRC · AWS SA |
| Owner | Delta Portal Team |
| Version | v2.0 — Revised from April 2026 draft |

---

## How to Use This Document
Sections 1–5 cover product context, personas, and requirements — relevant to all readers.
Section 6 contains user stories with acceptance criteria for the guided onboarding experience.
Technical specifications (data models, API contracts, migration scripts) are maintained separately in the Delta Portal Technical Dive Deep document (Owner: mbbardin@, rajeshch@).

---

## 1. Background and Purpose

One Delta Portal is a unified compliance portal that replaces GRC-NEXT. It serves two primary compliance programs that today operate in separate systems, creating duplication, inconsistent data, and an unclear user experience for app owners caught in the middle.

| | AWS Security Assurance | GFRC — Global Financial Risk & Compliance |
|--|------------------------|------------------------------------------|
| **Programs served** | DMP (Deployment Monitoring), AVM (Vulnerability Monitoring), SOC1 | SOX Financial Reporting, Financial Frameworks |
| **Current portal** | DeltaV3 | GRC-NEXT |
| **Problem** | Fragmented UX; in-memory pagination causes timeouts at scale | Siloed from Delta; onboarding is manual and multi-step across portals |

One Delta Portal replaces both portals with a single application that shares data, permissions, and violation management — while respecting the distinct compliance needs of each program.

### Core Design Principle

The portal is the **interaction layer** for compliance monitoring. Application inventory can originate from multiple authoritative systems depending on the compliance program and region:

- **GRC2 / GRCRMS** — primary source for SOX and financial frameworks in commercial regions
- **Delta itself** — source of record for China (BJS/ZHY) where GRC does not operate, and for teams that register directly in Delta without a GRC dependency
- **Other client systems** — compliance programs outside GRC that onboard to Delta monitoring independently (e.g., programs not subject to SOX)

**One Delta Portal does not replicate source-of-truth data.** Where GRC2 is the authoritative registry, Delta syncs from GRC2 via EventBridge. Where Delta is the registry (China, direct-onboard clients), Delta is the source of truth. This design keeps maintenance burden low and ensures each compliance program's data stays in its authoritative system.

---

## 2. Scope

### 2.1 In-Scope

1. Commercial regions — primary delivery target
2. Guided onboarding wizard replacing the current multi-page GRC + Delta flow
3. Unified violations view across all five monitoring services (FIM, Lambda, EC2, Carnaval, DBM)
4. Exclusion management with in-portal approval workflow
5. Quarterly baselining workflow with automated reminders
6. Self-service compliance reporting for GFRC (Phase TBD — see open item OI-10)
7. China (BJS/ZHY) and ADC partitions — Phase 6
8. Support for non-GRC compliance programs onboarding directly to Delta

### 2.2 Out of Scope

1. CAZ (Change Authorization) authorization flow — not proceeding; adds friction without sufficient value
2. Replication of all GRC2 data in Delta — portal links to GRC2 rather than duplicating records
3. DeltaV3 decommission — planned as the final phase after full migration

---

## 3. Schedule and Phases

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 0 | Commercial foundation | Complete |
| Phase 1 | Guided onboarding wizard; baselining workflow; baseline metrics collection | Planned |
| Phase 2 | Application data sync (GRC2 → Delta); AWSAccountRegistrations migration (pending EY approval) | Planned |
| Phase 3 | Violations page — unified view across all 5 services; DDB-native pagination | Planned |
| Phase 4 | Exclusion management; AppSec authorization certification | Planned |
| Phase 5 | UI build — Cloudscape (rounded) or Fintech UI Framework; information architecture finalized | Planned |
| Phase 6 | China (BJS/ZHY) and ADC partition support; cross-region violation awareness feed; non-GRC client onboarding | Planned |
| Final | DeltaV3 decommission — after full migration confirmed | TBD |

> ⚠️ Phase 2/3 may be delayed pending EY approval of AWSAccountRegistrations migration (53 downstream references). Initiate approval request immediately.
>
> ⚠️ Phase 6 scope includes Delta-native onboarding for China region apps (no GRC dependency) and direct-onboard compliance clients. Architecture decisions for multi-source inventory must be made before Phase 6 design begins (see OI-11).

---

## 4. Personas and Roles

Role is determined automatically by group membership — users do not select their role.

| Role | How Determined | What They Can Do |
|------|----------------|-----------------|
| SOX Admin | Compliance admin LDAP group | Full access to all apps; approve exclusions; manage global config; archive apps |
| App Owner | App write POSIX group | Manage own apps: resolve violations, onboard/offboard resources, request exclusions |
| App Viewer | App read POSIX group | Read-only view of assigned apps; export violations as CSV |
| Regular User | Authenticated Amazon employee | Register new applications; no access to existing apps they do not own |
| Third Line (Read-Only) | EY auditor or internal audit LDAP group | Scoped read-only access to formal audit evidence; no internal resolution context or exclusion rationale visible |

> **Note on Third Line access**: Third Line users (EY auditors, internal audit) require a dedicated role with scoped visibility. AI-generated responses and exclusion rationale must be filtered to expose only formal audit evidence — not internal resolution context. This role is not yet in the permissions matrix; it must be added before Phase 4 ships (see OI-12).

### 4.1 GFRC User Personas

Within the SOX Admin and App Owner roles, GFRC team members operate in three distinct contexts:

1. **Third Line (Compliance/Audit Oversight)** — EY auditors and internal audit; need scoped read-only access to formal audit evidence and in-scope/out-of-scope status history. AI responses must be restricted to formal audit evidence only — not internal resolution context or exclusion rationale.

2. **Second Line (WWSF, AWS Accounting)** — operational compliance; need pipeline registration status, monitoring health, and self-service reporting across their app portfolio.

3. **Controllership Contact** — GFRC representative surfaced on each app record for escalation and communications. Weekly baselining digest goes to the Controllership POC, not the broader GFRC team (confirmed).

### 4.2 Application Data — Required Fields Per App Record

App records in One Delta Portal can originate from multiple source systems. The required fields apply regardless of origin:

| Field | Required | Source | Notes |
|-------|----------|--------|-------|
| App ID | Yes | GRC2 (GRC-origin apps) or Delta-assigned (direct-onboard apps) | Primary identifier |
| Application Name | Yes | Source system or user-entered | — |
| Source System | Yes | System-assigned | GRC2, Delta-native, or other registered client system |
| Framework Membership | Yes | Source system | Which compliance frameworks the app belongs to (SOX, PCI, etc.) |
| GRC Document Significance | Conditional | GRC2 (GRC-origin only) | Key or Non-Key; N/A for non-GRC apps |
| GRC Document State | Conditional | GRC2 (GRC-origin only) | Draft or Published; N/A for non-GRC apps |
| Permissions (Ownership) | Yes | Write/read POSIX group | — |
| 3rd Party or Internal | Yes | Source system or user-entered | — |
| POC / GFRC Rep | Yes | Source system or user-entered | Primary contact for GFRC communications |
| Related Documents | No | User-entered | Links to related GRC2 or other documents |
| In Production Since | TBD | Confirm with GFRC | Required for SOX scope determination |

> **Architecture note**: For GRC-origin apps, data flows via GRC2 EventBridge sync — not manually entered. For Delta-native apps (China, direct-onboard), fields are entered during the onboarding wizard and stored in Delta. The portal must clearly surface which source system an app record originates from.

---

## 5. Permissions Matrix

✅ = permitted ❌ = not permitted ⚠️ = restricted (see note)

| Action | SOX Admin | App Owner | App Viewer | Regular User | Third Line |
|--------|-----------|-----------|------------|--------------|------------|
| Authenticate via Midway SSO | ✅ | ✅ | ✅ | ✅ | ✅ |
| View dashboard | ✅ All apps | ✅ Own apps | ✅ Assigned apps | ❌ Empty state | ✅ In-scope apps only |
| Filter and search applications | ✅ | ✅ | ✅ | ❌ | ✅ In-scope only |
| Register a new application | ✅ | ✅ | ❌ | ✅ | ❌ |
| View application detail | ✅ | ✅ | ✅ | ❌ | ✅ Audit evidence only |
| Edit application (CTI, POSIX groups) | ✅ | ✅ | ❌ | ❌ | ❌ |
| Archive application | ✅ | ✅ | ❌ | ❌ | ❌ |
| View audit / activity history | ✅ | ✅ | ✅ | ❌ | ✅ |
| Onboard Apollo environments (FIM) | ✅ | ✅ | ❌ | ❌ | ❌ |
| Onboard AWS accounts (EC2 / Lambda / DBM) | ✅ | ✅ | ❌ | ❌ | ❌ |
| Onboard Carnaval alarms | ✅ | ✅ | ❌ | ❌ | ❌ |
| Onboard pipelines via Bindle | ✅ | ✅ | ❌ | ❌ | ❌ |
| Offboard resources (single or bulk) | ✅ | ⚠️ Pending approval | ❌ | ❌ | ❌ |
| View violations | ✅ All apps | ✅ Own apps | ✅ Assigned apps | ❌ | ✅ In-scope apps only |
| Resolve violation (single or bulk) | ✅ | ✅ | ❌ | ❌ | ❌ |
| Ask AI about a violation | ✅ | ✅ | ✅ | ❌ | ⚠️ Formal audit evidence only |
| Export violations as CSV | ✅ | ✅ | ✅ | ❌ | ✅ In-scope only |
| Run ASAP/UDD validation script | ✅ | ✅ | ✅ View only | ❌ | ❌ |
| Request exclusion for a resource | ✅ | ✅ | ❌ | ❌ | ❌ |
| Approve exclusion request | ✅ Only | ❌ | ❌ | ❌ | ❌ |
| Revoke an active exclusion | ✅ | ✅ Own apps | ❌ | ❌ | ❌ |
| View exclusion history | ✅ | ✅ | ✅ | ❌ | ✅ Approved exclusions only |
| Manage exclusion reason codes | ✅ Only | ❌ | ❌ | ❌ | ❌ |
| Manage global excluded file paths | ✅ Only | ❌ | ❌ | ❌ | ❌ |

> ⚠️ **Offboarding policy**: All resource offboarding requires GFRC SOX Admin approval. App archival decisions should be made by the Controllership POC in GRC for GRC-origin apps; Delta-native app archival is managed by the SOX Admin in Delta directly. Confirm final workflow with GFRC before Phase 4 UX finalization.
>
> ⚠️ **AI responses for Third Line**: AI responses shown to Third Line users must be scoped to formal audit evidence only. Internal resolution context, exclusion rationale, and team-internal notes must not be surfaced. This requires a content-filtering layer in the AI response pipeline (see OI-12).


---

## 6. Business Requirements

Priority ratings: Critical · High · Medium · Low

### 6.1 Functional Requirements

#### 6.1.1 Onboarding and Baselining

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-01 | Onboarding | Guided wizard — all onboarding steps in a single linear flow with progress indicator | Critical | — |
| F-02 | Onboarding | Onboarding status visible in app dashboard — closed-loop for all stakeholders | Critical | — |
| F-03 | Onboarding | Complete Delta onboarding for all required services before a GRC-origin app can be published in GRC (unless justified exemption) — prevents teams from publishing with partial monitoring coverage | Critical | Process change; requires advance communication to existing teams |
| F-04 | Onboarding | New apps must justify why DV2 and/or FIM is not enabled — visible to Second and Third Line | High | — |
| F-05 | Onboarding | Enable Delta monitoring start date (prevents draft apps generating violations) | High | — |
| F-06 | Onboarding | Auto-check whether all pipelines are registered; flag result to app owner at SOX 302 review | High | — |
| F-07 | Onboarding | Show pipeline registration and AWS accounts on the same Onboarding tab | Medium | — |
| F-08 | Onboarding | Bindle-based pipeline registration carried over from GRC-NEXT | Medium | — |
| F-09 | Onboarding | Non-production account detection — if an AWS account is identified as non-prod, surface a prompt that monitoring registration is only required for production accounts; if non-prod contains production resources, allow user to proceed with a justification | High | Reduces unnecessary onboarding and exclusion requests |
| F-10 | Onboarding | Multi-source onboarding entry point — wizard adapts based on whether the app is GRC-linked, Delta-native (China), or from another registered client system | Critical | Required for Phase 6 and non-GRC program support |
| F-11 | Baselining | Quarterly baselining reminders (30-day and 7-day) with in-portal completion flow | High | Decision pending: confirm with GFRC whether a separate Delta baseline is needed given planned GRC quarterly 302 App Review integration |

#### 6.1.2 Application Data and Reporting

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-12 | App Data | Complete, accurate data transfer with validation mechanism for all source systems (GRC2 → Delta via EventBridge; Delta-native direct entry; other client system APIs) | Critical | EY audit integrity requires 100% accuracy for GRC-origin apps |
| F-13 | App Data | Show framework membership for each app (SOX, PCI, etc.) | High | — |
| F-14 | App Data | Show publication status (Draft / Published) for GRC-origin apps; show onboarding status for Delta-native apps | High | — |
| F-15 | App Data | Surface the source system for each app record (GRC2, Delta-native, other) — visible to App Owner and SOX Admin | High | Required for operators to understand data provenance |
| F-16 | Reporting | Self-service compliance reporting: framework per app, EY audit scope, monitoring service status, scope change history. Pre-built reports available on-demand; custom report builder is a future consideration (confirm scope with GFRC) | High | Phase TBD |
| F-17 | Reporting | EY audit scope admin page — in-scope / out-of-scope per app with historical change record. GFRC maintains the in-scope/out-of-scope designation in Delta portal | Medium | — |
| F-18 | App Data | GRC application page link in a prominent, accessible location (GRC-origin apps only) | Low | — |

#### 6.1.3 Violations

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-19 | Violations | Single violations page across all 5 services (FIM, Lambda, EC2, Carnaval, DBM) — no tile-per-service navigation | Critical | Default view decision pending (see OI-05) |
| F-20 | Violations | Violations grouped by resource (resource ARN as unique attribute) | Critical | Confirm ASAP/UDD grouping model (see OI-04) |
| F-21 | Violations | Violation detail panel (not popup) — summary, rule evaluation, remediation steps, resolution history | High | — |
| F-22 | Violations | Ask AI about a violation — app-specific constraints; Third Line users see formal audit evidence only | High | — |
| F-23 | Violations | Bulk resolve with structured justification (ticket ref, CM, peer confirmation) | High | — |
| F-24 | Violations | Export violations as CSV — select-all, resource filter, special violation types | Medium | — |
| F-25 | Violations | ASAP/UDD violations — clearly explain which violations cannot be resolved from Delta UI | Medium | — |
| F-26 | Violations | DDB-native pagination replacing in-memory pagination (fixes timeouts at scale) | Critical | — |
| F-27 | Violations | Host Not Reporting violations: portal distinguishes between 'host temporarily unreachable' and 'Chronicle not installed'; for the latter, surface a link to the Chronicle installation guide. Auto-resolved cases are clearly labelled | Medium | — |

#### 6.1.4 Exclusion Management

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-28 | Exclusions | In-portal exclusion request flow — no external SIM ticket required | High | SOX Exclusion workflow covers Framework 24 apps; exclusions for other framework apps: confirm whether process remains manual (see OI-13) |
| F-29 | Exclusions | SOX Admin approval/rejection with response note; in-portal notifications to requester | High | — |
| F-30 | Exclusions | Exclusion history with full approval chain visible to App Viewer and above | High | — |
| F-31 | Exclusions | FIM and DV2 exclusions handled with separate logic (different compliance treatment) | High | — |
| F-32 | Exclusions | Globally-excluded FIM paths surfaced as suggestions when creating app-level exclusion | Medium | Related to FIM chronicle watcher configuration — confirm use case with engineering |
| F-33 | Exclusions | Future: allow exclusion request directly from violation detail panel | Low | See OI-07 |

#### 6.1.5 Monitoring — Resource Type Specifics

**Apollo Environments (FIM)**

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-34 | FIM | Prefix search with multi-select tree — carry forward from DeltaV3 | High | — |
| F-35 | FIM | Right-side panel showing selected environments with option to remove before confirming | High | — |
| F-36 | FIM | Final review page before submission | High | — |
| F-37 | FIM | Evaluate Bindle-based discovery as a complementary approach | Low | Future consideration |

**AWS Accounts (EC2 / Lambda / DBM)**

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-38 | AWS Accounts | Fields per account: Account ID, Account Type (Conduit/Isengard), Services to Monitor (EC2, Lambda, DBM as independent checkboxes), CTI | High | — |
| F-39 | AWS Accounts | Pipeline Name field removed — Hawkeye approach supersedes it; pipeline event is not the source of truth | High | — |
| F-40 | AWS Accounts | China partition: automatically disable unsupported services (only Lambda, EC2, DynamoDB allowed) | High | — |
| F-41 | AWS Accounts | Pre-flight Isengard trust check before submission with deep link to Isengard console if not configured | High | — |
| F-42 | AWS Accounts | Evaluate auto-population of AWS accounts from existing Delta/GRC registrations to reduce manual entry (requested by 2LOD and app owners) | High | Currently manually entered; auto-population would reduce errors and onboarding time |

**Carnaval Alarms**

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-43 | Carnaval | Real-time alarm validation against Carnaval API + Monitor Portal API | High | Carry over from GRC-NEXT |
| F-44 | Carnaval | Rich metadata model: alarm link, description, control ID, CTI, owner, SOX contact | High | — |
| F-45 | Carnaval | Multiple alarms submittable in a single form with a review page before submission | Medium | — |
| F-46 | Carnaval | Only Carnaval alarms that support existing SOX controls require Delta monitoring — the onboarding flow must make this explicit to avoid unnecessary registrations | High | Updated from original draft; not all Carnaval alarms require Delta monitoring |

**Pipelines via Bindle**

| # | Epic / Theme | Requirement | Priority | Notes |
|---|-------------|-------------|----------|-------|
| F-47 | Pipelines | User inputs Bindle name → pipeline names populate automatically | High | — |
| F-48 | Pipelines | User selects desired pipelines; can manage targets per pipeline | High | — |
| F-49 | Pipelines | Offboarding is a separate UX interaction — not embedded in onboarding flow | Medium | — |


### 6.2 Non-Functional Requirements

#### 6.2.1 Performance and Latency

| # | Requirement | Priority |
|---|-------------|----------|
| NF-01 | Apollo prefix search returns results within 3 seconds | High |
| NF-02 | DDB-native pagination must replace in-memory pagination to eliminate timeout failures at scale | Critical |
| NF-03 | Onboarding status page auto-refreshes every 60 seconds when resources are in-progress; manual refresh also available | Medium |
| NF-04 | High-side violation awareness feed auto-refreshes every 4 hours by default | Low |

#### 6.2.2 Accessibility and Error States

| # | Requirement | Priority |
|---|-------------|----------|
| NF-05 | All inline validation errors must be specific and actionable — no generic system errors | High |
| NF-06 | Wizard browser refresh must not lose entered data (state persistence) | Critical |
| NF-07 | Pre-flight Isengard trust check must not block submission if result is inconclusive — warning banner shown instead | High |

#### 6.2.3 Notifications

| # | Requirement | Priority |
|---|-------------|----------|
| NF-08 | In-app notification + email sent on batch completion (success or failure) | High |
| NF-09 | In-portal banner + email to ReadWrite group sent 30 days and 7 days before baselining due date | High |
| NF-10 | Overdue notification sent on due date if baselining not completed | High |
| NF-11 | Controllership POC receives a weekly digest of overdue and upcoming baselining across all apps in scope (not the broader GFRC team) | High | 
| NF-12 | Notification preferences (email on/off, frequency) configurable per app by ReadWrite users | Medium |

#### 6.2.4 Security and Authorization

| # | Requirement | Priority |
|---|-------------|----------|
| NF-13 | AppSec certification required for commercial auth model before production — run review in parallel with Phase 4 | Critical |
| NF-14 | Helis authorization layer — under AppSec evaluation; must be resolved before Phase 4 | High |
| NF-15 | Sensitive high-side data must not be exposed in cross-region feed — only region, violation type, and count surfaced | Critical |
| NF-16 | AI response content filtering — Third Line users must only see formal audit evidence; internal resolution context and exclusion rationale must be suppressed at the response layer | Critical |

#### 6.2.5 Partition and Region Support

| # | Requirement | Priority |
|---|-------------|----------|
| NF-17 | China partition (BJS/ZHY): restrict CTI entries to ZHY-compatible options with in-context creation guide | High |
| NF-18 | China partition: automatically disable unsupported monitoring services (only Lambda, EC2, DynamoDB allowed) | High |
| NF-19 | China partition: Delta is the source of record for app registrations (GRC does not operate in BJS/ZHY) — onboarding wizard must support Delta-native registration without a GRC ID requirement | Critical |
| NF-20 | Cloudscape CDN availability must be verified in all partitions before Phase 5 UI build | Medium |

---

## 7. User Stories — Guided Onboarding and Baselining

**Design principles behind this experience:**
- Onboarding lives in Delta portal — not split across GRC portal and Delta
- The wizard adapts based on the app's source system (GRC-linked, Delta-native, other client)
- Guided wizard with persistent progress state — no freeform tab navigation
- Batch processing status is explicit — users know exactly what is pending and what failed
- Exclusion and baselining approvals are in-portal — no external SIM ticket required
- Quarterly baselining is surfaced and reminded proactively — not tracked outside the portal

---

### Epic 1 — Guided Onboarding Wizard

#### Epic 1 · Story 1.1 — Onboarding Entry Point

As an app owner arriving at the Delta portal for the first time, I want a clear, prominent entry point that initiates a guided onboarding flow, so that I know exactly where to start without navigating across tabs or reading a user guide.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Landing page displays a 'Begin Onboarding' CTA as a primary action, visually distinct from secondary navigation. |
| 2 | CTA is shown only to users who have no existing Delta apps; existing app owners land on their dashboard. |
| 3 | Clicking 'Begin Onboarding' launches the wizard; browser refresh does not lose entered data. |
| 4 | 'Resume Onboarding' prompt appears for users who started but did not complete, showing which step they left off at. |
| 5 | Wizard displays a step indicator (e.g., Step 2 of 5) at all times. |

#### Epic 1 · Story 1.2 — App Identity Setup

As an app owner, I want to provide basic identity information with inline validation and optional GRC ID lookup, so that I can configure the app correctly for my compliance program without leaving the portal.

> **Context**: App owners must first create a SOX application in GRC before Delta onboarding for GRC-linked apps, so the app ID and name can be populated. For China-region apps and non-GRC compliance programs, a GRC ID is not required — Delta assigns its own ID. The wizard must detect and guide appropriately based on compliance program type.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Wizard first prompts for compliance program type to determine the appropriate onboarding path: GRC-linked (SOX/SOC/other GRC program), Delta-native (China/BJS/ZHY or non-GRC program), or Other registered client system. |
| 2 | For GRC-linked apps: wizard prompts for GRC App ID (with 'Look up GRC ID' that queries GRC by partial app name), Application Name (auto-populated from GRC if ID provided), Application Description (optional). |
| 3 | For Delta-native apps: wizard prompts for Application Name (required), Application Description (optional), Compliance Framework (required), Region (required — auto-restricts available services for China). |
| 4 | If a GRC ID already maps to an existing Delta app, an inline warning is shown before the user can advance. |
| 5 | For GRC-linked apps: if no GRC ID is entered, a Delta app ID is auto-assigned. For SOX apps, the GRC App ID is the primary identifier — no separate Delta App ID is surfaced. |
| 6 | Application Name validates for uniqueness inline; conflict is surfaced before the user advances. |
| 7 | User cannot advance until Application Name is populated. |

#### Epic 1 · Story 1.3 — CTI and Resolver Group Configuration

As an app owner, I want to configure my CTI and resolver group with live validation against the SIM system, so that Delta violations are routed to the correct team and I catch errors before monitoring begins.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Cascading dropdowns for Category → Type → Item are populated from live SIM CTI data. |
| 2 | Resolver Group auto-suggests based on selected CTI, with a free-text override option. |
| 3 | Live validation confirms CTI/RG exists and is active; inline error shown if validation fails. |
| 4 | China region users (BJS/ZHY): form restricts to ZHY-compatible CTI entries with an in-context link to the ZHY CTI creation guide. |
| 5 | User cannot advance until CTI validation passes. |

#### Epic 1 · Story 1.4 — Permissions Configuration

As an app owner, I want to assign POSIX/LDAP groups to ReadWrite and ReadOnly roles with real-time group validation, so that I don't misconfigure access controls due to typos in group names.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Two fields: ReadWrite group (required) and ReadOnly group (optional), each accepting one POSIX/LDAP group name. |
| 2 | As the user types, field queries POSIX/LDAP and provides auto-complete suggestions. |
| 3 | Unresolved group name shows an inline error: 'Group [name] not found. Check for typos or create the group before proceeding.' |
| 4 | Validated group name shows a green check and displays the current member count. |
| 5 | User cannot advance until ReadWrite group is validated. |
| 6 | Plain-language summary shown: '[ReadWrite group] members can view and edit. [ReadOnly group] members can view only.' |

#### Epic 1 · Story 1.5 — App Creation Confirmation

As an app owner, I want a summary confirmation screen before my app is created, so that I can review all inputs and catch errors before committing.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Wizard displays read-only summary: App Name, Description, App ID (GRC App ID for SOX/GRC-linked apps; Delta-assigned ID for Delta-native apps), CTI, Resolver Group, ReadWrite Group, ReadOnly Group, Source System. |
| 2 | Each field has an 'Edit' affordance returning the user to that step without resetting subsequent steps. |
| 3 | On confirming, app is created and user advances to resource onboarding without a separate navigation step. |
| 4 | Success banner shows the assigned App ID (GRC App ID for SOX apps; Delta App ID for Delta-native apps) with copy-to-clipboard. |
| 5 | If app creation fails, error message is specific and actionable — not a generic system error. |

---

### Epic 2 — Resource Onboarding

#### Epic 2 · Story 2.1 — Resource Type Selection

As an app owner, I want to see a clear summary of all resource types I can onboard with plain-language descriptions, so that I can make an informed selection without consulting external documentation.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Resource type selection screen shows tiles for all applicable services based on compliance program and region: Apollo Hosts (FIM), Lambda Monitoring, Carnaval Alarms, EC2 Monitoring, DBM (DynamoDB Monitoring). China-region apps automatically suppress FIM and Carnaval tiles. |
| 2 | Each tile includes a one-sentence description of what is monitored and what violation type it generates. |
| 3 | User can select multiple resource types in one session; unselected types can be added later. |
| 4 | 'What should I onboard?' helper text provides decision guidance per type, including which services are required for the app's compliance framework. |
| 5 | Selecting a type expands an inline configuration section — no page navigation. |

#### Epic 2 · Story 2.2 — Apollo Environment Bulk Onboarding

As an app owner, I want to search for my Apollo environments by prefix and bulk-select them from a tree view, so that I can onboard large environment sets without manual entry and avoid human error.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Prefix string (min. 3 characters) searches Apollo APIs; matched environments returned as navigable tree within 3 seconds. |
| 2 | Tree shows branch nodes (child environments) and leaf nodes (stages: Alpha, Beta, Gamma, Prod), sorted alphabetically. |
| 3 | Single click selects/deselects individual stages or entire subtrees; selections shown in real-time 'Selected Resources' panel. |
| 4 | Environments already onboarded to another app are greyed out with a tooltip identifying the owning app. |
| 5 | Search returning 500+ matches prompts the user to narrow prefix; no silent truncation. |
| 6 | Fully qualified environment/stage name can be entered for exact-match lookup. |
| 7 | User sees count of selected resources before confirming (e.g., 'You are about to onboard 27 Apollo environments'). |
| 8 | On confirmation, progress indicator shows enqueued count; failures surface with resource name and reason. |

#### Epic 2 · Story 2.3 — AWS Account Onboarding (Lambda / EC2 / DBM)

As an app owner, I want to enter AWS account IDs for Lambda or EC2 monitoring with inline Isengard trust validation and production/non-production detection, so that I know before submitting whether Delta will be able to connect to my accounts and whether monitoring registration is even required.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Multi-account form: Account ID (required), Account Type — Isengard or Conduit (required), Monitoring Services — EC2, Lambda, DBM as independent checkboxes (required). |
| 2 | Account ID validates as 12-digit numeric inline. |
| 3 | Portal detects whether the account is production or non-production. If non-production, surfaces a prompt: 'Non-production accounts do not require Delta monitoring registration.' If non-production but the owner believes it contains production resources, allows proceed with mandatory justification. |
| 4 | Pre-flight check shows Isengard/Conduit trust status: ✅ Trust configured or ⚠ Trust not detected — onboarding may fail. |
| 5 | If trust not configured, inline guide with direct deep-link to Isengard console is shown. |
| 6 | User is not blocked if trust check is inconclusive; a warning banner is shown. |
| 7 | Onboarding confirmation surfaces next batch window time with instruction to return and verify status. |
| 8 | Future: AWS accounts from existing Delta/GRC registrations surfaced as suggestions to reduce manual entry. |

#### Epic 2 · Story 2.4 — Carnaval Alarms Onboarding

As an app owner, I want to onboard Carnaval alarms that support my SOX controls by URL with real-time validation against the Carnaval API, so that I register only the correct alarms and immediately know if a link is invalid.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Multi-alarm form: HTTPS alarm URL (required), description (optional), control ID (optional). |
| 2 | Each URL is validated in real time against Carnaval API + Monitor Portal API; invalid links are flagged inline before submission. |
| 3 | Inline info callout explains: 'Only register Carnaval alarms that support existing SOX controls. Delta raises a violation for all registered alarm configuration changes — including automated updates.' |
| 4 | Review page shown before final submission. |
| 5 | Onboarding confirmation surfaces batch window countdown consistent with Lambda/EC2 flow. |

#### Epic 2 · Story 2.5 — Onboarding Status Dashboard

As an app owner who has submitted resources for onboarding, I want a clear consolidated status view with a live countdown to the next batch run, so that I don't poll the portal out of uncertainty and immediately know if something failed.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | 'Monitored Resources' tab shows a progress bar with time elapsed, time until next batch window, percentage progress. |
| 2 | Each resource row shows status badge: Onboarding in Progress / Onboarded / Failed / Off-boarding in Progress / Off-boarded. |
| 3 | Failed resources show an inline reason code with a recommended action (e.g., 'EC2: Onboarding Failed — Delta trusted service not configured in Conduit'). |
| 4 | 'Retry Failed Resources' available per failed resource without full re-onboarding. |
| 5 | In-app notification (and optional email) sent when batch completes — success or failure. |
| 6 | Page auto-refreshes every 60 seconds when 'Onboarding in Progress' resources are present; manual refresh also available. |

---

### Epic 3 — Quarterly Baselining

> **Scope note**: Application attestation/baselining for SOX occurs in GRC. A separate Delta baselining workflow risks confusing app owners and appearing duplicative. This epic should be scoped and sequenced carefully against the planned integration of Delta data into the GRC Quarterly 302 App Review. Confirm with GFRC whether a separate Delta baseline is required before finalizing Epic 3 design.

#### Epic 3 · Story 3.1 — Baselining Status Visibility

As an app owner or compliance manager, I want to see baselining status and next due date for each app in a single view, so that I can identify apps approaching or past their quarterly deadline without clicking into each app individually.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | App dashboard shows a Baselining column with: Last Baselined date, Last Baselined by (alias), Next Due date, Status (Up to Date / Due in X Days / Overdue). |
| 2 | Apps with due date within 14 days show a 'Due Soon' warning badge. |
| 3 | Apps past their due date show an 'Overdue' badge in red. |
| 4 | Clicking baselining status launches the baselining review flow inline without full-page navigation. |
| 5 | GFRC compliance managers and Controllership POCs can view a baselining health report across all apps in their scope. |

#### Epic 3 · Story 3.2 — Baselining Review and Confirm Flow

As an app owner performing quarterly baselining, I want to review each configuration section, confirm or update it, and mark baselining complete in one flow, so that the baselining process is structured and produces an auditable completion record.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Baselining flow presents sections sequentially: (1) Service/Feature Attributes, (2) Bindles, (3) Pipeline Resources, (4) Manually Managed Resources. |
| 2 | Each section shows current configured values in read-only form with an 'Edit' affordance. |
| 3 | User must explicitly confirm each section ('Confirmed — no changes' or 'Updated and confirmed'); passive scrolling does not count. |
| 4 | Changes during baselining generate an audit entry: what changed, by whom, when. |
| 5 | Completion summary shows: sections reviewed, sections with changes, completion timestamp. |
| 6 | 'Mark as Baselined' button finalizes submission; action is not reversible. |
| 7 | Baseline Status widget updates immediately: 'Baselined by [alias] on [date]. Next due [date + 90 days].' |
| 8 | Completion record is available in the app's Audit History tab. |

#### Epic 3 · Story 3.0 — First Baselining Pre-Population

As an app owner completing onboarding, I want my first baselining review to be pre-populated with the configuration I entered during onboarding, so that I don't re-enter information I already provided and the audit record reflects a continuous, traceable history from day one.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | First baselining flow presents all onboarding-confirmed sections (service attributes, resources, pipelines) as pre-filled, read-only entries. |
| 2 | App owner can confirm each section or make edits; edits generate an audit entry. |
| 3 | Completion of the onboarding wizard sets the baselining clock — 'Next Due' is 90 days from onboarding confirmation date. |
| 4 | If onboarding was completed before this feature ships, first baselining shows a 'No prior baseline on record' state rather than blank entries. |

#### Epic 3 · Story 3.3 — Baselining Reminders

As an app owner, I want automated reminders when my app's quarterly baselining is approaching or overdue, so that I don't miss the deadline and face compliance exposure.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | In-portal banner + email to ReadWrite group sent 30 days before due date. |
| 2 | Second reminder sent 7 days before due date. |
| 3 | Overdue notification sent on the due date if baselining is not completed. |
| 4 | Notifications include: app name, due date, last baselined date, and a direct deep-link into the baselining review flow. |
| 5 | Notification preferences (email on/off, frequency) configurable per app by ReadWrite users. |
| 6 | Controllership POC receives a weekly digest of overdue and upcoming baselining across all apps in their scope. |

---

### Epic 4 — Exclusion Management

#### Epic 4 · Story 4.1 — Inline Exclusion Request

As an app owner who needs to exclude a resource from Delta monitoring, I want to submit an exclusion request in-portal with a justification field that routes to the compliance approver automatically, so that I don't need to manually cut a SIM ticket and track the approval externally.

> **Scope note**: The SOX Exclusion workflow initially covers Framework 24 apps. For apps under other frameworks, the manual process remains in place until that scope is confirmed and prioritized (see OI-13).

| # | Acceptance Criteria |
|---|---------------------|
| 1 | 'Add Exclusion' request available from FIM Excluded Paths, EC2 Instance Exclusions, Lambda Exclusions, and AWS Region Exclusions sections. |
| 2 | Form captures: resource identifier (path, ARN, account ID, or region), justification (required), supporting ticket/CM reference (optional). |
| 3 | Submission moves request to 'Pending Compliance Approval' status; routed to the designated SOX Admin. |
| 4 | SOX Admin receives in-portal notification and email with approve/reject actions — no separate tool login required. |
| 5 | Requester receives in-portal notification when approved or rejected, with the approver's response note. |
| 6 | Approved exclusions applied immediately with audit record: requester, approver, timestamp, justification, supporting ticket. |
| 7 | Rejected requests surface rejection reason inline and allow resubmission. |
| 8 | Exclusions tab shows all exclusions with status (Active / Pending Approval / Rejected) and full approval chain. |

#### Epic 4 · Story 4.2 — FIM Excluded Paths Suggestions

As an app owner receiving FIM violations from dynamically created files in /apollo/env, I want the portal to surface known globally excluded paths as suggestions when I create an exclusion, so that I can quickly identify whether my issue is already covered without initiating a new request.

> **Note**: This feature is related to chronicle watcher configuration. Confirm the exact use case and data source with engineering before scheduling.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | As user types a file path in the exclusion form, matching globally-excluded paths are suggested. |
| 2 | If a suggested path covers the user's case, inline message shows: 'This path is already excluded globally. No action needed.' |
| 3 | Suggested paths show scope (Global or App-level) and the compliance justification that approved them. |
| 4 | If no global match exists, user proceeds to submit an app-level exclusion request per Story 4.1. |


### Epic 5 — Violation Lifecycle Management

#### Epic 5 · Story 5.1 — Violation Triage View

As an app owner with active violations, I want a consolidated violation view that groups violations by type, severity, and age with inline resolution guidance, so that I can prioritize and act on violations without consulting the user guide.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | Violations grouped by type: FIM File Change, Host Not Reporting, Lambda, Carnaval, EC2. |
| 2 | Each row shows: resource name, hostname (FIM), change path/type, violation status, age in days, ticket ID link, escalation timeline (e.g., 'Escalates to Sev-3 in 4 days'). |
| 3 | 'How to resolve' tooltip available per violation type — plain-language guidance inline. |
| 4 | Violations within 3 days of Sev-3 escalation highlighted with a warning badge. |
| 5 | Violations where user lacks edit permission show the ReadWrite group name to contact. |
| 6 | Host Not Reporting violations clearly distinguish between 'host temporarily unreachable' (auto-resolves) and 'Chronicle not installed' (requires action). For the latter, a direct link to the Chronicle installation guide is surfaced. |

#### Epic 5 · Story 5.2 — Bulk Resolution with Guided Justification

As an app owner resolving multiple FIM violations from a known-good change, I want to bulk-resolve violations with a structured justification form, so that my justification is compliant and I don't inadvertently close violations with insufficient documentation.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | 'Bulk Resolve' available when one or more violations are selected via checkboxes. |
| 2 | Resolution dialog presents structured justification form: Sev-1/Sev-2 ticket reference (if applicable), CM number (if applicable), peer confirmation statement (if no ticket/CM), free text explanation. |
| 3 | At least one justification option must be populated before 'Resolve' button is active. |
| 4 | If a ticket number is entered, portal validates it is a real, open ticket via SIM API (warning if not found; non-blocking). |
| 5 | On resolution, violations transition to RESOLVED; ticket auto-resolves within 24 hours if no active violations remain. |
| 6 | Resolved violations remain visible with RESOLVED status when active-only filter is cleared. |

---

### Epic 6 — Cross-Region Visibility (ADC)

#### Epic 6 · Story 6.1 — Low-Side Violation Awareness Feed

As a low-side app owner whose app also runs in ADC regions, I want a feed of active high-side Delta violations for my app visible in the low-side portal, so that I have situational awareness and can coordinate with my ADC counterparts without relying solely on a Sev-4 email.

| # | Acceptance Criteria |
|---|---------------------|
| 1 | App dashboard includes a 'High-Side Violations' indicator for apps with matching GRC IDs that have active ADC violations. |
| 2 | Clicking expands read-only summary: ADC region, violation type, count of active violations, age of oldest violation. |
| 3 | Sensitive high-side data is not exposed — only region, type, and count are surfaced. |
| 4 | 'Contact ADC Counterpart' action pre-populates a SIM ticket or email template with relevant violation context. |
| 5 | Limina Sev-4 ticket continues to be cut (existing behaviour); this feed is additive. |
| 6 | Feed auto-refreshes every 4 hours by default; last-updated timestamp is shown. |

---

## 8. Metrics and Success Criteria

Baseline values to be collected during SME interviews before Phase 1 kick-off. Targets to be confirmed with GFRC and AWS SA after baseline is established.

| Metric | Current Baseline | Target | Rationale |
|--------|-----------------|--------|-----------|
| Time to complete full app onboarding (minutes) | TBD | TBD | Guided wizard should measurably reduce time and errors |
| Onboarding error rate (% of attempts requiring retry) | TBD | TBD | Inline validation should reduce errors |
| Time to resolve a violation (minutes) | TBD | TBD | Unified page + AI guidance should reduce investigation time |
| Time to complete quarterly baselining (hours) | TBD | TBD | In-portal guided flow replaces multi-system manual process |
| Violation false positive rate (%) | TBD | TBD | ASAP/UDD clarity should reduce false escalations |
| Data sync accuracy GRC2 → Delta (%) | TBD | 100% | Complete transfer is a hard requirement for EY audit integrity |
| Support tickets per month (onboarding-related) | TBD | TBD | Self-service improvements should reduce support load |
| Non-production account registrations prevented (count) | TBD | TBD | New: measures impact of non-prod detection in onboarding |

---

## 9. Open Issues and Decisions Log

### 9.1 Decisions Needed Before Build

| # | Open Item | Owner | Priority | Impact if Unresolved |
|---|-----------|-------|----------|---------------------|
| OI-01 | DBM row granularity — one row per account vs. split RDS/Redshift rows | TBD | 🔴 Critical | Blocks data model and migration stories |
| OI-02 | Offboarding policy — confirm who can offboard; for GRC-origin apps, archival decision belongs to the Controllership POC in GRC; for Delta-native apps, SOX Admin in Delta. Workflow must be built in GRC for GRC-origin apps | GFRC | 🟠 High | UX cannot be finalized without this decision |
| OI-03 | Separate monitoring pages per service vs. unified tiled approach | Michael Bardin / Rajesh Chamarthi | 🟠 High | Drives information architecture and hierarchy |
| OI-04 | Violations grouping taxonomy — confirm ASAP/UDD inclusion and grouping model | TBD | 🟠 High | Violations page design cannot proceed |
| OI-05 | Violations default view — app-scoped or global? | Michael Bardin / Rajesh Chamarthi | 🟠 High | Determines URL structure and navigation model |
| OI-06 | Cloudscape version (rounded) or Jupyter framework — CDN availability in all partitions? | Kristen Liu / Rajesh Chamarthi | 🟡 Medium | Blocks Phase 5 UI build if unavailable in ADC |
| OI-07 | Allow exclusion request from violation detail panel? | Anatoly Ermakov | 🟡 Medium | Affects exclusion UX flow design |
| OI-08 | Information architecture page hierarchy | Kristen Liu | 🟠 High | Must be finalized before Phase 5 UI build |
| OI-09 | EY approval timeline for AWSAccountRegistrations migration (53 references). Note: this is a data migration for the DeltaOne integration, not purely a UI change — Delta registration data and AWS account migration are both in scope | GFRC | 🔴 Critical | Blocks Phase 2/3 if delayed |
| OI-10 | Self-service reporting scope — what phase? Confirm whether GFRC can build their own reports (ad-hoc) or if this is a set of pre-built reports available on demand | GFRC | 🟠 High | Affects Phase sequencing |
| OI-11 | Multi-source inventory architecture — data model and sync strategy for Delta-native apps (China), direct-onboard clients, and other non-GRC compliance programs. Must define: how apps are registered, what fields differ by source system, how permissions and roles apply across sources | Engineering / GFRC | 🔴 Critical | Blocks Phase 6 and non-GRC client support |
| OI-12 | Third Line (EY) role — finalize role definition, LDAP group setup, and AI content filtering requirements. Add to permissions matrix before Phase 4 | GFRC / AppSec | 🟠 High | Security and compliance risk if not addressed before Phase 4 |
| OI-13 | Exclusion workflow coverage — SOX exclusion covers Framework 24 apps; confirm plan for exclusions under other frameworks (manual process continues or extends to Phase N) | GFRC | 🟡 Medium | App owners in other frameworks lack in-portal exclusion path |

### 9.2 Key Risks

| Risk | Detail | Mitigation |
|------|--------|------------|
| EY Approval delay | Merging GRC-NEXT and DeltaV3 account registrations requires external audit approval; could extend Phase 2/3. Includes both UI and underlying data migration scope | Initiate approval request immediately; run dual write in parallel while approval is in progress |
| Data migration inconsistency | 53 downstream references to AWSAccountRegistrations; dual-write period creates inconsistency window | Feature-flagged dual-write; automated validation job comparing item counts post-migration |
| Multi-source inventory complexity | Introducing Delta-native and non-GRC source systems adds data model complexity and risks inconsistent behavior for app owners across source types | Define multi-source architecture (OI-11) before Phase 6 design begins; isolate source-system-specific logic behind adapters |
| Compliance gate impact | Requiring complete Delta onboarding before GRC publication is a process change for existing teams | Communicate change window in advance; provide guided migration path for apps in Draft state |
| Authorization certification | AppSec certification required for commercial auth model before production | Run AppSec review in parallel with Phase 4 development; start engagement early |
| Partition differences | Carnaval disabled in China; auth flows differ for ADC; Cloudscape CDN unverified in all partitions; GRC does not operate in BJS/ZHY | Validate CDN availability in Phase 6 pre-work; maintain partition config flags per region; design Delta-native onboarding path for China before Phase 6 |
| Third Line access scope creep | Without a clearly defined and technically enforced Third Line role, AI content filtering may be bypassed or inconsistently applied | Define and implement AI content filtering layer before Phase 4 ships (OI-12) |

---

## 10. Integration Dependencies

| System | Status | Purpose |
|--------|--------|---------|
| GRC2 / GRCRMS | Existing | Application registry and event source for GRC-origin apps; EventBridge sync drives One Delta Portal application data for commercial regions |
| DeltaControlPlane | Existing | Violation tracking, ticket creation and lifecycle management |
| Apollo | Existing | FIM environment monitoring; prefix-search API for onboarding |
| Bindles | Existing | Pipeline registration and access control |
| Radar API Gateway | Existing | Pipeline/bindle data lookup — One Delta Portal calls directly rather than storing pipeline data |
| Carnaval API + Monitor Portal API | Existing | Real-time alarm validation at onboarding time |
| ADMS | Existing | AWS account existence validation during onboarding |
| TT / Ticket | Existing | Violation ticket creation and escalation management |
| GRC2 EventBridge | New | Real-time sync of application and resource events from GRC2 to One Delta Portal (GRC-origin apps only) |
| Helis | Under evaluation | Authorization layer — under AppSec evaluation |
| Fintech UI Framework | New | Frontend component library |
| Cloudscape (updated) | New — verify CDN | Rounded shapes, new components — CDN availability must be confirmed in all partitions before Phase 5 |
| Delta (as source of record) | New — Phase 6 | For China (BJS/ZHY) and non-GRC clients, Delta itself is the application registry — no GRC dependency; requires Delta-native registration APIs and data model extensions |
| Non-GRC Client System APIs | New — Phase 6 | For compliance programs that onboard to Delta monitoring outside of GRC; integration contracts TBD per client system |

---

## Appendix: Comment Resolution Log

The following inline comments from the April 2026 draft have been resolved in this version:

| Comment | Resolution |
|---------|------------|
| Self-service reporting — which phase? | Added to OI-10; noted as Phase TBD in F-16 |
| Third Line (EY) column missing from permissions matrix | Third Line column added to Section 5; AI content filtering requirement added as NF-16; OI-12 created |
| App owner must first create SOX app in GRC before Delta onboarding | Addressed in Story 1.2 with compliance program type detection step |
| F-03 wording — clarify "complete Delta onboarding before publishing in GRC" | Reworded to: "Complete Delta onboarding for all required services before a GRC-origin app can be published in GRC" |
| EY/Third Line 'View as EY' option for GFRC | Third Line role added to personas and permissions; scoped read-only access defined; OI-12 created for implementation |
| AI responses scoped to app owner's data; Third Line restricted to formal audit evidence | AI scoping requirement added to F-22, NF-16, permissions matrix note, and OI-12 |
| Story 2.1 — why only those services listed? | Story 2.1 AC-1 updated to include all 5 services (FIM, Lambda, Carnaval, EC2, DBM) with region-based suppression |
| Story 2.3 — auto-populate AWS accounts from existing registrations | Added as F-42 (High priority) and as AC-8 in Story 2.3 |
| Story 2.3 — non-prod account detection | Added as F-09 and AC-3 in Story 2.3 |
| Story 2.4 — not all Carnaval alarms require Delta monitoring | Updated callout wording in Story 2.4 AC-3 and requirement F-46 |
| Story 3 — baselining may be duplicative given GRC quarterly 302 App Review | Scope note added to Epic 3 header; F-11 updated with open question |
| NF-11 — weekly digest to Controllership POC, not GFRC team | NF-11 updated to 'Controllership POC'; confirmed in Story 3.3 AC-6 |
| Offboarding — app archival decision belongs to Controllership POC in GRC | OI-02 updated with GRC-origin vs. Delta-native split; risk table updated |
| Exclusion — SOX covers Framework 24 only; other frameworks manual | Scope note added to Story 4.1; OI-13 created |
| Story 5.1 — Host Not Reporting: distinguish unreachable vs. Chronicle not installed | Updated AC-6 in Story 5.1 and requirement F-27 |
| Story 1.5 — SOX apps use GRC App ID, no separate Delta App ID | Addressed in Story 1.5 AC-1 and AC-4 with source-system-aware ID handling |
| China region — GRC does not operate; Delta is source of record | Core Design Principle updated; NF-19 added; Story 1.2 multi-source detection added; OI-11 created; Phase 6 scope updated |
| Non-GRC compliance programs | Section 2.1 updated; F-10 added; NF-19 updated; OI-11 created; Phase 6 updated; Integration table updated |
| EY approval — clarify it covers both UI and data migration | OI-09 updated with clarification |

---

*Amazon Confidential*

*Built with pm-builder-agent*
