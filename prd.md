# PRD — CRM & Sales Management System

**Version:** 1.0
**Status:** Draft
**Difficulty:** Intermediate to Advanced
**Stack:** Laravel (PHP), MySQL/PostgreSQL, REST API, Blade/Inertia/Vue or React frontend
**Document Owner:** Product / Engineering Lead

---

## 1. Overview

### 1.1 Product Summary
A CRM & Sales Management System that helps companies manage **leads, customers, and sales teams** in one place. It replaces spreadsheets and scattered notes with a single workflow engine that tracks every prospect, records every interaction, prompts the next action, and measures performance against sales targets.

### 1.2 Problem Statement
Sales teams lose deals because:
- Leads are forgotten after the first contact.
- Communication history is stored in personal inboxes and lost when a rep leaves.
- Managers have no real-time visibility into pipeline health.
- Targets are tracked manually and reported late.
- Permissions are unclear — everyone can see and edit everything.

### 1.3 Product Vision
> A CRM that is not a digital address book, but an **active workflow system**: a lead enters the pipeline, every interaction is recorded, the system proactively prompts the next action, and results are measured against goals — all governed by a clear hierarchy of roles and permissions.

### 1.4 Target Users
- B2B and B2C sales companies (SMB to mid-market)
- Sales teams of 3–100 reps
- Sales operations and management

---

## 2. Goals & Success Metrics

| Goal | Metric | Target |
|---|---|---|
| Reduce lost leads due to missed follow-up | % of leads with an overdue task | < 5% |
| Improve pipeline visibility | % of deals with complete stage history | 100% |
| Increase follow-up consistency | Avg. days between touches on active lead | ≤ 3 days |
| Improve manager oversight | Time to view team pipeline report | < 10 seconds |
| Improve accountability | % of reps with a set monthly target | 100% |
| Secure access control | Unauthorized access attempts blocked | 100% |

---

## 3. Scope

### 3.1 In Scope (v1)
- Lead tracking and sales pipelines
- Customer communication history
- Follow-up reminders and tasks
- Sales targets and analytics dashboards
- Authentication, authorization, and role-based access control
- Scheduled commands and notifications
- Dashboard APIs

### 3.2 Out of Scope (v1)
- Email inbox sync (IMAP/Gmail/Outlook)
- Telephony / VoIP integration
- Marketing automation (campaigns, drip emails)
- Billing and invoicing
- Customer-facing portal
- Mobile native apps (responsive web only)
- Multi-tenant SaaS mode (planned for v2)

---

## 4. User Roles & Permissions

### 4.1 Role Hierarchy

```
Super Admin        → platform/system owner (bypasses all checks)
   │
   ▼
Admin              → runs the company's CRM
   │
   ▼
Sales Manager      → runs a team
   │
   ▼
Sales Representative → works own leads
   │
   ▼
Analyst / Viewer   → read-only insight
```

### 4.2 Role Definitions

#### 4.2.1 Super Admin
**Purpose:** System/platform owner. Runs the CRM itself, not the sales business.

**Capabilities:**
- Bypass **all** permission and policy checks via `Gate::before`
- Manage Admins (create, suspend, delete)
- Reset passwords for any user
- Force logout / revoke sessions and API tokens
- Impersonate any user (audit-logged, signed short-lived token)
- Access system health, queues, failed jobs, and logs
- Manage API keys, webhooks, and integrations
- Restore or permanently delete any record (including soft-deleted)
- Cross-tenant access (when multi-tenant mode is enabled)
- Manage role definitions and permission lists

**Constraints:**
- Seeded only via artisan command (`php artisan crm:create-super-admin`)
- Mandatory 2FA
- Every action audit-logged
- Last Super Admin cannot be deleted or demoted
- Excluded from analytics and leaderboards

#### 4.2.2 Admin (CRM Administrator)
**Purpose:** Runs the company’s CRM day to day.

**Capabilities:**
- Manage users (except Super Admin)
- Configure pipelines, stages, lead sources, custom fields
- View all leads, customers, deals, tasks, and analytics
- Reassign leads and override ownership
- Set company-wide targets
- Access all reports and audit logs
- Manage notification templates

**Constraints:**
- Cannot manage Super Admin
- Cannot change system-level settings reserved for Super Admin
- Still bound by policies (no bypass)

