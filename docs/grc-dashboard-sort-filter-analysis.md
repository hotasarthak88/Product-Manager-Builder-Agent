# GRC Dashboard Sort & Filter Analysis

**Date:** June 8, 2026  
**Author:** PM Builder Agent (for sarthah)  
**Purpose:** Column-by-column Sort and Filter recommendations for all GRC document dashboard views, with reasoning and implementation logic.

---

## Evaluation Criteria

**Sort = Yes** when:
- The cell maps to a single, comparable value (scalar, not a list)
- "Ordered list of these" answers a real user question (e.g., "which was created most recently?")

**Sort = No** when:
- The cell contains multiple values (multi-select, lists of people/tags)
- No meaningful natural ordering exists
- Sorting would produce confusing results (e.g., sorting by "5 People")

**Filter = Yes** when:
- Users have a known target value or category they want to narrow by
- The column has a finite, enumerable set of values (states, types, roles)
- "Show me only X" is a real workflow

**Filter = No** when:
- The column has near-infinite cardinality with no common groupings
- Filtering would produce no useful narrowing (e.g., custom permissions lists)
- The value is already unique per row (like a document UUID)

---

## 1. Applications Dashboard

**URL:** `https://grc.a2z.com/list/Application?viewAs=sarthah`  
**Total columns captured:** 20  
**Total records:** 9,843