#### 4.2.3 Sales Manager
**Purpose:** Responsible for a sales team’s performance.

**Capabilities:**
- View all leads/deals **for their team**
- Reassign leads within the team
- Set targets for sales reps
- View team analytics: win rate, pipeline value, rep leaderboard
- Create/edit tasks for team members
- View team communication history
- Approve deal stage changes (if approval workflow enabled)

**Constraints:**
- Cannot change system settings or manage user roles
- Cannot see other teams’ data unless explicitly permitted

#### 4.2.4 Sales Representative
**Purpose:** Frontline user who works leads daily.

**Capabilities:**
- Create and manage **their own** leads and deals
- Move deals through pipeline stages
- Log calls, emails, meetings, and notes
- Create and complete own follow-up tasks
- View own targets and performance
- View communication history for own accounts

**Constraints:**
- Cannot see other reps’ leads (unless shared)
- Cannot set targets, view company-wide analytics, or delete records permanently
- Cannot change pipeline settings

#### 4.2.5 Analyst / Viewer
**Purpose:** Management or reporting users who need insight without modifying data.

**Capabilities:**
- View dashboards, targets, pipeline reports, win/loss analytics
- Export reports (if permitted)
- View leads/deals in read-only mode

**Constraints:**
- Cannot create, edit, or delete any sales records
- Cannot log communications or reassign deals
- Cannot change targets

### 4.3 Permission Matrix

| Permission | Super Admin | Admin | Sales Manager | Sales Rep | Analyst |
|---|:---:|:---:|:---:|:---:|:---:|
| Manage Super Admins | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage Admins | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage users | ✅ | ✅ (except SA) | ❌ | ❌ | ❌ |
| System settings / integrations | ✅ | Limited | ❌ | ❌ | ❌ |
| Pipeline config | ✅ | ✅ | ❌ | ❌ | ❌ |
| View all leads | ✅ | ✅ | Team only | Own only | Read-only |
| Create/edit leads | ✅ | ✅ | ✅ | ✅ own | ❌ |
| Delete/restore records | ✅ | Limited | ❌ | ❌ | ❌ |
| Reassign leads | ✅ | ✅ | Team | ❌ | ❌ |
| Log communications | ✅ | ✅ | ✅ | ✅ own | ❌ |
| Create follow-up tasks | ✅ | ✅ | ✅ team | ✅ own | ❌ |
| Set targets | ✅ | ✅ | ✅ team | ❌ | ❌ |
| View analytics | ✅ all | ✅ all | ✅ team | ✅ own | ✅ read-only |
| Export reports | ✅ | ✅ | ✅ | ❌ | ✅ if allowed |
| Impersonate users | ✅ | ❌ | ❌ | ❌ | ❌ |
| Access logs / queues | ✅ | ❌ | ❌ | ❌ | ❌ |
| Cross-tenant access | ✅ | ❌ | ❌ | ❌ | ❌ |

### 4.4 Authorization Design Rules
1. **Two layers:** Permission (what action) + Data scope (which records).
2. **Data scopes:** `own` → `team` → `all`.
3. **Policies** enforce record-level checks (`LeadPolicy::update($user, $lead)`).
4. **Gate::before** grants Super Admin a universal bypass.
5. **Query scoping** by `owner_id`, `team_id`, or `manager_id` at the repository/query level.
6. **UI-level checks** (`@can`) mirror server-side checks — never as the only enforcement.

---

## 5. Features

### 5.1 Lead Tracking & Sales Pipelines

**Description**
Leads enter the system from various sources and are tracked through a configurable sales pipeline with multiple stages.

**Requirements**
- **FR-1.1** Create leads with: name, company, email, phone, source, owner, status, value, expected close date.
- **FR-1.2** Configurable lead sources (Website, Referral, Cold Call, Event, Other).
- **FR-1.3** Configurable pipelines with ordered stages (e.g., New → Contacted → Interested → Negotiating → Won/Lost).
- **FR-1.4** Kanban board view with drag-and-drop stage changes (permission-checked).
- **FR-1.5** List/table view with filters (owner, stage, source, value, date).
- **FR-1.6** Automatic stage-change history logging (who, when, from → to).
- **FR-1.7** Lead reassignment (Admin/Manager; audit-logged).
- **FR-1.8** Soft delete + restore (Admin+). Permanent delete (Super Admin only).
- **FR-1.9** CSV import/export (Admin+).
- **FR-1.10** Duplicate detection on email/phone at creation time.

### 5.2 Customer Communication History

**Description**
Every interaction with a lead or customer is logged against the record, creating an institutional memory.

**Requirements**
- **FR-2.1** Log interactions of type: Call, Email, Meeting, Note, Proposal, Other.
- **FR-2.2** Each entry stores: user, timestamp, type, subject, body/notes, outcome, optional attachments.
- **FR-2.3** Chronological timeline per lead/customer/deal.
- **FR-2.4** Filter timeline by type, user, and date range.
- **FR-2.5** Immutable history — edits allowed only within 15 minutes and audit-logged.
- **FR-2.6** Communication history preserved after reassignment (no data loss on handoff).
- **FR-2.7** Search across communication history (full-text on subject + body).

### 5.3 Follow-Up Reminders & Tasks

**Description**
The active workflow engine that prevents leads from going cold.

**Requirements**
- **FR-3.1** Create follow-up tasks tied to a lead/deal: title, due date/time, assignee, priority, notes.
- **FR-3.2** Task statuses: Pending, In Progress, Completed, Overdue, Cancelled.
- **FR-3.3** In-app + email + database notifications when a task is due or overdue.
- **FR-3.4** Scheduled command runs (e.g., every 15 min) to:
  - Flag tasks as overdue
  - Dispatch reminder notifications
  - Flag leads with no contact in N days (configurable, default 14)
- **FR-3.5** Automatic task creation triggers (rule-based):
  - On lead stage change (e.g., moving to "Interested" creates "Send case study" task)
  - On lead assignment (creates "Intro call" task)
  - On N days of inactivity
- **FR-3.6** Task list view: My Tasks, Team Tasks (Manager), All Tasks (Admin).
- **FR-3.7** Calendar view of upcoming follow-ups.

### 5.4 Sales Targets & Analytics

**Description**
Turns pipeline and communication data into measurable performance.

**Requirements**
- **FR-4.1** Set targets by period (monthly/quarterly) for individuals, teams, or company.
- **FR-4.2** Target types: revenue, deal count, call count.
- **FR-4.3** Real-time progress display vs. target.
- **FR-4.4** Dashboard metrics:
  - Total pipeline value (by stage)
  - Deals won/lost (count and value)
  - Win rate
  - Avg. deal cycle length
  - Lead source performance (leads → won)
  - Rep leaderboard (permission-scoped)
  - Tasks overdue count
- **FR-4.5** Dashboard APIs return aggregated JSON (see §7.5).
- **FR-4.6** Date range filtering on all analytics.
- **FR-4.7** Export reports to CSV/PDF (permission-checked).
- **FR-4.8** Super Admin excluded from all analytics and leaderboards.

---

## 6. Data Model (High Level)

### 6.1 Core Tables

| Table | Purpose |
|---|---|
| `users` | All users (all roles) |
| `roles` | Role definitions |
| `permissions` | Permission definitions |
| `role_has_permissions` | Many-to-many role ↔ permission |
| `model_has_roles` | User ↔ role assignment |
| `teams` | Sales teams (for scoping) |
| `team_user` | Team membership |
| `leads` | Lead records |
| `pipelines` | Pipeline definitions |
| `pipeline_stages` | Ordered stages per pipeline |
| `lead_stage_history` | Stage transitions (audit) |
| `lead_sources` | Configurable sources |
| `customers` | Converted customers (from won leads) |
| `communications` | Interaction log entries |
| `tasks` | Follow-up tasks |
| `targets` | Sales targets per user/team |
| `activities` | Global audit log |
| `notifications` | Laravel database notifications |

### 6.2 Key Relationships
- `users` ↔ `roles` (many-to-many)
- `users` ↔ `teams` (many-to-many)
- `leads.owner_id` → `users.id`
- `leads.pipeline_id` → `pipelines.id`
- `leads.stage_id` → `pipeline_stages.id`
- `communications.lead_id` → `leads.id`
- `tasks.lead_id` → `leads.id`, `tasks.assignee_id` → `users.id`
- `targets.user_id` / `targets.team_id`

### 6.3 Soft Deletes
- `leads`, `communications`, `tasks` → soft-deletable
- Restore: Admin+; Permanent delete: Super Admin only

---

## 7. Technical Architecture