| # | Column | Sort | Filter | Reasoning | Implementation Logic |
|---|--------|------|--------|-----------|---------------------|
| 1 | **ID** | ✅ Yes | ✅ Yes | Numeric sequential identifier. Sort: users look for "newest" or "oldest" by ID. Filter: users know a specific ID and want to jump to it. | Sort: numeric ascending/descending. Filter: text input, exact or prefix match on numeric string. |
| 2 | **Name** | ✅ Yes | ✅ Yes | Single text value, alphabetically sortable. Filter: users search by known application name (e.g., "PeopleSoft"). | Sort: lexicographic A→Z / Z→A, case-insensitive. Filter: text input with substring/contains match. |
| 3 | **Type** | ❌ No | ✅ Yes | All values observed are "Application Document" — no meaningful sort order when values are identical. Filter: useful if multiple types exist; even with one type, it's a valid category filter for cross-dashboard queries. | Sort: N/A (single-value or low-cardinality enum — not useful). Filter: dropdown/multi-select with enumerated type values. |
| 4 | **Is Template** | ✅ Yes | ✅ Yes | Boolean (Yes/No). Sort: group templates together. Filter: "show me only templates" or "exclude templates" is a common workflow. | Sort: boolean sort (No before Yes or vice versa). Filter: checkbox or Yes/No/All toggle. |
| 5 | **State** | ✅ Yes | ✅ Yes | Enum (Published, Draft, Archived). Sort: group by lifecycle state. Filter: "show me only Published" is the #1 use case — users rarely want to see Archived mixed with Published. | Sort: enum order (Draft → Published → Archived). Filter: multi-select dropdown with state values. **Recommend: default filter to "Published" only.** |
| 6 | **Significance** | ✅ Yes | ✅ Yes | Enum (Key, Non-Key, or "-"). Sort: group Key applications at top. Filter: "show me only Key applications" is critical for SOX scoping. | Sort: enum order (Key → Non-Key → unset). Filter: dropdown (Key / Non-Key / All). |
| 7 | **Related Controls** | ✅ Yes | ❌ No | Numeric count (e.g., "26", "8", "0"). Sort: "which apps have the most controls?" answers a real question for scoping. Filter: no user would filter by exact count — range filter would be possible but low value. | Sort: numeric descending (most related first). Filter: not recommended (consider range filter only if explicitly requested: "0", "1-10", "10+"). |
| 8 | **Related Applications** | ✅ Yes | ❌ No | Numeric count. Sort: find highly-connected apps. Filter: same as above — exact count filtering is not a real workflow. | Sort: numeric descending. Filter: not recommended. |
| 9 | **Related Processes** | ✅ Yes | ❌ No | Numeric count. Sort: find apps with many process dependencies. Filter: low value. | Sort: numeric descending. Filter: not recommended. |
| 10 | **Related Risks** | ✅ Yes | ❌ No | Numeric count. Sort: find highest-risk apps. Filter: low value at exact-count level. | Sort: numeric descending. Filter: not recommended (but consider "Has Risks: Yes/No" boolean filter). |
| 11 | **Privacy Level** | ✅ Yes | ✅ Yes | Enum (Private, Public, Confidential). Sort: group by sensitivity. Filter: "show me only Private" or "show me Public-facing apps" is a real compliance workflow. | Sort: enum order (Public → Private → Confidential). Filter: dropdown multi-select. |
| 12 | **Geographies** | ❌ No | ✅ Yes | Multi-value tag list (e.g., "APA DCA Global LCK", "DE FR GB"). Sort: no meaningful sort on a multi-value set. Filter: "show me all applications in US" or "show me Global" is extremely common for regional compliance teams. | Sort: N/A (multi-value). Filter: multi-select dropdown with all geography values (Global, US, DE, JP, etc.). Match = row contains ANY selected geography. |
| 13 | **My Roles** | ❌ No | ✅ Yes | Multi-value (e.g., "Viewer, Custom", "Editor, Viewer, Custom"). Sort: no natural order. Filter: "show me things where I'm an Owner" or "where I'm an Editor" is a key workflow. | Sort: N/A (multi-value). Filter: multi-select (Owner, Editor, Viewer, Custom). Match = row contains ANY selected role. |
| 14 | **Published on** | ✅ Yes | ✅ Yes | Date-time value. Sort: "most recently published" is a top user question. Filter: date range filter for "published in last 30/90 days" or "published before X". | Sort: chronological ascending/descending. Filter: date range picker (from → to). Consider presets: "Last 30 days", "Last 90 days", "This year". |
| 15 | **Owners** | ❌ No | ✅ Yes | Person list (e.g., "5 People" or "balarant(Principal)"). Sort: "5 People" vs "2 People" is not meaningful ordering. Filter: "show me applications owned by [person]" is critical for org-level views. | Sort: N/A (person lists or counts). Filter: person/alias search input. Match = row's owner list contains the entered alias. |
| 16 | **Editors** | ❌ No | ✅ Yes | Same as Owners — person list with count display. Filter: "show me where I'm an editor" or "show me [alias]'s edit permissions". | Sort: N/A. Filter: person/alias search input. |
| 17 | **Viewers** | ❌ No | ❌ No | Person count display ("28 People"). Sort: meaningless. Filter: rarely would a user say "show me things with exactly 28 viewers" — the "My Roles" filter already covers "where am I a viewer". | Sort: N/A. Filter: not recommended (covered by My Roles filter). |
| 18 | **Custom Permissions** | ❌ No | ❌ No | Person count display. Sort: meaningless. Filter: low value — permission-based filtering is better served by "My Roles" or "Owners" columns. | Sort: N/A. Filter: not recommended. |
| 19 | **Created on** | ✅ Yes | ✅ Yes | Date-time value. Sort: "newest created" is useful for tracking recent additions. Filter: date range for "created in the last week" helps identify new documents. | Sort: chronological ascending/descending. Filter: date range picker with presets. |
| 20 | **Archived on** | ✅ Yes | ✅ Yes | Date-time value (or "-" if not archived). Sort: "most recently archived" helps audit cleanups. Filter: "show me things archived after X date" or "not archived" (value = "-"). | Sort: chronological (nulls/"-" last). Filter: date range OR simple "Archived: Yes/No" toggle. |

---

## 2. Reviews Dashboard

**URL:** `https://grc.a2z.com/list/Review?viewAs=umashanc`  
**Total columns captured:** 27

| # | Column | Sort | Filter | Reasoning | Implementation Logic |
|---|--------|------|--------|-----------|---------------------|
| 1 | **Review ID** | ✅ Yes | ✅ Yes | Numeric identifier. Sort: find newest/oldest reviews. Filter: jump to a known review. | Sort: numeric asc/desc. Filter: exact or prefix text input. |
| 2 | **Review Name** | ✅ Yes | ✅ Yes | Single text value. Sort: alphabetical browsing. Filter: search by known name. | Sort: lexicographic. Filter: substring/contains search. |
| 3 | **Item Under Review** | ✅ Yes | ✅ Yes | The control/application being reviewed (single reference). Sort: group reviews by item. Filter: "show me all reviews for [Control X]" is a core workflow. | Sort: lexicographic. Filter: text search with autocomplete (matches against known item names). |
| 4 | **Review Set ID** | ✅ Yes | ✅ Yes | Numeric. Sort: group reviews by set. Filter: "show me all reviews in set 47" is a common workflow for set-level oversight. | Sort: numeric. Filter: exact numeric input. |
| 5 | **Review Set Name** | ✅ Yes | ✅ Yes | Text value linking to the parent set. Sort: alphabetical grouping. Filter: "show me reviews in H1 2026 Review Set". | Sort: lexicographic. Filter: dropdown or text search. |
| 6 | **Period start date** | ✅ Yes | ✅ Yes | Date. Sort: chronological ordering of review periods. Filter: "show me reviews starting after Jan 2026". | Sort: chronological. Filter: date range picker. |
| 7 | **Period end date** | ✅ Yes | ✅ Yes | Date. Sort: find reviews with latest/earliest end dates. Filter: "ending before X" for deadline tracking. | Sort: chronological. Filter: date range picker. |
| 8 | **In-Scope Document ID** | ✅ Yes | ✅ Yes | Numeric reference to the scoped document. Sort: group by document. Filter: "all reviews against document #52". | Sort: numeric. Filter: exact input. |
| 9 | **In-Scope Document Name** | ✅ Yes | ✅ Yes | Text reference. Sort: alphabetical. Filter: search by document name. | Sort: lexicographic. Filter: text search. |
| 10 | **Review type** | ❌ No | ✅ Yes | Low-cardinality enum (e.g., "Periodic", "Ad-hoc"). Sort: all reviews of same type would cluster — limited value with low cardinality. Filter: "show me only Periodic reviews" is critical. | Sort: not needed (low cardinality). Filter: dropdown multi-select. |
| 11 | **Status** | ✅ Yes | ✅ Yes | Enum (Open, Submitted, Approved, etc.). Sort: group by lifecycle. Filter: "show me only Open reviews" is the #1 filter — users want their active work. | Sort: enum order (Open → In Progress → Submitted → Approved). Filter: multi-select dropdown. **Recommend: default to Open.** |
| 12 | **Reviewer** | ❌ No | ✅ Yes | Person alias. Sort: alphabetical by name is possible but low value. Filter: "show me reviews assigned to [person]" is essential for managers. | Sort: N/A (person name sort is low value). Filter: person/alias search. |
| 13 | **Due Date** | ✅ Yes | ✅ Yes | Date. Sort: "what's due soonest?" is the most critical sort in this dashboard. Filter: "overdue" (before today) or "due this week". | Sort: chronological ascending (soonest first). Filter: date range + preset "Overdue", "Due this week", "Due this month". **Recommend: default sort by Due Date ascending.** |
| 14 | **First Approver** | ❌ No | ✅ Yes | Person alias. Sort: low value. Filter: "show me reviews where I'm first approver" is critical for approver workflow. | Sort: N/A. Filter: person/alias search. |
| 15 | **First Approver Due Date** | ✅ Yes | ✅ Yes | Date. Sort: soonest approval deadlines. Filter: "overdue approvals". | Sort: chronological. Filter: date range + "Overdue" preset. |
| 16 | **Second Approver** | ❌ No | ✅ Yes | Person alias. Same logic as First Approver. | Sort: N/A. Filter: person/alias search. |
| 17 | **Second Approver Due Date** | ✅ Yes | ✅ Yes | Date. Same logic as First Approver Due Date. | Sort: chronological. Filter: date range + "Overdue" preset. |
| 18 | **Type** | ❌ No | ✅ Yes | Document type enum. Filter: standard type filtering. | Sort: N/A. Filter: dropdown. |
| 19 | **Template/Document** | ❌ No | ✅ Yes | Boolean-like (Template vs Document). Filter: exclude templates from working views. | Sort: N/A. Filter: toggle. |
| 20 | **State** | ✅ Yes | ✅ Yes | Document lifecycle state (Published, Draft, Archived). Same logic as Applications. | Sort: enum order. Filter: multi-select. |
| 21 | **Review owners** | ❌ No | ✅ Yes | Person list. Filter: "reviews I own" or "reviews owned by [manager]". | Sort: N/A. Filter: person/alias search. |
| 22 | **Editors** | ❌ No | ✅ Yes | Person list. Filter: person search. | Sort: N/A. Filter: person search. |
| 23 | **Viewers** | ❌ No | ❌ No | Person count. Low value for filtering (covered by My Roles). | Sort: N/A. Filter: N/A. |
| 24 | **Published On** | ✅ Yes | ✅ Yes | Date. Sort: recency. Filter: date range. | Sort: chronological. Filter: date range picker. |
| 25 | **Created On** | ✅ Yes | ✅ Yes | Date. Sort: recency. Filter: date range. | Sort: chronological. Filter: date range picker. |
| 26 | **Archived On** | ✅ Yes | ✅ Yes | Date. Sort: recency. Filter: date range or "is archived" toggle. | Sort: chronological (nulls last). Filter: date range or Yes/No toggle. |
| 27 | **Effectiveness** | ✅ Yes | ✅ Yes | Enum or outcome value (Effective / Not Effective / Pending). Sort: group by effectiveness. Filter: "show me only Not Effective reviews" is critical for remediation workflows. | Sort: enum order. Filter: dropdown. **High-priority filter for oversight users.** |