### 7.1 Laravel Skills Mapping

| Laravel Skill | Role in the CRM |
|---|---|
| **Authentication** | Sanctum/Breeze/Fortify; per-user login; 2FA for Super Admin |
| **Permissions** | `spatie/laravel-permission` + policies + `Gate::before` bypass for Super Admin |
| **Scheduled Commands** | Kernel schedule: overdue task flagging, reminders, inactivity detection |
| **Notifications** | Mail + database + broadcast channels; queued |
| **Dashboard APIs** | Versioned REST API with aggregated endpoints consumed by the frontend |

### 7.2 Authentication
- Laravel Breeze or Fortify for web; Sanctum for API tokens.
- Super Admin: mandatory 2FA (Fortify two-factor).
- Session timeout for Super Admin (configurable, default 30 min).
- Password reset via signed email link.
- Force logout of a user by Super Admin (revokes sessions + tokens).

### 7.3 Authorization
- `spatie/laravel-permission` for roles and permissions.
- `Gate::before` in `AppServiceProvider`:

```php
Gate::before(function (User $user) {
    return $user->hasRole('super-admin') ? true : null;
});
```

- Policies per model: `LeadPolicy`, `CommunicationPolicy`, `TaskPolicy`, `TargetPolicy`.
- Data scoping via query scopes: `scopeOwnedBy`, `scopeForTeam`, `scopeVisibleTo($user)`.

### 7.4 Scheduled Commands

| Command | Frequency | Purpose |
|---|---|---|
| `crm:flag-overdue-tasks` | Every 15 min | Mark tasks past due as Overdue |
| `crm:send-task-reminders` | Every 15 min | Notify assignees of due/overdue tasks |
| `crm:flag-stalled-leads` | Daily 06:00 | Flag leads with no contact in N days |
| `crm:recalculate-targets` | Hourly | Refresh target progress aggregates |
| `crm:purge-expired-tokens` | Daily | Clean expired Sanctum tokens |

All commands implemented as queued jobs and monitored for failure.

### 7.5 Dashboard API Endpoints (v1)

```
GET  /api/v1/dashboard/summary
GET  /api/v1/dashboard/pipeline-value
GET  /api/v1/dashboard/win-rate
GET  /api/v1/dashboard/lead-source-performance
GET  /api/v1/dashboard/rep-leaderboard
GET  /api/v1/dashboard/target-progress
GET  /api/v1/dashboard/tasks-overdue
GET  /api/v1/analytics/deals?from=&to=
```

All endpoints:
- Require authentication (Sanctum)
- Enforce role + data scope
- Return aggregated JSON only (no raw row dumps)
- Cache aggregates (Redis) with short TTL (e.g., 60s) and invalidate on write

**Example response:**
```json
{
  "pipeline_value": {
    "New": 45000,
    "Contacted": 120000,
    "Interested": 87000,
    "Negotiating": 210000,
    "currency": "USD"
  },
  "as_of": "2025-01-15T10:30:00Z"
}
```

### 7.6 Notifications
- Channels: mail, database, broadcast (for in-app realtime).
- Queued via `ShouldQueue`.
- Templates configurable by Admin.
- Notification types: TaskDue, TaskOverdue, LeadAssigned, TargetReached, DealStageChanged.

### 7.7 Queue & Jobs
- Redis or database queue driver.
- Job types: notification dispatch, reminder batches, report generation, CSV imports.
- Failed jobs monitored via Horizon (recommended) or dashboard.

---

## 8. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Performance** | Dashboard APIs < 500 ms (cached); list views paginated; page load < 2 s |
| **Scalability** | Support up to 100k leads, 1M communications per tenant |
| **Security** | OWASP Top 10 compliant; CSRF, XSS, SQLi protected; passwords hashed (bcrypt/argon2) |
| **Auditability** | All Super Admin and Admin actions logged in `activities` |
| **Backup** | Daily DB backups, 30-day retention |
| **Availability** | 99.5% uptime target |
| **Accessibility** | WCAG 2.1 AA on core views |
| **Localization** | i18n-ready (timezone, currency, date formats) |

---

## 9. Security & Audit Requirements