---

## 3. Action Items Dashboard

**URL:** `https://grc.a2z.com/list/ActionItem?documentState=Open&viewAs=umashanc`  
**Total columns captured:** 10

| # | Column | Sort | Filter | Reasoning | Implementation Logic |
|---|--------|------|--------|-----------|---------------------|
| 1 | **ID** | ✅ Yes | ✅ Yes | Numeric. Sort/Filter: standard identifier use. | Sort: numeric. Filter: exact input. |
| 2 | **Action Item Name** | ✅ Yes | ✅ Yes | Text. Sort: alphabetical. Filter: search by keyword. | Sort: lexicographic. Filter: substring search. |
| 3 | **Parent/Source** | ✅ Yes | ✅ Yes | Reference to originating document (Review, Finding, etc.). Sort: group action items by source. Filter: "show me all action items from Review X" is essential. | Sort: lexicographic. Filter: text search or linked-document picker. |
| 4 | **Status** | ✅ Yes | ✅ Yes | Enum (Open, In Progress, Closed, Overdue). Sort: group by status. Filter: "Open only" is the default view. | Sort: enum order. Filter: multi-select. **Already filtered by URL param `documentState=Open`.** |
| 5 | **Owner** | ❌ No | ✅ Yes | Person alias. Filter: "action items assigned to me" or "assigned to [person]" is the primary workflow. | Sort: N/A. Filter: person search. |
| 6 | **Requestor** | ❌ No | ✅ Yes | Person alias. Filter: "action items I requested" or "requested by [manager]". | Sort: N/A. Filter: person search. |
| 7 | **Start Date** | ✅ Yes | ✅ Yes | Date. Sort: chronological. Filter: range. | Sort: chronological. Filter: date range. |
| 8 | **Current Due Date** | ✅ Yes | ✅ Yes | Date. Sort: "what's due soonest?" is critical. Filter: "overdue" (before today). | Sort: chronological ascending. Filter: date range + "Overdue" preset. **Recommend: default sort.** |
| 9 | **Completion Date** | ✅ Yes | ✅ Yes | Date (null if not complete). Sort: "most recently completed". Filter: "completed in Q1 2026". | Sort: chronological (nulls last). Filter: date range. |
| 10 | **Importance** | ✅ Yes | ✅ Yes | Enum (High, Medium, Low, Critical). Sort: highest importance first. Filter: "show me only Critical/High items". | Sort: enum order (Critical → High → Medium → Low). Filter: multi-select. |

---

## 4. Standard Document Dashboards (Shared Schema)

**Applies to:** Process, Controls, Risks, Frameworks, Findings, General Documents, IPEs, Review Sets, Projects

These dashboards share a **common column schema** (with minor additions for some — notably Risks and Controls have "Related Controls/Applications/Processes/Risks" columns, and Applications adds "Significance").

### Base columns (all standard dashboards):

| # | Column | Sort | Filter | Reasoning | Implementation Logic |
|---|--------|------|--------|-----------|---------------------|
| 1 | **ID** | ✅ Yes | ✅ Yes | Numeric identifier. | Sort: numeric. Filter: exact/prefix. |
| 2 | **Name** | ✅ Yes | ✅ Yes | Text value. | Sort: lexicographic. Filter: substring search. |
| 3 | **Type** | ❌ No | ✅ Yes | Low-cardinality enum. | Sort: N/A. Filter: dropdown. |
| 4 | **Is Template** | ✅ Yes | ✅ Yes | Boolean. | Sort: boolean. Filter: Yes/No toggle. |
| 5 | **State** | ✅ Yes | ✅ Yes | Enum (Published/Draft/Archived). | Sort: enum order. Filter: multi-select. **Default: Published.** |
| 6 | **Privacy Level** | ✅ Yes | ✅ Yes | Enum. | Sort: enum order. Filter: dropdown. |
| 7 | **Geographies** | ❌ No | ✅ Yes | Multi-value tag. | Sort: N/A. Filter: multi-select (contains any). |
| 8 | **My Roles** | ❌ No | ✅ Yes | Multi-value. | Sort: N/A. Filter: multi-select (Owner/Editor/Viewer/Custom). |
| 9 | **Published on** | ✅ Yes | ✅ Yes | Date. | Sort: chronological. Filter: date range. |
| 10 | **Owners** | ❌ No | ✅ Yes | Person list. | Sort: N/A. Filter: person search. |
| 11 | **Editors** | ❌ No | ✅ Yes | Person list. | Sort: N/A. Filter: person search. |
| 12 | **Viewers** | ❌ No | ❌ No | Count display. | Sort: N/A. Filter: N/A (covered by My Roles). |
| 13 | **Custom Permissions** | ❌ No | ❌ No | Count display. | Sort: N/A. Filter: N/A. |
| 14 | **Created on** | ✅ Yes | ✅ Yes | Date. | Sort: chronological. Filter: date range. |
| 15 | **Archived on** | ✅ Yes | ✅ Yes | Date (nullable). | Sort: chronological (nulls last). Filter: date range or Yes/No toggle. |

### Additional columns for Controls / Risks / Frameworks / Applications:

| # | Column | Sort | Filter | Reasoning | Implementation Logic |
|---|--------|------|--------|-----------|---------------------|
| A1 | **Significance** | ✅ Yes | ✅ Yes | Enum (Key/Non-Key). | Sort: Key first. Filter: dropdown. |
| A2 | **Related Controls** (count) | ✅ Yes | ❌ No | Numeric count. Sort: find highly-connected entities. | Sort: numeric desc. Filter: N/A (consider "Has controls: Yes/No"). |
| A3 | **Related Applications** (count) | ✅ Yes | ❌ No | Numeric count. | Sort: numeric desc. Filter: N/A. |
| A4 | **Related Processes** (count) | ✅ Yes | ❌ No | Numeric count. | Sort: numeric desc. Filter: N/A. |
| A5 | **Related Risks** (count) | ✅ Yes | ❌ No | Numeric count. | Sort: numeric desc. Filter: N/A. |

---

## Summary: Implementation Priority Matrix

### Highest-Value Filters (implement first):

| Dashboard | Column | Why |
|-----------|--------|-----|
| All | **State** | Every user wants Published-only by default. Reduces noise by 30-50%. |
| Reviews | **Status** | "Show me only Open reviews" is the #1 workflow. |
| Reviews | **Due Date** | "Overdue" filter prevents missed deadlines. |
| Reviews | **Reviewer** | Managers need "reviews assigned to [person]" daily. |
| Reviews | **Effectiveness** | Oversight users need "Not Effective" reviews immediately. |
| Action Items | **Owner** | "My action items" is the default mental model. |
| Action Items | **Current Due Date** | "Overdue" filter is critical. |
| All | **Geographies** | Regional compliance teams only need their region. |
| All | **Owners** | "Things I own" or "things [person] owns" is universal. |
| Applications | **Significance** | SOX scoping requires Key/Non-Key separation. |

### Highest-Value Sorts (implement first):

| Dashboard | Column | Default? | Why |
|-----------|--------|----------|-----|
| Reviews | **Due Date** | ✅ Default ascending | Soonest-due first prevents missed deadlines. |
| Action Items | **Current Due Date** | ✅ Default ascending | Same as above. |
| All | **Published on** | Optional default descending | "What changed recently?" is a common entry point. |
| All | **ID** | Current default | Sequential gives "creation order" which is intuitive. |
| Reviews | **Effectiveness** | No | Useful for oversight but not default. |