- **2FA mandatory** for Super Admin.
- **Impersonation** only via signed, short-lived, audit-logged route.
- **Last Super Admin protection** — cannot be deleted or demoted.
- **Super Admin seed only via artisan** — no UI-based creation.
- **All role changes logged** in `activities` table.
- **Sensitive actions** (delete, restore, reassign, impersonate, settings change) always audit-logged with actor, target, timestamp, and IP.
- **Rate limiting** on login and API endpoints.
- **API tokens** scoped with abilities matching the user’s role.

---

## 10. User Stories (Sample)

| # | As a… | I want to… | So that… |
|---|---|---|---|
| US-01 | Sales Rep | See only my leads on the Kanban board | I can focus on my pipeline |
| US-02 | Sales Rep | Log a call with notes | My manager can see what happened |
| US-03 | Sales Rep | Get a reminder when a follow-up is due | I never miss a callback |
| US-04 | Sales Manager | View my team’s pipeline value by stage | I can forecast this quarter |
| US-05 | Sales Manager | Set a monthly target for a rep | Accountability is clear |
| US-06 | Admin | Configure a new pipeline stage | The CRM matches our sales process |
| US-07 | Admin | Reassign a lead to another rep | Deals don’t get stuck when someone is on leave |
| US-08 | Analyst | View win-rate trend over 6 months | I can report to leadership |
| US-09 | Super Admin | Impersonate a user to debug a reported issue | Support is fast and accurate |
| US-10 | Super Admin | Review all audit logs | I can investigate security incidents |

---

## 11. UI / UX Outline

### 11.1 Screens
- Login / 2FA
- Dashboard (role-scoped)
- Leads List
- Lead Detail (timeline, tasks, stage history)
- Pipeline Kanban
- Tasks (My / Team / All)
- Targets (list + progress)
- Analytics (charts, filters, export)
- Communications Timeline
- Admin: Users, Roles, Pipelines, Sources, Templates
- Super Admin: System Settings, Audit Log, Impersonation

### 11.2 UX Rules
- Kanban drag-and-drop only for permitted users (Server-side re-checked).
- Empty states guide the user to the next action.
- Analytics always show the current date range.
- Read-only views clearly marked for Analysts.
- Super Admin panel visually distinguished.

---

## 12. Release Plan

### Phase 1 — Foundation (Weeks 1–3)
- Auth, roles, permissions, Super Admin seed + bypass
- Users, teams, pipelines, lead sources
- Basic lead CRUD + list view

### Phase 2 — Pipeline & Communication (Weeks 4–6)
- Kanban board + stage history
- Communication log + timeline
- Lead assignment and reassignment

### Phase 3 — Workflow (Weeks 7–9)
- Tasks + reminders
- Scheduled commands + notifications
- Automatic task triggers

### Phase 4 — Analytics (Weeks 10–12)
- Targets (CRUD + progress)
- Dashboard APIs + caching
- Analytics screens + export

### Phase 5 — Hardening (Week 13+)
- Audit logs, impersonation, 2FA enforcement
- Performance tuning, tests, documentation
- UAT and launch

---

## 13. Acceptance Criteria (Definition of Done)

A feature is **Done** when:
- All functional requirements (§5) are implemented and tested.
- Role + data-scope checks verified on every endpoint and UI action.
- Super Admin bypass tested and audit-logged.
- Scheduled commands run on schedule and are monitored.
- Notifications dispatched via queue with retry.
- Dashboard APIs respond within the performance budget.
- Unit + feature tests cover permission boundaries.
- Documentation updated (API + user guide).

---

## 14. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Over-permissive Super Admin use | Security breach | 2FA, audit logs, seed-only creation, last-admin protection |
| Notification fatigue | Users ignore reminders | Configurable frequency, priority levels, digest mode |
| Analytics slow on large data | Poor UX | Caching, pre-aggregation, indexed queries |
| Scope-creep (inbox sync, telephony) | Delays v1 | Strict in/out of scope (§3) |
| Role confusion | Access bugs | Single permission matrix (§4.3) and tests |

---

## 15. Open Questions

1. Multi-tenant (SaaS) mode in v2 — confirmed? If yes, Super Admin becomes a **platform-level** user separate from tenant Admins.
2. Which notification channels for v1 — mail + database only, or add broadcast?
3. Approval workflow for deal stage changes — required for Manager role?
4. Default inactivity threshold for stalled leads — 14 days acceptable?
5. Target periods — monthly and quarterly only, or add annual?

---

**End of PRD**