---

## Implementation Logic: Technical Conditions

### Sort Implementation

```
SORT CONDITIONS:
- Numeric columns (ID, counts): compare as integers, not strings
- Date columns: compare as ISO timestamps (null/"—" sorted LAST in ascending, FIRST in descending)
- Enum columns: define explicit order map:
  - State: { Draft: 0, Published: 1, Archived: 2 }
  - Significance: { Key: 0, "Non-Key": 1, "-": 2 }
  - Status (Reviews): { Open: 0, "In Progress": 1, Submitted: 2, Approved: 3 }
  - Importance (Action Items): { Critical: 0, High: 1, Medium: 2, Low: 3 }
  - Effectiveness: { "Not Effective": 0, Pending: 1, Effective: 2 }
- Text columns: case-insensitive lexicographic (locale-aware for internationalization)
- Boolean columns: treat as enum { No: 0, Yes: 1 }
- Multi-value columns: DO NOT implement sort (no meaningful ordering)
- Person/count columns (e.g., "5 People"): DO NOT implement sort
```

### Filter Implementation

```
FILTER CONDITIONS:
- Text search (Name, Item Under Review): 
  - Type: free-text input
  - Match: case-insensitive substring/contains
  - Debounce: 300ms after typing stops
  
- Enum filters (State, Status, Type, Significance, Effectiveness, Importance):
  - Type: multi-select dropdown
  - Match: row.value IN selectedValues
  - If nothing selected: show all (no filter applied)
  
- Date range filters (Published on, Due Date, Created on, etc.):
  - Type: date range picker (from → to)
  - Presets: "Last 7 days", "Last 30 days", "Last 90 days", "This year", "Overdue" (< today)
  - Match: from <= row.date <= to
  - Null handling: null dates excluded from range filters unless "Include empty" is checked
  
- Person filters (Owners, Editors, Reviewer, Approver, Owner):
  - Type: text input with autocomplete (alias lookup)
  - Match: row.personList.contains(enteredAlias)
  - Allow multiple aliases (OR logic)
  
- Multi-value tag filters (Geographies, My Roles):
  - Type: multi-select chips/dropdown
  - Match: row.tags.intersects(selectedTags) (ANY match, not ALL)
  
- Boolean filters (Is Template):
  - Type: segmented control (All / Yes / No)
  - Match: exact value match
  
- ID/Numeric exact filters:
  - Type: text input
  - Match: exact or prefix (startsWith)
```

### Combined Filter Logic

```
When multiple filters are active simultaneously:
  finalResult = rows.filter(row => 
    filter1(row) AND filter2(row) AND filter3(row) ...
  )
  
All filters combine with AND logic.
Within a single multi-select filter, values combine with OR logic.
Example: State=[Published, Draft] AND Geography=[US, Global] 
  → shows rows that are (Published OR Draft) AND (in US OR Global)
```

### Performance Considerations

```
For dashboards with 9,843+ records (Applications):
- Server-side sort and filter (not client-side)
- Paginate results (current: page-based navigation observed)
- Debounce text search inputs (300ms)
- Cache filter dropdown options (enum values don't change frequently)
- For person search: use backend autocomplete API with 3-char minimum
- For date filters: index date columns for range queries
```

---

## Columns NOT Recommended for Sort or Filter

| Column | Why Neither |
|--------|-------------|
| Viewers (count) | Count isn't actionable; "My Roles" covers the "am I a viewer?" use case |
| Custom Permissions (count) | Same reasoning — no user workflow requires filtering by permission count |

---

## Current State vs Recommendation (Observed from Live Dashboard)

From the Applications dashboard snapshot, I observed that some columns **already have sort** (indicated by a clickable button with sort icon):
- ✅ Already sortable: ID, Name, Is Template, State, Privacy Level, Published on, Created on, Archived on
- ❌ Not sortable (but should be per above): Significance, Related Controls/Apps/Processes/Risks counts
- ✅ Correctly not sortable: Type, Geographies, My Roles, Owners, Editors, Viewers, Custom Permissions

**Gap:** The "Significance" column and all "Related X" count columns should be sortable but currently are not (they render as plain text without the sort button). This is a missed opportunity — SOX-scoped users frequently need to sort by significance.

---

*Built with pm-builder-agent*
