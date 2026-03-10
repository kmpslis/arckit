# Project Requirements: Task Management Portal

> **Template Origin**: Official | **ArcKit Version**: 4.1.1 | **Command**: `/arckit:requirements`

## Document Control

| Field | Value |
|-------|-------|
| **Document ID** | ARC-001-REQ-v1.1 |
| **Document Type** | Business and Technical Requirements |
| **Project** | Task Management Portal (Project 001) |
| **Classification** | PUBLIC |
| **Status** | DRAFT |
| **Version** | 1.1 |
| **Created Date** | 2026-03-10 |
| **Last Modified** | 2026-03-10 |
| **Review Cycle** | Monthly |
| **Next Review Date** | 2026-04-10 |
| **Owner** | Jane Smith, Head of Engineering |
| **Reviewed By** | PENDING |
| **Approved By** | PENDING |
| **Distribution** | Engineering, Product, Leadership, and Operations Teams |

## Revision History

| Version | Date | Author | Changes | Approved By | Approval Date |
|---------|------|--------|---------|-------------|---------------|
| 1.0 | 2026-03-10 | ArcKit AI | Initial creation from `/arckit:requirements` command | PENDING | PENDING |
| 1.1 | 2026-03-10 | ArcKit AI | Added UC-5 (sub-task creation and management), FR-023 to FR-026, updated DR-003 Task entity, new DR-008a Sub-Task entity | PENDING | PENDING |

## Document Purpose

This document defines comprehensive business, functional, non-functional, integration, and data requirements for Quento1's Task Management Portal. It is derived from stakeholder drivers analysis (ARC-001-STKE-v1.0) and architecture principles (ARC-000-PRIN-v1.0). It will be used to guide vendor evaluation, architecture design, backlog creation, and acceptance testing.

---

## Executive Summary

### Business Context

Quento1 is building a Task Management Portal as its primary revenue-generating SaaS product. The portal must enable teams to create, assign, track, and collaborate on tasks within a shared workspace. The market includes established players (Jira, Asana, Linear, Notion), and Quento1's differentiation strategy is superior performance and simplicity — a tool that is genuinely fast, intuitive, and easy to adopt without training.

The commercial imperative is to launch an MVP to 100 paying customers by 30 September 2026 and grow to £180K ARR within 18 months. This requires a disciplined MVP scope, a solid architectural foundation aligned to ARC-000-PRIN-v1.0, and a product iteration cadence that responds quickly to user feedback.

The system must be built to Quento1's architecture standards from day one: scalable, secure by design, fully observable, infrastructure-as-code, and continuously deployed. Cutting corners on these foundations to accelerate short-term delivery would undermine long-term velocity and operational stability.

### Objectives

- Deliver an MVP Task Management Portal to production by 30 September 2026 with a paying beta cohort of 100+ customers
- Achieve p95 response times below 500ms for all core user actions under normal load
- Reach 99.5% monthly uptime from launch
- Achieve NPS of 40+ within 12 months of launch
- Keep infrastructure costs below 20% of MRR at all customer volume milestones

### Expected Outcomes

- £180K ARR within 18 months of launch (O-1, driven by SD-1, SD-3)
- Zero architecture principle violations at production launch (O-2, driven by SD-2)
- NPS 40+ and 65%+ Monthly Active User ratio by Month 12 (O-3, driven by SD-7)

### Project Scope

**In Scope**:

- User registration, authentication, and workspace management
- Task creation, editing, assignment, prioritization, and status tracking (including sub-tasks)
- Project/board management with list and kanban views
- Commenting and activity history on tasks
- In-app and email notification system
- Personal and team dashboards with progress metrics
- Data export (CSV and JSON)
- Workspace admin panel for user and billing management
- RESTful API for all portal functions
- Integration with email delivery, OAuth identity providers, and payment processing
- Web application (responsive, desktop-first)

**Out of Scope**:

- Native mobile application (iOS/Android) — deferred to Phase 2
- Advanced reporting and business intelligence dashboards — Phase 2
- Time tracking — Phase 2
- Two-way external integrations (Slack, GitHub, Jira) — Phase 2
- Custom workflow automation and rule builder — Phase 2
- Offline mode — not planned

---

## Stakeholders

| Stakeholder | Role | Organization | Involvement Level |
|-------------|------|--------------|-------------------|
| CEO / Founder | Executive Sponsor | Quento1 Leadership | Decision maker — commercial requirements |
| Jane Smith | Head of Engineering / Enterprise Architect | Quento1 Engineering | Technical oversight — architecture, NFR sign-off |
| Product Manager | Product Owner | Quento1 Product | Requirements definition and prioritization |
| Engineering Team | Delivery Team | Quento1 Engineering | Implementation, technical input |
| Operations / DevOps | Operations | Quento1 Engineering | Operational requirements, deployment |
| Finance | Budget Controller | Quento1 Finance | Cost and ROI requirements |
| End Users / Customers | User Representative | Customer Organizations | User acceptance, usability requirements |

---

## Business Requirements

### BR-001: Launch MVP to 100 Paying Customers by 30 September 2026

**Description**: Quento1 must launch a production-ready MVP of the Task Management Portal and acquire 100 paying customers by 30 September 2026.

**Rationale**: Commercial validation before runway expires. The CEO (SD-1) has identified September 2026 as the critical milestone for demonstrating product-market fit. Delay beyond this date risks missing the funding window. Traces to Goal G-1.

**Success Criteria**:

- Production system live and available to all paying customers by 30 September 2026
- 100 paying customers with active subscriptions on or before 30 September 2026
- Monthly churn rate below 3% in the first 3 months post-launch
- Beta programme running from 30 June 2026 with at least 10 paying pilot customers

**Priority**: MUST_HAVE

**Stakeholder**: CEO / Founder (SD-1), Product Manager (SD-3)

---

### BR-002: Generate £180K ARR Within 18 Months of Launch

**Description**: The portal must be commercially viable, generating £180,000 Annual Recurring Revenue within 18 months of production launch.

**Rationale**: Financial viability and investor confidence. Traces to Outcome O-1. Finance (SD-5) requires a demonstrable ROI within a defined timeframe. £180K ARR represents approximately 500 customers at £30/month or 250 customers at £60/month.

**Success Criteria**:

- MRR of £15,000 by end of Month 6 post-launch (100 customers)
- MRR of £37,500 by end of Month 12 (250 customers)
- MRR of £75,000 by end of Month 18 (500 customers)
- Net Revenue Retention above 100% by Month 12

**Priority**: MUST_HAVE

**Stakeholder**: CEO / Founder (SD-1), Finance (SD-5)

---

### BR-003: Infrastructure Costs Below 20% of MRR at All Scale Milestones

**Description**: Cloud infrastructure and hosting costs must not exceed 20% of Monthly Recurring Revenue at any customer volume milestone between 100 and 1,000 customers.

**Rationale**: SaaS unit economics require sub-linear infrastructure cost scaling to maintain healthy gross margins. Finance (SD-5) requires this for ROI validation. Traces to Goal G-5 and Architecture Principle 11 (Performance and Efficiency).

**Success Criteria**:

- Infrastructure cost below £3,000/month at 100 customers (assuming £15K MRR = 20%)
- Infrastructure cost below £7,500/month at 250 customers
- Cost per active customer declining month-on-month as scale increases
- Monthly cloud cost report generated and reviewed by Finance and Engineering

**Priority**: MUST_HAVE

**Stakeholder**: Finance (SD-5), CTO Jane Smith (SD-2)

---

### BR-004: Achieve NPS of 40 or Above Within 12 Months of Launch

**Description**: The Task Management Portal must achieve a Net Promoter Score of 40 or above, measured via quarterly surveys of active customers, within 12 months of production launch.

**Rationale**: NPS is the leading indicator of retention and organic growth. Product Manager (SD-3) and End Users (SD-7) require a product that genuinely improves their workflow. NPS 40 is the "good" SaaS threshold and signals product-market fit. Traces to Goal G-4 and Outcome O-3.

**Success Criteria**:

- NPS of 20+ by Month 6 (first survey post-beta)
- NPS of 40+ by Month 12 (second survey)
- Monthly Active User rate of 65%+ of registered users by Month 12
- Support ticket volume below 0.1 per active user per month by Month 6

**Priority**: MUST_HAVE

**Stakeholder**: CEO / Founder (SD-1), Product Manager (SD-3), End Users (SD-7)

---

## Functional Requirements

### User Personas

#### Persona 1: Team Member (Individual Contributor)

- **Role**: Software developer, designer, marketing executive, or any knowledge worker using the portal day-to-day
- **Goals**: Create and track their own tasks, see what's assigned to them, update status quickly, collaborate with colleagues via comments
- **Pain Points**: Existing tools (Jira, Asana) are too complex; Notion is flexible but unstructured; spreadsheets have no workflow
- **Technical Proficiency**: Medium — comfortable with SaaS tools but does not want configuration overhead

#### Persona 2: Team Lead / Project Manager

- **Role**: Engineering lead, product manager, or operations manager responsible for a team or project
- **Goals**: View overall project progress, assign and reprioritize tasks, track blockers, report status to leadership
- **Pain Points**: Context switching between tools; difficulty getting visibility across multiple projects; reporting takes too long
- **Technical Proficiency**: Medium to High

#### Persona 3: Workspace Administrator

- **Role**: IT administrator, engineering lead, or founder who manages the Quento1 subscription and workspace
- **Goals**: Manage user accounts, control billing, set workspace-level permissions, onboard new team members
- **Pain Points**: Managing user access across multiple tools; understanding spend; onboarding friction
- **Technical Proficiency**: High

---

### Use Cases

#### UC-1: Create and Assign a Task

**Actor**: Team Member (Persona 1)

**Preconditions**:

- User is authenticated and has at least Member role in a Project
- At least one Project exists in the Workspace

**Main Flow**:

1. User navigates to a Project and selects "New Task"
2. System opens a task creation form
3. User enters a title (required), description (optional), priority, due date, and assignee
4. User submits the form
5. System creates the task, assigns it to the selected user, and adds it to the project backlog
6. System sends an in-app notification to the assigned user

**Postconditions**:

- Task record created with status "To Do"
- Assignee receives notification
- Task appears in assignee's personal dashboard

**Alternative Flows**:

- **Alt 1a**: If user does not specify an assignee, task is created unassigned
- **Alt 2a**: If user assigns the task to themselves, no notification is sent

**Exception Flows**:

- **Ex 1**: If title is missing, system displays validation error and prevents submission
- **Ex 2**: If due date is in the past, system warns but allows submission

**Business Rules**:

- Task title is mandatory, max 500 characters
- Priority values: Critical, High, Medium, Low (default: Medium)
- Due date is optional but must be a future date if provided

**Priority**: CRITICAL

---

#### UC-2: Update Task Status

**Actor**: Team Member (Persona 1), Team Lead (Persona 2)

**Preconditions**:

- User is authenticated and has at least Member role in the Project containing the task
- Task exists and is not archived

**Main Flow**:

1. User opens the task detail view or uses the Kanban board
2. User changes the task status (To Do → In Progress → Done)
3. System updates the status and timestamps the change
4. System appends a status change entry to the task activity log
5. System notifies the task creator if they are not the same user

**Postconditions**:

- Task status updated
- Activity log entry added with timestamp and actor
- Creator notified if different user

**Business Rules**:

- Status transitions are freeform (any status to any status)
- Archiving a task requires Done status or explicit archive action
- Only Workspace Admins and Team Leads can archive tasks in bulk

**Priority**: CRITICAL

---

#### UC-3: Invite a Team Member to a Workspace

**Actor**: Workspace Administrator (Persona 3)

**Preconditions**:

- User is authenticated with Admin role
- Workspace is on an active paid subscription

**Main Flow**:

1. Admin navigates to Workspace Settings > Members
2. Admin enters email address and selects a role (Admin, Member, Viewer)
3. System sends an invitation email to the specified address
4. Invited user clicks the link, creates an account (or logs in), and accepts the invitation
5. System adds the user to the Workspace with the assigned role

**Postconditions**:

- New user is a member of the Workspace
- Invitation link expires after 7 days
- Admin receives confirmation of successful join

**Exception Flows**:

- **Ex 1**: Invitation email not delivered — admin can resend; link expires in 7 days
- **Ex 2**: Invitee already has an account — they are added directly without creating a new account

**Business Rules**:

- Workspace Admin can invite unlimited members (subject to subscription tier)
- Invitation links are single-use and expire after 7 days

**Priority**: HIGH

---

#### UC-4: View Personal Dashboard

**Actor**: Team Member (Persona 1)

**Preconditions**:

- User is authenticated
- User has at least one task assigned to them or has created tasks

**Main Flow**:

1. User navigates to the Home / Dashboard view
2. System renders the user's assigned tasks, grouped by status and sorted by due date
3. User can filter by Project, Priority, or Due Date range
4. User selects a task to open the detail view

**Postconditions**:

- No data is modified; this is a read-only view

**Priority**: HIGH

---

#### UC-5: Create and Manage Sub-Tasks

**Actor**: Team Member (Persona 1), Team Lead (Persona 2)

**Preconditions**:

- User is authenticated with Member or Admin role in the Project
- A parent Task exists and is not archived or deleted
- Sub-task nesting depth does not already equal the maximum (2 levels)

**Main Flow**:

1. User opens a parent Task detail view
2. User selects "Add Sub-task" in the Sub-tasks section
3. System opens an inline sub-task creation form within the parent task view
4. User enters a title (required) and optionally sets assignee, priority, and due date
5. User submits the form
6. System creates the sub-task linked to the parent task, applies status "To Do", and appends a creation entry to the parent task's activity log
7. System updates the parent task's sub-task progress indicator (e.g., "0 / 1 complete")

**Postconditions**:

- Sub-task record created with parent_task_id referencing the parent
- Parent task's sub-task completion count is updated
- If a sub-task is assigned, the assignee receives an in-app notification

**Alternative Flows**:

- **Alt 1a**: If the user does not set an assignee, the sub-task is created unassigned
- **Alt 2a**: Sub-task status can be updated directly from the parent task's sub-task list without opening the sub-task detail view

**Exception Flows**:

- **Ex 1**: If the parent task is archived, the "Add Sub-task" control is hidden and the action is denied if attempted via API
- **Ex 2**: If nesting depth would exceed 2 levels (sub-tasks of sub-tasks are not permitted), the system returns a validation error: "Sub-tasks cannot be nested more than one level below a parent task"

**Business Rules**:

- Maximum sub-task nesting depth: 1 level below a parent task (sub-tasks cannot have their own sub-tasks)
- A parent task may have an unlimited number of sub-tasks
- Deleting or archiving a parent task cascades to all its sub-tasks (soft archive)
- Completing all sub-tasks does NOT automatically mark the parent task as Done — the parent must be updated explicitly
- Sub-tasks inherit the parent's Project and Workspace membership; they are not independently movable to a different Project

**Priority**: HIGH

---

### Functional Requirements Detail

#### FR-001: User Registration

**Description**: New users must be able to create a Quento1 account using email and password, or via OAuth with Google or Microsoft.

**Relates To**: BR-001, UC-3

**Acceptance Criteria**:

- [ ] Given an unauthenticated user, when they submit a valid email and password, then an account is created, a verification email is sent, and they are redirected to the onboarding flow
- [ ] Given an unverified email, when the user attempts to create a task, then the system prompts them to verify first
- [ ] Given a user who clicks "Sign in with Google", then they are redirected to Google OAuth and authenticated without a password
- [ ] Given a duplicate email, when registration is attempted, then the system shows an informative error without revealing whether the email is registered (security)

**Data Requirements**:

- **Inputs**: Email, password (bcrypt-hashed min 12 chars), or OAuth token
- **Outputs**: User account record, session token
- **Validations**: Valid email format; password complexity (min 12 chars, uppercase, number, symbol)

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: NFR-SEC-001, INT-002

**Assumptions**: Google and Microsoft OAuth apps registered before Sprint 2

---

#### FR-002: User Authentication and Session Management

**Description**: Authenticated sessions must be managed securely with configurable timeouts and support for multi-factor authentication (MFA) for Admin users.

**Relates To**: BR-001, NFR-SEC-001

**Acceptance Criteria**:

- [ ] Given an authenticated session, when it is inactive for 60 minutes, then the session expires and the user is redirected to login
- [ ] Given an Admin user at login, when MFA is enabled, then they must complete TOTP verification before accessing the application
- [ ] Given a valid refresh token, when it has not expired (7 days), then the session is renewed without requiring re-login

**Data Requirements**:

- **Inputs**: Email, password, optional TOTP code
- **Outputs**: JWT access token (15-minute TTL), refresh token (7-day TTL)
- **Validations**: TOTP code valid within 30-second window

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-001, NFR-SEC-001, NFR-SEC-002

---

#### FR-003: User Profile Management

**Description**: Users must be able to view and update their profile including display name, avatar, notification preferences, and password.

**Relates To**: BR-004

**Acceptance Criteria**:

- [ ] Given an authenticated user, when they update their display name, then the change is reflected immediately across all task assignments and comments
- [ ] Given an authenticated user, when they upload a new avatar (JPEG/PNG, max 2MB), then it is resized to 128x128 and stored
- [ ] Given an authenticated user, when they change their password, then they must provide their current password and the new password must meet complexity rules
- [ ] Given an authenticated user, when they configure notification preferences, then email notifications are sent only for selected event types

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-001, FR-002

---

#### FR-004: Workspace Management

**Description**: Users must be able to create a Workspace (representing their organisation or team), with a unique subdomain or slug, and manage its settings.

**Relates To**: BR-001, BR-002

**Acceptance Criteria**:

- [ ] Given a new user completing onboarding, when they provide a workspace name, then a workspace is created with a URL-safe slug and they become its Admin
- [ ] Given a Workspace Admin, when they update workspace name or logo, then changes are reflected immediately for all members
- [ ] Given a workspace slug, when it conflicts with an existing slug, then the system suggests an alternative

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-001, INT-003

---

#### FR-005: Role-Based Access Control Within Workspace

**Description**: The system must enforce three roles: Admin (full control), Member (create/edit tasks and projects), and Viewer (read-only).

**Relates To**: NFR-SEC-002

**Acceptance Criteria**:

- [ ] Given a Viewer role, when the user attempts to create or edit a task, then the system returns a 403 error and the UI hides creation controls
- [ ] Given a Member role, when they attempt to access Workspace Settings, then access is denied
- [ ] Given an Admin, when they change a Member's role to Viewer, then that user's write access is immediately revoked (no session restart required)

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-001, NFR-SEC-002

---

#### FR-006: Task Creation

**Description**: Authenticated users with Member or Admin role must be able to create tasks within a Project, specifying title, description, priority, due date, and assignee.

**Relates To**: UC-1, BR-004

**Acceptance Criteria**:

- [ ] Given a Member in a Project, when they submit a task with a title, then the task is created with status "To Do" and appears in the project task list within 500ms
- [ ] Given a task creation, when the user assigns it to another Member, then an in-app notification is sent to the assignee
- [ ] Given a task with a due date set to today or earlier, when submitted, then the system shows a warning but allows creation

**Data Requirements**:

- **Inputs**: Title (required, max 500 chars), description (optional, Markdown, max 50,000 chars), priority enum, due date (optional), assignee user ID (optional)
- **Outputs**: Task record with generated UUID, created_at timestamp, created_by user ID

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-004, FR-005

---

#### FR-007: Task Editing

**Description**: Task creators, task assignees, and Admins must be able to edit all task fields. Members who are neither creator nor assignee may edit only comments.

**Relates To**: UC-1, UC-2

**Acceptance Criteria**:

- [ ] Given a task creator, when they update the title, then the change is persisted and an activity log entry is created
- [ ] Given a Member who is not the creator or assignee, when they attempt to edit the task title, then the edit controls are disabled
- [ ] Given any field change, when saved, then the activity log records: field changed, old value, new value, timestamp, actor

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-006

---

#### FR-008: Task Status Workflow

**Description**: Tasks must support a configurable status workflow with at least four states: To Do, In Progress, In Review, Done. Status transitions must be unrestricted.

**Relates To**: UC-2

**Acceptance Criteria**:

- [ ] Given a task in any status, when a Member moves it to a new status (drag-and-drop on Kanban or dropdown), then the change is saved within 300ms and reflected for all users viewing the project
- [ ] Given a status change, when the task creator is a different user, then a notification is sent to the creator
- [ ] Given a task moved to Done, when it has a due date that has passed, then it is marked as completed-late in the activity log

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006

---

#### FR-009: Task Deletion and Archiving

**Description**: Admins and task creators may delete tasks (hard delete, permanent) or archive them (soft delete, recoverable for 30 days).

**Relates To**: DR-010

**Acceptance Criteria**:

- [ ] Given an Admin or task creator, when they archive a task, then it is hidden from all project views but recoverable from the Archive section for 30 days
- [ ] Given an Admin, when they permanently delete a task, then all associated comments, attachments, and activity logs are deleted within 24 hours (background job)
- [ ] Given a non-creator, non-admin Member, when they attempt to delete or archive a task, then the action is denied with a clear error message

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-006, FR-005

---

#### FR-010: Task Search and Filtering

**Description**: Users must be able to search tasks by title/description keyword and filter by project, assignee, priority, status, due date range, and label.

**Relates To**: BR-004

**Acceptance Criteria**:

- [ ] Given a search query of 3+ characters, when entered, then results appear within 500ms showing tasks matching the query across all Projects the user has access to
- [ ] Given applied filters, when navigating away and returning, then filters are preserved for the session
- [ ] Given a search with no results, then a helpful empty state is shown with suggestions

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006

---

#### FR-011: Task Comments and Mentions

**Description**: Members must be able to add comments to tasks using Markdown formatting, mention other workspace members using @username, and receive notifications when mentioned.

**Relates To**: UC-1, BR-004

**Acceptance Criteria**:

- [ ] Given a Member adding a comment, when they submit it, then it appears in the task activity feed within 500ms for all users viewing the task
- [ ] Given an @mention in a comment, when the comment is submitted, then the mentioned user receives an in-app and email notification
- [ ] Given a comment author, when they edit or delete their own comment within 10 minutes of posting, then the action is permitted; after 10 minutes, editing is disabled

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006

---

#### FR-012: Task Activity Log

**Description**: Every task must maintain an immutable chronological activity log recording all changes: field edits, status changes, assignments, comments, and file attachments.

**Relates To**: NFR-C-002

**Acceptance Criteria**:

- [ ] Given any change to a task, when it is saved, then an activity log entry is created with: actor, action type, old value, new value, timestamp (UTC, millisecond precision)
- [ ] Given a user viewing the activity log, then entries are shown in reverse chronological order
- [ ] Given an Admin attempting to delete an activity log entry, then the action is denied (immutable)

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-006

---

#### FR-013: Project Management

**Description**: Members and Admins must be able to create Projects within a Workspace, each with a name, description, and optional icon. Projects are the primary organisational container for tasks.

**Relates To**: UC-1, BR-001

**Acceptance Criteria**:

- [ ] Given an Admin or Member, when they create a project with a name, then it appears immediately in the Workspace project list
- [ ] Given a project, when it has no tasks, then a clear empty state guides the user to create their first task
- [ ] Given an Admin, when they archive a project, then all its tasks are archived and it is hidden from the default project list

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-004

---

#### FR-014: Project Views — List and Kanban

**Description**: Every project must support two views: a List view (sorted, filterable table of tasks) and a Kanban board (tasks grouped by status in columns, draggable).

**Relates To**: UC-2, BR-004

**Acceptance Criteria**:

- [ ] Given a project with tasks, when the user switches between List and Kanban views, then the transition completes within 300ms
- [ ] Given the Kanban view, when a user drags a task to a new status column, then the task status is updated within 300ms and reflected for all users viewing the project
- [ ] Given the List view, when sorted by due date, then overdue tasks are highlighted

**Priority**: MUST_HAVE

**Complexity**: HIGH

**Dependencies**: FR-006, FR-008

---

#### FR-015: Project Member Management

**Description**: Project Admins must be able to add or remove Workspace Members from a specific Project, controlling which members can see and interact with that project's tasks.

**Relates To**: FR-005

**Acceptance Criteria**:

- [ ] Given a Project Admin, when they add a Workspace Member to the project, then that Member immediately gains access to all project tasks
- [ ] Given a Project Admin, when they remove a Member from the project, then that Member loses access immediately (no session restart required)
- [ ] Given a Workspace Admin, then they have implicit access to all Projects in the Workspace

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-005, FR-013

---

#### FR-016: Personal Dashboard

**Description**: Each authenticated user must have a personal dashboard showing: tasks assigned to them (sorted by due date and priority), tasks they created, and recent activity across their projects.

**Relates To**: UC-4, BR-004

**Acceptance Criteria**:

- [ ] Given an authenticated user, when they navigate to the dashboard, then it loads within 500ms and shows assigned tasks, grouped by status, sorted by due date ascending
- [ ] Given an overdue task, then it is visually highlighted with a red indicator
- [ ] Given a user with no assigned tasks, then the dashboard shows a helpful empty state

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006

---

#### FR-017: In-App Notifications

**Description**: The system must deliver real-time in-app notifications for: task assignment, task status change (creator notified), @mentions in comments, task due date reminders (24 hours before), and workspace invitations.

**Relates To**: UC-1, FR-011

**Acceptance Criteria**:

- [ ] Given a new notification, when the user is logged in, then it appears in the notification bell within 5 seconds of the triggering event (no page refresh required)
- [ ] Given a notification, when the user clicks it, then they are navigated to the relevant task or workspace context
- [ ] Given a user who marks all notifications as read, then the unread count resets to zero immediately

**Priority**: MUST_HAVE

**Complexity**: HIGH

**Dependencies**: FR-006, FR-011, INT-001

---

#### FR-018: Email Notifications

**Description**: The system must send transactional email notifications for the same events as in-app notifications, with user-configurable opt-outs per event type.

**Relates To**: FR-017, INT-001

**Acceptance Criteria**:

- [ ] Given a user who disables email notifications for task assignments, when a task is assigned to them, then no email is sent but the in-app notification still appears
- [ ] Given a triggered notification email, when sent, then it is delivered within 60 seconds of the triggering event
- [ ] Given any notification email, it must include an unsubscribe link that immediately disables all email notifications for that user

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-017, INT-001

---

#### FR-019: Data Export

**Description**: Workspace Admins and Members must be able to export all tasks in a Project (or their personal tasks) to CSV or JSON format.

**Relates To**: NFR-C-001 (GDPR data portability)

**Acceptance Criteria**:

- [ ] Given a Project Admin, when they request a project export, then a CSV or JSON file is generated and available for download within 60 seconds for projects with up to 10,000 tasks
- [ ] Given a Member, when they request an export of their own tasks across all projects, then only tasks they created or are assigned to are included
- [ ] Given an export request, the exported file includes: task ID, title, description, status, priority, assignee, creator, due date, created date, project name

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-006, NFR-C-001

---

#### FR-020: Workspace Admin Panel — User Management

**Description**: Workspace Admins must have a dedicated admin panel to manage members: view all members, change roles, remove members, and view pending invitations.

**Relates To**: UC-3, FR-005

**Acceptance Criteria**:

- [ ] Given a Workspace Admin, when they navigate to Settings > Members, then they see a list of all active members and pending invitations with their roles and join dates
- [ ] Given an Admin, when they change a Member's role, then the change takes effect within 5 seconds without requiring the member to log out
- [ ] Given an Admin, when they remove a Member, then that user loses all workspace access immediately and their tasks become unassigned

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-005, FR-004

---

#### FR-021: Subscription and Billing Management

**Description**: Workspace Admins must be able to manage their Quento1 subscription: view current plan, upgrade/downgrade, update payment details, and download invoices — all via a self-service billing portal.

**Relates To**: BR-002, INT-003

**Acceptance Criteria**:

- [ ] Given a Workspace Admin, when they click "Manage Subscription", then they are redirected to a Stripe-hosted billing portal within 2 seconds
- [ ] Given a subscription cancellation, when confirmed, then access continues until the end of the paid period; no refunds are issued
- [ ] Given a failed payment, when it occurs, then the Admin receives an email and in-app notification; workspace access is suspended after a 7-day grace period

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: INT-003, FR-004

---

#### FR-022: Team and Project Progress Dashboard

**Description**: Team Leads and Admins must be able to view an aggregate project progress dashboard showing: task completion rate, overdue task count, tasks by assignee, and velocity (tasks completed per week).

**Relates To**: BR-004

**Acceptance Criteria**:

- [ ] Given a Project Admin or Team Lead, when they open the project dashboard, then it shows: total tasks, tasks by status (pie/bar chart), tasks by assignee, and overdue count — all loading within 500ms
- [ ] Given the velocity chart, when viewing the last 4 weeks, then it shows tasks completed per week with a trend line

**Priority**: SHOULD_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006, FR-013

---

#### FR-023: Sub-Task Creation

**Description**: Members and Admins with access to a Project must be able to create sub-tasks under any existing, non-archived parent task. Sub-tasks must support the same core fields as tasks: title, assignee, priority, and due date. Sub-tasks are limited to one level of nesting (no sub-tasks of sub-tasks).

**Relates To**: UC-5, FR-006, BR-004

**Acceptance Criteria**:

- [ ] Given a Member viewing a parent task, when they select "Add Sub-task" and submit a title, then a sub-task is created linked to the parent and appears in the sub-task list within 500ms
- [ ] Given a sub-task creation, when an assignee is specified, then an in-app notification is sent to that user
- [ ] Given a sub-task, when the user attempts to add a further sub-task beneath it (level 3), then the system returns a validation error: "Sub-tasks cannot be nested more than one level below a parent task"
- [ ] Given a parent task that is archived, when a user attempts to add a sub-task via the API, then the system returns HTTP 422 with error code PARENT_ARCHIVED
- [ ] Given a sub-task creation via API without a title, then the system returns HTTP 422 with field-level validation error

**Data Requirements**:

- **Inputs**: title (required, max 500 chars), parent_task_id (required), assignee_id (optional), priority (optional, default: Medium), due_date (optional)
- **Outputs**: Sub-task record (DR-008a) with parent_task_id set, created_at timestamp
- **Validations**: parent_task_id must reference an active (non-archived, non-deleted) task in the same Workspace; nesting depth must be exactly 1

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-006, FR-005, UC-5, DR-008a

**Assumptions**: Sub-tasks share the same Project and Workspace as their parent; cross-project sub-tasks are not supported

---

#### FR-024: Sub-Task Status Management

**Description**: Sub-task status must be independently manageable from the parent task. Each sub-task follows the same status workflow as parent tasks (To Do, In Progress, In Review, Done). Status changes on sub-tasks must be reflected in the parent task's completion progress indicator.

**Relates To**: UC-5, FR-008

**Acceptance Criteria**:

- [ ] Given a sub-task in any status, when a Member changes its status, then the change is persisted within 300ms and reflected immediately in the parent task's sub-task progress indicator (e.g., "2 / 3 complete")
- [ ] Given all sub-tasks of a parent being moved to Done, when this occurs, then the parent task is NOT automatically moved to Done — a banner is shown: "All sub-tasks are complete. Mark parent task as Done?"
- [ ] Given a sub-task status change, when it is saved, then an activity log entry is appended to the sub-task's own log AND a summary entry (e.g., "Sub-task 'X' marked Done") is appended to the parent task's activity log

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-023, FR-008, FR-012

---

#### FR-025: Sub-Task Assignment and Notifications

**Description**: Sub-tasks must support independent assignment to any Workspace Member, separate from the parent task's assignee. Notification rules for sub-tasks mirror those for parent tasks.

**Relates To**: UC-5, FR-017, FR-018

**Acceptance Criteria**:

- [ ] Given a sub-task with an assignee, when the assignee is changed to a different user, then the new assignee receives an in-app and email notification (subject to their notification preferences)
- [ ] Given a sub-task due date that is 24 hours away, when the nightly reminder job runs, then the assignee receives a due-date reminder notification identical in format to parent task reminders
- [ ] Given a sub-task with no assignee, when a Member views the parent task, then the sub-task is displayed with an "Unassigned" indicator

**Priority**: MUST_HAVE

**Complexity**: LOW

**Dependencies**: FR-023, FR-017, FR-018

---

#### FR-026: Sub-Task Progress Display on Parent Task

**Description**: Every parent task with at least one sub-task must display a progress indicator showing the count of completed sub-tasks versus total sub-tasks. This indicator must be visible in both the task detail view and the project list/Kanban views.

**Relates To**: UC-5, FR-014, BR-004

**Acceptance Criteria**:

- [ ] Given a parent task with 3 sub-tasks (1 Done, 2 To Do), when a user views the task card in Kanban or List view, then a progress indicator is shown: "1 / 3" with a mini progress bar
- [ ] Given a parent task with all sub-tasks in Done status, when viewed, then the progress indicator is shown in green with a checkmark to visually signal completion
- [ ] Given a parent task with no sub-tasks, then no progress indicator is shown (no empty "0 / 0" display)
- [ ] Given a sub-task status change, when it is saved, then the parent task's progress indicator updates within 500ms for all users currently viewing the parent task or the project board

**Priority**: MUST_HAVE

**Complexity**: MEDIUM

**Dependencies**: FR-023, FR-024, FR-014

---

## Non-Functional Requirements (NFRs)

### Performance Requirements

#### NFR-P-001: Core Action Response Time

**Requirement**: All core user actions (task creation, task update, task list load, dashboard render, search) MUST respond within 500ms at the 95th percentile (p95) under normal operating load (up to 500 concurrent users).

**Measurement Method**: APM tooling (Datadog / Grafana) for production; k6 load testing suite in CI/CD pipeline

**Load Conditions**:

- Peak load: 500 concurrent users
- Average load: 50 requests per second
- Data volume: 100,000 tasks in primary database at Year 1 load test

**Alignment**: Architecture Principle 11 (Performance and Efficiency), Stakeholder Goal G-3

**Priority**: CRITICAL

---

#### NFR-P-002: p99 Response Time

**Requirement**: p99 response time for core user actions MUST remain below 1,500ms under normal load (500 concurrent users).

**Measurement Method**: Same as NFR-P-001

**Priority**: HIGH

---

#### NFR-P-003: Page Load Time

**Requirement**: Initial application load (first meaningful paint) MUST complete within 2 seconds on a standard broadband connection (50 Mbps) for returning users with cached assets.

**Measurement Method**: Lighthouse CI in deployment pipeline; real user monitoring (RUM) in production

**Priority**: HIGH

**Alignment**: SD-7 (User Productivity), G-3

---

#### NFR-P-004: API Throughput

**Requirement**: The API layer MUST sustain 200 requests per second across all endpoints combined during peak load, with an error rate below 0.1%.

**Measurement Method**: k6 load test in CI/CD; production monitoring dashboards

**Priority**: HIGH

---

### Availability and Resilience Requirements

#### NFR-A-001: Monthly Uptime SLA

**Requirement**: The Task Management Portal MUST achieve 99.5% monthly uptime (maximum 3.6 hours downtime per rolling 30-day period) from the production launch date.

**Maintenance Windows**: Planned maintenance is permitted between 02:00-04:00 UTC on Sundays, notified 48 hours in advance. Planned downtime counts toward the SLA.

**Alignment**: Architecture Principle 12 (Availability and Reliability), Goal G-2, SD-6, SD-7

**Priority**: CRITICAL

---

#### NFR-A-002: Recovery Objectives

**RPO (Recovery Point Objective)**: Maximum acceptable data loss = 1 hour (database point-in-time recovery to within 1 hour of failure).

**RTO (Recovery Time Objective)**: Maximum acceptable downtime per incident = 30 minutes.

**Backup Requirements**:

- Database: Continuous write-ahead log shipping + daily snapshot; 30-day retention
- Application state: Stateless; no application-level backup required
- Geographic backup: Backups stored in a different cloud region from production

**Alignment**: Architecture Principle 2 (Resilience and Fault Tolerance), Architecture Principle 12

**Priority**: CRITICAL

---

#### NFR-A-003: Fault Tolerance and Graceful Degradation

**Requirement**: The system MUST continue to operate in a degraded mode if non-critical dependencies (email delivery, analytics) fail. Task creation, editing, and viewing MUST remain functional if notification services are unavailable.

**Resilience Patterns Required**:

- [ ] Circuit breaker for all external dependencies (email, analytics, payment)
- [ ] Retry with exponential backoff and jitter for transient failures
- [ ] Timeout on all outbound network calls (max 5 seconds)
- [ ] Bulkhead isolation for notification processing (async queue; failure does not block task operations)
- [ ] Graceful degradation: notification failures are logged and retried, not surfaced as user errors

**Alignment**: Architecture Principle 2 (Resilience and Fault Tolerance), Architecture Principle 10 (Asynchronous Communication)

**Priority**: CRITICAL

---

#### NFR-A-004: Multi-Availability-Zone Deployment

**Requirement**: All production components (application servers, database, caching layer) MUST be deployed across a minimum of two cloud availability zones to eliminate single-AZ failure as a cause of full service outage.

**Alignment**: Architecture Principle 12 (Availability and Reliability)

**Priority**: CRITICAL

---

### Scalability Requirements

#### NFR-S-001: Horizontal Scaling

**Requirement**: The application tier MUST support horizontal scaling (adding compute instances) without code changes or downtime, to handle 10x the initial load without architectural rework.

**Growth Projections**:

- Year 1: 500 concurrent users, 50 req/s sustained, 100K tasks
- Year 2: 2,000 concurrent users, 200 req/s sustained, 500K tasks
- Year 3: 5,000 concurrent users, 500 req/s sustained, 2M tasks

**Scaling Triggers**: Auto-scale when CPU utilisation exceeds 70% or memory exceeds 80% for 3 consecutive minutes.

**Alignment**: Architecture Principle 1 (Scalability and Elasticity)

**Priority**: CRITICAL

---

#### NFR-S-002: Database Scaling and Data Volume

**Requirement**: The database MUST handle data growth to 2 million task records with associated comments and activity logs (estimated 50GB total) by Year 3, with no architectural change required.

**Data Archival Strategy**: Tasks archived for more than 12 months are eligible for cold storage migration. Activity logs older than 2 years are compressed and moved to archival storage. Hot data retention: 12 months in primary database.

**Alignment**: Architecture Principle 1, Principle 7 (Data Quality and Lineage)

**Priority**: HIGH

---

### Security Requirements

#### NFR-SEC-001: Authentication

**Requirement**: All users MUST authenticate via a secure mechanism before accessing any application resource. Supported methods: email/password (bcrypt, min 12 chars), Google OAuth 2.0, and Microsoft OAuth 2.0.

**Multi-Factor Authentication (MFA)**:

- Required for: Workspace Admin role (enforced, cannot be disabled by user)
- Optional for: Member and Viewer roles (user-configurable)
- MFA methods: TOTP (authenticator app, RFC 6238)

**Session Management**:

- Access token TTL: 15 minutes (JWT)
- Refresh token TTL: 7 days (rotating refresh token)
- Absolute session timeout: 30 days (requires full re-authentication)
- Re-authentication required for: billing changes, role changes, password changes

**Alignment**: Architecture Principle 4 (Security by Design — NON-NEGOTIABLE)

**Priority**: CRITICAL

---

#### NFR-SEC-002: Authorisation

**Requirement**: Role-based access control (RBAC) MUST enforce least-privilege access at the Workspace level (Admin, Member, Viewer) and Project level (Project Admin, Project Member). All authorization checks MUST be enforced server-side — client-side UI restrictions are supplementary only.

**Privilege Elevation**: No privilege escalation is permitted within the system. Role changes require an existing Admin action. No impersonation capability.

**Alignment**: Architecture Principle 4, Zero Trust Pillar: Least Privilege

**Priority**: CRITICAL

---

#### NFR-SEC-003: Data Encryption

**Requirement**:

- Data in transit: TLS 1.3+ enforced for all client-server and service-to-service communication; TLS 1.2 is acceptable for external integrations where 1.3 is not supported
- Data at rest: AES-256 encryption for all database storage, file storage, and backups
- Key management: Cloud-native KMS (AWS KMS or equivalent); keys rotated annually

**Encryption Scope**:

- [ ] Database encryption at rest (AES-256)
- [ ] Backup encryption (AES-256)
- [ ] File/object storage encryption (AES-256)
- [ ] TLS termination at load balancer; internal traffic encrypted between services

**Alignment**: Architecture Principle 4, Zero Trust Pillar: Encryption Everywhere

**Priority**: CRITICAL

---

#### NFR-SEC-004: Secrets Management

**Requirement**: No secrets (API keys, database credentials, OAuth client secrets, certificates) MAY exist in source code, configuration files, container images, or environment variable files committed to version control. All secrets MUST be stored in a dedicated secrets management service.

**Secrets Storage**: Cloud-native secrets manager (AWS Secrets Manager, GCP Secret Manager, or Azure Key Vault)

**Secrets Rotation**: Database credentials rotated every 90 days (automated); OAuth client secrets reviewed annually.

**Alignment**: Architecture Principle 4 (NON-NEGOTIABLE), Principle 14 (Infrastructure as Code)

**Priority**: CRITICAL

---

#### NFR-SEC-005: Vulnerability Management

**Requirement**:

- Dependency vulnerability scanning MUST run in every CI/CD pipeline execution; builds with Critical or High severity vulnerabilities MUST fail
- Static Application Security Testing (SAST) MUST run on every pull request
- Annual third-party penetration test by a qualified external security firm
- OWASP Top 10 mitigations MUST be addressed in the architecture and verified by security review before launch

**Remediation SLA**:

- Critical vulnerabilities: patched within 24 hours of disclosure
- High vulnerabilities: patched within 7 days
- Medium vulnerabilities: patched within 30 days

**Alignment**: Architecture Principle 4 (Security by Design), Principle 15 (Automated Testing)

**Priority**: CRITICAL

---

#### NFR-SEC-006: OWASP Top 10 Mitigations

**Requirement**: The system MUST be designed and tested against the OWASP Top 10 (2021). The following controls are mandatory:

- [ ] Injection prevention (parameterised queries, ORM usage; no raw SQL with user input)
- [ ] Broken authentication prevention (strong session management per NFR-SEC-001)
- [ ] Sensitive data exposure prevention (PII encrypted at rest and in transit)
- [ ] XML/JSON injection prevention (input validation and output encoding)
- [ ] Broken access control prevention (server-side RBAC checks on every endpoint per NFR-SEC-002)
- [ ] Security misconfiguration prevention (infrastructure hardening, default credentials removed)
- [ ] Cross-Site Scripting (XSS) prevention (Content Security Policy, output encoding)
- [ ] Insecure deserialization prevention (input validation on all API payloads)
- [ ] Logging and monitoring (per NFR-M-001)

**Alignment**: Architecture Principle 4

**Priority**: CRITICAL

---

### Compliance Requirements

#### NFR-C-001: GDPR Compliance

**Applicable Regulations**: UK GDPR (retained EU law post-Brexit), EU GDPR for EU-based customers.

**Compliance Requirements**:

- [ ] Right of access: users can request and receive a copy of their personal data within 30 days (automated export via FR-019)
- [ ] Right to erasure: users can request deletion of their account and personal data; deletion completed within 30 days; task content created by that user is anonymised, not deleted, to preserve project integrity
- [ ] Right to data portability: users can export their personal data in machine-readable format (JSON) — satisfied by FR-019
- [ ] Privacy by design: personal data collection is minimised to what is necessary for service operation
- [ ] Data breach notification: Quento1's Privacy Officer must be able to notify affected users within 72 hours of discovery
- [ ] Consent management: cookie consent banner on first visit; analytics cookies require opt-in

**Data Residency**: UK and EU customer data stored within UK/EU cloud regions. Data transfer to third-party processors (e.g. email provider, analytics) under Standard Contractual Clauses.

**Data Retention**: Account and personal data retained while subscription is active. Following cancellation: data retained for 90 days (recovery window), then permanently deleted.

**Alignment**: Architecture Principle 6 (Data Sovereignty and Governance)

**Priority**: CRITICAL

---

#### NFR-C-002: Audit Logging

**Requirement**: The system MUST maintain a comprehensive, tamper-evident audit log of all authentication events, authorization decisions, and sensitive data operations.

**Audit Log Contents**:

- Who: User ID and role
- What: Action type (login, logout, task create, role change, etc.)
- When: UTC timestamp with millisecond precision
- Where: Service name, endpoint
- Result: Success or failure with error code

**Log Retention**: Security and audit logs retained for 1 year in hot storage; 2 years in cold/archival storage (immutable, append-only).

**Log Integrity**: Structured logs written to append-only log stream; no log deletion permitted by application users or administrators.

**Alignment**: Architecture Principle 5 (Observability), Principle 4 (Security by Design)

**Priority**: CRITICAL

---

#### NFR-C-003: Cookie and Tracking Consent

**Requirement**: The application MUST implement a cookie consent mechanism that blocks analytics and non-essential tracking cookies until the user provides explicit opt-in consent, in compliance with UK PECR and GDPR.

**Requirements**:

- Cookie consent banner displayed on first visit
- Essential cookies (session, auth) exempt from consent
- Analytics and marketing cookies require explicit opt-in
- Consent preferences stored and respected on subsequent visits
- Users can revoke consent at any time via Privacy Settings

**Priority**: MUST_HAVE

---

### Usability Requirements

#### NFR-U-001: Time-to-First-Value

**Requirement**: A new user MUST be able to register, create their first workspace, and create their first task within 5 minutes, with no external help or documentation.

**Measurement Method**: Usability testing with 5 representative users before launch; task completion time recorded; target 5 minutes median

**Alignment**: SD-7 (User Productivity), G-4 (NPS)

**Priority**: MUST_HAVE

---

#### NFR-U-002: Accessibility

**Requirement**: The application MUST meet WCAG 2.1 Level AA across all core user workflows.

**Accessibility Features**:

- [ ] Full keyboard navigation for all interactive elements
- [ ] Screen reader compatibility (tested with NVDA and VoiceOver)
- [ ] Minimum contrast ratio 4.5:1 for normal text, 3:1 for large text
- [ ] Drag-and-drop interactions (Kanban) have a keyboard-accessible alternative
- [ ] Focus indicators visible on all interactive elements
- [ ] Error messages are programmatically associated with their form fields

**Testing**: Automated accessibility audit (axe-core) in CI/CD pipeline; manual NVDA/VoiceOver test before launch

**Priority**: MUST_HAVE

---

#### NFR-U-003: Responsive Design

**Requirement**: The application MUST be fully usable on desktop (1280px+), tablet (768-1279px), and mobile (375-767px) screen widths. All core workflows (task creation, status update, viewing dashboard) MUST be operable without horizontal scrolling on any supported viewport.

**Browser Support**: Chrome, Firefox, Safari, and Edge — last 2 major releases at time of launch.

**Priority**: MUST_HAVE

---

#### NFR-U-004: Localisation — English (UK) Primary

**Requirement**: The application MUST use British English spelling and date formats (DD/MM/YYYY) as the default. US English is not acceptable as the primary locale for a UK-founded product.

**Future Localisation**: Architecture MUST support i18n (string externalisation) to enable future language additions without code changes. Internationalisation scaffolding required from Sprint 1.

**Priority**: MUST_HAVE

---

### Maintainability and Observability Requirements

#### NFR-M-001: Observability — Logs, Metrics, Traces

**Requirement**: All services MUST emit structured telemetry enabling real-time monitoring, troubleshooting, and capacity planning, in compliance with Architecture Principle 5.

**Telemetry Requirements**:

- **Logging**: Structured JSON logs with correlation ID (request trace ID) on every log entry; log levels: DEBUG, INFO, WARN, ERROR, FATAL; centralized log aggregation
- **Metrics**: RED metrics per endpoint (Rate, Errors, Duration); resource utilisation (CPU, memory, I/O); business metrics (tasks created/day, active users); Prometheus-compatible format
- **Tracing**: Distributed tracing with OpenTelemetry; trace context propagated across all service calls; minimum 5% trace sampling in production, 100% on error
- **Dashboards**: Real-time operational dashboard for: uptime, p95 latency, error rate, task creation rate, active user count
- **Alerting**: SLO-based alerts for uptime and latency SLIs; PagerDuty or equivalent on-call escalation

**Log Retention** (aligned with ARC-000-PRIN-v1.0 Principle 5):

- Security/audit logs: 1 year hot, 2 years cold
- Application logs: 30 days
- Metrics: 13 months with rollup aggregation

**Alignment**: Architecture Principle 5 (Observability and Operational Excellence)

**Priority**: CRITICAL

---

#### NFR-M-002: API Documentation

**Requirement**: All API endpoints MUST have an OpenAPI 3.0 specification, generated from code annotations (not hand-authored). The spec MUST be available at `/api/docs` in all non-production environments.

**Alignment**: Architecture Principle 3 (Interoperability and Integration)

**Priority**: MUST_HAVE

---

#### NFR-M-003: Infrastructure as Code

**Requirement**: All cloud infrastructure (compute, database, networking, DNS, secrets, monitoring) MUST be defined as code in a declarative IaC framework (Terraform or Pulumi) and stored in version control. No manual infrastructure changes to production are permitted.

**Requirements**:

- [ ] All environments (dev, staging, production) reproducible from IaC alone
- [ ] Infrastructure changes go through the same PR review process as application code
- [ ] IaC linting and plan review run in CI/CD before apply

**Alignment**: Architecture Principle 14 (Infrastructure as Code — HIGH)

**Priority**: CRITICAL

---

#### NFR-M-004: Automated Test Coverage

**Requirement**: Core application modules MUST achieve and maintain a minimum of 80% automated test coverage (unit + integration) before merging to the main branch. End-to-end tests MUST cover all critical user journeys (UC-1 through UC-5).

**Test Pyramid Target**:

- Unit tests: 70% of test suite — fast, isolated, high coverage
- Integration tests: 25% — component interaction tests
- End-to-end tests: 5% — critical journeys only

**Alignment**: Architecture Principle 15 (Automated Testing), Conflict C-1 resolution

**Priority**: CRITICAL

---

#### NFR-M-005: CI/CD Pipeline

**Requirement**: All code changes MUST go through an automated CI/CD pipeline before deployment to any environment. Production deployments MUST require: all tests passing, dependency scan clean (no Critical/High), code review approval, and a successful staging deployment.

**Pipeline Stages**:

1. Lint and static analysis (including SAST)
2. Unit test execution
3. Integration test execution
4. Dependency vulnerability scan
5. Build and package
6. Deploy to staging
7. End-to-end test on staging
8. Production deployment (manual gate or auto on merge to main)

**Rollback**: Production rollback MUST be achievable within 10 minutes by any on-call engineer without code changes.

**Alignment**: Architecture Principle 16 (CI/CD)

**Priority**: CRITICAL

---

## Integration Requirements

### INT-001: Email Delivery Service

**Purpose**: Send transactional emails (invitations, notifications, password resets, billing alerts) reliably and with delivery tracking.

**Integration Type**: Outbound API (synchronous for transactional; async queue for batch notifications)

**Data Exchanged**:

- From Portal to Email Service: Recipient address, template ID, template variables, send priority
- From Email Service to Portal: Delivery status webhooks (delivered, bounced, spam-reported)

**Integration Pattern**: REST API for sending; webhook receiver for delivery events

**Authentication**: API key (stored in secrets manager per NFR-SEC-004)

**Error Handling**: Failed sends queued for retry up to 3 times with exponential backoff; after 3 failures, alert raised and notification degraded gracefully per NFR-A-003

**SLA**: Email delivery within 60 seconds of trigger event for transactional (invitation, password reset); within 5 minutes for notification emails

**Candidate Services**: AWS SES, SendGrid, Postmark

**Priority**: MUST_HAVE

---

### INT-002: OAuth Identity Providers (Google, Microsoft)

**Purpose**: Enable users to authenticate using existing Google or Microsoft accounts, reducing registration friction and supporting enterprise SSO.

**Integration Type**: OAuth 2.0 Authorization Code flow with PKCE

**Data Exchanged**:

- From Provider to Portal: User ID, email, display name, profile picture URL, access token
- Portal stores: Provider user ID, email (as primary identifier), display name

**Authentication**: OAuth 2.0 client credentials (client ID and secret stored in secrets manager)

**Error Handling**: OAuth failure (user denies, provider timeout) returns user to login with informative error; fallback to email/password always available

**Priority**: MUST_HAVE

---

### INT-003: Payment Processing (Stripe)

**Purpose**: Manage subscription billing: create subscriptions, handle payment method updates, process renewals, and manage dunning for failed payments.

**Integration Type**: Stripe SDK (backend); Stripe Customer Portal (frontend redirect for self-service)

**Data Exchanged**:

- From Portal to Stripe: Customer record, subscription plan, payment method (Stripe handles card data — PCI compliance delegated to Stripe)
- From Stripe to Portal: Subscription status webhooks (created, renewed, cancelled, payment_failed)

**Authentication**: Stripe API key (restricted key with minimum required permissions, stored in secrets manager)

**Error Handling**: Payment failure triggers grace period of 7 days per FR-021; webhook processing is idempotent (Stripe event IDs tracked to prevent duplicate processing)

**PCI Compliance**: Card data is never processed or stored by the portal — Stripe handles all card tokenization. Portal is PCI SAQ A compliant.

**Priority**: MUST_HAVE

---

### INT-004: Product Analytics

**Purpose**: Capture user behaviour events (feature usage, funnel conversion, retention) to inform product decisions and NPS tracking.

**Integration Type**: Client-side JavaScript SDK (browser); server-side event capture for backend events

**Data Exchanged**:

- From Portal to Analytics: Anonymised user ID (not email), event name, properties, timestamp
- PII: Email and display name MUST NOT be sent to analytics without user consent per NFR-C-001/C-003

**Authentication**: Analytics write key (public client-side key acceptable; server key stored in secrets manager)

**Alignment**: NFR-C-003 (cookie consent gate — analytics events only sent after user opt-in)

**Candidate Services**: PostHog (self-hosted or cloud), Mixpanel

**Priority**: SHOULD_HAVE

---

### INT-005: Cloud Infrastructure Provider

**Purpose**: Host all application compute, database, object storage, CDN, and networking.

**Integration Type**: Cloud-native services via IaC (Terraform/Pulumi)

**Requirements**:

- UK or EU region required for data residency (NFR-C-001)
- Managed database service with automated backup and point-in-time recovery
- Managed Kubernetes or container orchestration for application tier
- Object/blob storage for file attachments and exports
- CDN for static asset delivery

**Candidate Providers**: AWS (eu-west-2 London), GCP (europe-west2), Azure (UK South)

**Priority**: MUST_HAVE

---

## Data Requirements

### DR-001: User Entity

**Description**: Represents a Quento1 platform user account. Contains identity, authentication, and preference data.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Unique user identifier | Primary key, immutable |
| email | String(320) | Yes | User email address | Unique, lowercase, indexed |
| password_hash | String(72) | No | bcrypt password hash | Null if OAuth-only |
| display_name | String(100) | Yes | Shown in UI | Not null |
| avatar_url | String(2048) | No | Profile picture URL | Nullable |
| mfa_enabled | Boolean | Yes | MFA status | Default false |
| mfa_secret | String(32) | No | TOTP secret (encrypted) | Nullable |
| email_verified | Boolean | Yes | Verification status | Default false |
| notification_prefs | JSONB | Yes | Per-event email opt-outs | Default: all enabled |
| created_at | Timestamptz | Yes | Account creation time | Indexed |
| last_login_at | Timestamptz | No | Last successful login | Indexed |
| deleted_at | Timestamptz | No | Soft delete timestamp | Null = active |

**Data Classification**: CONFIDENTIAL (contains PII — email, display name)

**Data Retention**: Retained while active; soft-deleted on account cancellation; hard-deleted 90 days after cancellation per NFR-C-001

**Volume**: Year 1: 500 records; Year 3: 5,000 records

---

### DR-002: Workspace Entity

**Description**: Represents a team or organisation's isolated environment within Quento1. All Projects and Tasks belong to a Workspace.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Unique workspace identifier | Primary key |
| name | String(100) | Yes | Display name | Not null |
| slug | String(50) | Yes | URL-safe identifier | Unique, lowercase, immutable after 24h |
| logo_url | String(2048) | No | Workspace logo | Nullable |
| stripe_customer_id | String(50) | No | Stripe customer reference | Nullable, indexed |
| subscription_status | Enum | Yes | Current billing status | active, trialing, past_due, cancelled |
| created_at | Timestamptz | Yes | Creation timestamp | Indexed |

**Data Classification**: INTERNAL

**Volume**: Year 1: 100 records; Year 3: 1,000 records

---

### DR-003: Task Entity

**Description**: Core entity. Represents a unit of work within a Project.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Unique task identifier | Primary key |
| project_id | UUID | Yes | Parent project | Foreign key, indexed |
| workspace_id | UUID | Yes | Parent workspace | Foreign key, indexed |
| title | String(500) | Yes | Task title | Not null |
| description | Text | No | Markdown content | Max 50,000 chars |
| status | Enum | Yes | Workflow status | to_do, in_progress, in_review, done, archived |
| priority | Enum | Yes | Importance level | critical, high, medium, low |
| assignee_id | UUID | No | Assigned user | Foreign key nullable, indexed |
| created_by | UUID | Yes | Creator user ID | Foreign key, indexed |
| due_date | Date | No | Target completion | Nullable, indexed |
| created_at | Timestamptz | Yes | Creation timestamp | Indexed |
| updated_at | Timestamptz | Yes | Last update timestamp | Indexed |
| archived_at | Timestamptz | No | Archive timestamp | Null = active |
| deleted_at | Timestamptz | No | Soft delete timestamp | Null = not deleted |
| parent_task_id | UUID | No | Parent task for sub-tasks | FK → Task(id), nullable, indexed; NULL = top-level task |

**Relationships**:

- Many-to-one with Project (DR-004) via project_id
- Many-to-one with User (DR-001) via assignee_id (nullable)
- Many-to-one with User (DR-001) via created_by
- One-to-many with Comment (DR-005) via task_id
- One-to-many with ActivityLog (DR-007) via task_id
- Self-referential many-to-one via parent_task_id (sub-task → parent task; max 1 level deep)

**Data Classification**: INTERNAL

**Access Patterns**: Most queries filter by (project_id, status), (assignee_id, status), (workspace_id, due_date), and (parent_task_id) for sub-task retrieval. Compound indexes required. A partial index on `parent_task_id IS NULL` optimises top-level task queries.

**Volume**: Year 1: 100,000 records (includes sub-tasks); Year 3: 2,000,000 records

---

### DR-008a: Sub-Task Considerations

**Description**: Sub-tasks are stored as Task records (DR-003) with a non-null `parent_task_id`. No separate table is required; the self-referential FK on the Task entity is sufficient. The following constraints are enforced at the application and database level:

**Integrity Rules**:

| Rule | Enforcement |
|------|-------------|
| Maximum nesting depth = 1 | Application-layer check before INSERT: if parent_task.parent_task_id IS NOT NULL → reject |
| Sub-task belongs to same Project as parent | Application-layer check on create |
| Parent task archival cascades to sub-tasks | Database trigger or application-layer job: set sub-tasks.archived_at when parent archived |
| Parent task deletion cascades to sub-tasks | Application-layer soft delete of all sub-tasks on parent soft delete |
| Sub-task count visible on parent | Computed at query time via COUNT(*) WHERE parent_task_id = :id AND deleted_at IS NULL; cached in Redis with 5-second TTL for Kanban view performance |

**Progress Calculation**:

- `completed_count` = COUNT(*) WHERE parent_task_id = :id AND status = 'done' AND deleted_at IS NULL
- `total_count` = COUNT(*) WHERE parent_task_id = :id AND deleted_at IS NULL
- Progress = completed_count / total_count (percentage for progress bar)

**Data Classification**: INTERNAL (same as parent Task entity)

**Volume**: Estimated 3-5 sub-tasks per task that uses sub-tasks; approximately 20% of tasks expected to have sub-tasks. Year 1: 20,000 sub-task records within the 100,000 Task total.

---

### DR-004: Project Entity

**Description**: Organises Tasks within a Workspace. Users collaborate on Projects.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Unique project identifier | Primary key |
| workspace_id | UUID | Yes | Parent workspace | Foreign key, indexed |
| name | String(100) | Yes | Project name | Not null |
| description | Text | No | Project description | Max 5,000 chars |
| icon | String(50) | No | Emoji or icon identifier | Nullable |
| created_by | UUID | Yes | Creator user ID | Foreign key |
| created_at | Timestamptz | Yes | Creation timestamp | |
| archived_at | Timestamptz | No | Archive timestamp | Null = active |

**Data Classification**: INTERNAL

**Volume**: Year 1: 500 records; Year 3: 5,000 records

---

### DR-005: Comment Entity

**Description**: User comments attached to Tasks. Supports Markdown and @mentions.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Unique comment identifier | Primary key |
| task_id | UUID | Yes | Parent task | Foreign key, indexed |
| author_id | UUID | Yes | Author user ID | Foreign key, indexed |
| content | Text | Yes | Markdown content | Max 10,000 chars |
| created_at | Timestamptz | Yes | Post timestamp | Indexed |
| edited_at | Timestamptz | No | Last edit timestamp | Null = never edited |
| deleted_at | Timestamptz | No | Soft delete timestamp | Null = visible |

**Data Classification**: INTERNAL

**Volume**: Year 1: 500,000 records; Year 3: 10,000,000 records

---

### DR-006: Notification Entity

**Description**: Tracks in-app notifications for each user. Consumed by the notification bell UI.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Notification ID | Primary key |
| user_id | UUID | Yes | Recipient user | Foreign key, indexed |
| event_type | Enum | Yes | Notification category | task_assigned, mentioned, status_changed, etc. |
| payload | JSONB | Yes | Event-specific data | Context for rendering |
| read_at | Timestamptz | No | Read timestamp | Null = unread |
| created_at | Timestamptz | Yes | Notification created | Indexed |

**Data Classification**: INTERNAL

**Retention**: Notifications older than 90 days are deleted (background job)

**Volume**: Year 1: 5,000,000 records (high velocity, aggressive retention)

---

### DR-007: Activity Log Entity

**Description**: Immutable audit trail of all changes to Task entities and authentication events. Append-only.

**Attributes**:

| Attribute | Type | Required | Description | Constraints |
|-----------|------|----------|-------------|-------------|
| id | UUID | Yes | Log entry ID | Primary key |
| entity_type | Enum | Yes | Type of entity changed | task, user, workspace, auth |
| entity_id | UUID | Yes | ID of changed entity | Indexed |
| actor_id | UUID | No | User who caused change | Nullable (system events) |
| action | String(100) | Yes | Action type | created, updated, deleted, status_changed, etc. |
| old_value | JSONB | No | Previous state | Nullable |
| new_value | JSONB | No | New state | Nullable |
| created_at | Timestamptz | Yes | Event timestamp | Indexed (partition key) |

**Data Classification**: INTERNAL / AUDIT

**Retention**: 1 year hot storage; 2 years cold storage (NFR-C-002)

**Immutability**: No UPDATE or DELETE operations permitted on this table. Write-once, read-many.

**Volume**: Year 1: 10,000,000 records; Year 3: 100,000,000 records (requires partitioning by month)

---

### DR-008: Data Migration

**Migration Scope**: No data migration required. The Task Management Portal is a greenfield system with no legacy data to migrate.

**Future Consideration**: A bulk import capability for CSV-formatted task data (from Jira, Asana) is a Phase 2 requirement and is out of scope for v1.0.

---

### DR-009: Data Quality Standards

**Data Accuracy**: All task field values validated at API boundary before persistence. Invalid inputs return HTTP 422 with field-level error details.

**Data Completeness**: Required fields enforced at database constraint level (NOT NULL) and API validation level. Dual enforcement prevents data integrity issues from direct database access during development.

**Data Consistency**: Workspace member access validated on every request (not cached beyond the request session). No eventual consistency for access control decisions.

**Data Lineage**: All task changes tracked via Activity Log (DR-007). Transformation logic for exports is version-controlled. Source-to-target mapping documented for analytics integration (INT-004).

**Alignment**: Architecture Principle 7 (Data Quality and Lineage), Principle 8 (Single Source of Truth)

---

## Constraints and Assumptions

### Technical Constraints

**TC-1**: The system must be built as a web application with a RESTful backend API, in alignment with Quento1's architecture principles (ARC-000-PRIN-v1.0). No monolithic or desktop-only architecture.

**TC-2**: All infrastructure must be deployed to a cloud provider with UK or EU region presence to satisfy data residency requirements (NFR-C-001).

**TC-3**: The technology stack must be maintainable by a team of 3-5 engineers. Exotic or niche frameworks that reduce the hiring pool are to be avoided.

**TC-4**: Payment processing must use Stripe as the billing provider (existing commercial relationship at Quento1).

---

### Business Constraints

**BC-1**: Production launch must not slip beyond 30 September 2026 without CEO and board approval. This date is non-negotiable for commercial reasons.

**BC-2**: Cloud infrastructure budget is capped at £3,000/month for the first 6 months post-launch. This must be architected from day one — not retrofitted.

**BC-3**: The MVP feature set is fixed as defined in FR-001 through FR-021 and FR-023 through FR-026 (FR-022 is SHOULD_HAVE). No new MUST_HAVE features may be added after Sprint 1 planning without CEO sign-off and a corresponding scope removal.

---

### Assumptions

**A-1**: Google and Microsoft OAuth developer application credentials will be available to the team before Sprint 2.

**A-2**: Stripe account will be configured with the appropriate product and pricing plan before Sprint 4.

**A-3**: Beta customers will be available for usability testing from Month 3 (end of April 2026).

**A-4**: The engineering team will consist of at least 2 backend engineers, 1 frontend engineer, and 1 full-stack/DevOps engineer for the duration of the build.

**A-5**: A cloud provider account with UK/EU region access will be provisioned and accessible before Sprint 1.

**Validation Plan**: All assumptions validated in Sprint 1 kickoff. Blockers on A-1, A-2, and A-5 escalated to CTO and CEO immediately.

---

## Success Criteria and KPIs

### Business Success Metrics

| Metric | Baseline | Target | Timeline | Measurement Method |
|--------|----------|--------|----------|--------------------|
| Paying customers | 0 | 100 | Month 6 (Sept 2026) | Stripe billing system |
| Monthly Recurring Revenue | £0 | £15,000 | Month 6 | Stripe |
| ARR | £0 | £180,000 | Month 18 | Stripe |
| Net Promoter Score | N/A | 40+ | Month 12 | Quarterly NPS survey |
| Monthly churn rate | N/A | Below 3% | From Month 1 | Cohort analysis |
| Infrastructure cost / MRR | N/A | Below 20% | All milestones | Cloud billing + Stripe |

### Technical Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| System availability (monthly) | 99.5% | Uptime monitoring |
| API p95 response time | Below 500ms | APM tooling |
| API error rate | Below 0.1% | Log aggregation and APM |
| Deployment frequency | At least 2x per week | CI/CD pipeline metrics |
| Mean Time to Recovery (MTTR) | Below 30 minutes | Incident tracking |
| Architecture principle compliance | 100% (16/16) | Architecture review gates |
| Test coverage (core modules) | 80%+ | CI/CD coverage report |

### User Adoption Metrics

| Metric | Target | Timeline | Measurement Method |
|--------|--------|----------|--------------------|
| Monthly Active Users / Registered | 65%+ | Month 12 | Product analytics (INT-004) |
| Time-to-first-task (new users) | Below 5 minutes | Pre-launch usability test | Usability test recording |
| Support ticket volume per user | Below 0.1/month | From Month 3 | Support ticketing system |
| Feature adoption (task comments) | 40%+ of tasks have 1+ comment | Month 6 | Product analytics |

---

## Dependencies and Risks

### Dependencies

| Dependency | Description | Owner | Target Date | Status | Impact if Delayed |
|------------|-------------|-------|-------------|--------|-------------------|
| Cloud provider account | Production account in UK/EU region | Engineering Lead | Sprint 1 Week 1 | Not started | HIGH — blocks all IaC work |
| Google OAuth credentials | OAuth app registration for Google sign-in | Product Manager | Sprint 2 Week 1 | Not started | MEDIUM — blocks FR-001 OAuth |
| Microsoft OAuth credentials | OAuth app registration for Microsoft | Product Manager | Sprint 2 Week 1 | Not started | MEDIUM — blocks FR-001 OAuth |
| Stripe account | Commercial billing account configured | CEO / Finance | Sprint 4 | Not started | HIGH — blocks FR-021 |
| Beta customer cohort | 10+ paying pilot customers for testing | CEO / Product Manager | Month 3 (June 2026) | Not started | HIGH — blocks NPS measurement |
| Penetration test scope | External security firm engaged | CTO (Jane Smith) | Month 5 | Not started | MEDIUM — must complete before launch |

### Risks

| Risk ID | Description | Probability | Impact | Mitigation Strategy | Owner |
|---------|-------------|-------------|--------|---------------------|-------|
| R-1 | MVP scope creep delays launch | HIGH | HIGH | Formal scope sign-off; change control process | CEO, Product Manager |
| R-2 | Architecture foundations deprioritised for speed | MEDIUM | HIGH | CTO veto on principle violations; foundations in Sprint 1-2 non-negotiable | CTO (Jane Smith) |
| R-3 | Poor beta adoption reveals product-market fit issues | MEDIUM | CRITICAL | User interviews before Sprint 1; beta programme from Month 3 | Product Manager |
| R-4 | Infrastructure costs exceed 20% MRR threshold | MEDIUM | MEDIUM | Cost monitoring from Day 1; auto-scaling with right-sizing | CTO, Finance |
| R-5 | Third-party service outage (email, auth, payments) | LOW | HIGH | Circuit breakers, graceful degradation (NFR-A-003) | Engineering |
| R-6 | Key engineer departure | LOW | HIGH | ADR documentation; knowledge sharing culture; no single points of knowledge | CTO (Jane Smith) |

---

## Requirement Conflicts & Resolutions

### Conflict C-1: Speed-to-Market vs Architecture Foundations

**Conflicting Requirements**:

- **Requirement A**: BR-001 — Launch MVP to 100 paying customers by 30 September 2026 (6-month timeline)
- **Requirement B**: NFR-M-003, NFR-M-004, NFR-M-005 — Infrastructure as Code, 80% test coverage, full CI/CD pipeline from Sprint 1

**Stakeholders Involved**:

- **CEO / Founder** (SD-1, HIGH pressure): Wants the commercial milestone hit by September 2026. Every sprint spent on "invisible" infrastructure is a sprint not spent on customer-facing features.
- **CTO / Jane Smith** (SD-2, CRITICAL): Architecture Principle 14 (IaC) and Principle 15 (Automated Testing) are non-negotiable. Skipping them to accelerate delivery will create a debt that costs 3-5x more to repay in 12 months.

**Nature of Conflict**: A 6-month build timeline is genuinely tight for a well-architected SaaS system. Sprints 1-2 invested in foundations (IaC, CI/CD, auth, database schema) produce no user-visible features, creating short-term pressure from the CEO. However, building without these foundations will slow velocity from Month 4 onwards — precisely when the business needs to iterate fast on customer feedback.

**Trade-off Analysis**:

| Option | Pros | Cons | Impact |
|--------|------|------|--------|
| **Option 1**: Skip foundations, ship features first | Faster early delivery of visible features | Technical debt compounds; Sprint 6+ velocity collapses; security risk; architectural rework likely | CEO short-term happy; CTO objection; Engineering burnout |
| **Option 2**: Full foundations before any features (Sprints 1-4) | Clean architecture from day one | Visible features delayed; commercial milestone risk | CTO happy; CEO frustrated; beta delayed |
| **Option 3**: Foundations in Sprints 1-2, features from Sprint 3 (chosen) | Balance: architecture protected, feature delivery starts early | Slightly later feature velocity in early sprints | Both stakeholders satisfied with phased approach |

**Resolution Strategy**: PHASE

**Decision**: Option 3 — Sprints 1-2 are dedicated exclusively to non-negotiable architecture foundations: IaC for all environments, CI/CD pipeline with quality gates, authentication service (FR-001, FR-002), database schema and migrations, observability stack. Feature delivery begins Sprint 3.

**Rationale**: The CEO accepted this after the CTO modelled the cost of technical debt: a 2-sprint investment now protects velocity from Sprint 3 to launch and prevents a £50K+ remediation sprint post-launch. The 6-month commercial deadline is preserved because feature delivery is compressed into Sprints 3-12, not the full 12 sprints.

**Decision Authority**: CTO (Jane Smith) — veto authority on architecture principle compliance; CEO sign-off on timeline trade-off. Aligned with RACI in ARC-001-STKE-v1.0.

**Impact on Requirements**:

- **Modified**: Sprint 1-2 scope locked to: IaC, CI/CD, auth (FR-001, FR-002), DB schema (DR-001 through DR-007), observability (NFR-M-001)
- **Preserved**: BR-001 September 2026 target maintained
- **Non-negotiable**: NFR-M-003, NFR-M-004, NFR-M-005 remain CRITICAL requirements — cannot be traded against any business requirement

**Stakeholder Management**:

- **CEO (accepted timeline for foundations)**: Receives weekly sprint progress report showing velocity trajectory and feature readiness forecast from Sprint 3
- **CTO (accepted compressed feature delivery)**: Architecture review gates enforced at Sprint 2 completion; any principle violation must be raised immediately

---

### Conflict C-2: Agile Roadmap Flexibility vs Engineering Stability

**Conflicting Requirements**:

- **Requirement A**: Product Manager requires the ability to reprioritise the backlog based on beta user feedback (agile, customer-driven iteration)
- **Requirement B**: Engineering Team requires stable sprint scope to maintain velocity and code quality; mid-sprint changes break estimates, create context-switching, and reduce test coverage

**Stakeholders Involved**:

- **Product Manager** (SD-3): Customer feedback from beta users may reveal that planned features are wrong. The PM must be able to pivot the roadmap quickly to chase product-market fit.
- **Engineering Team** (SD-4): Stable requirements within a sprint allow engineers to complete tasks to a quality standard. Frequent changes create partially-completed work, reduce coverage, and increase bugs.

**Nature of Conflict**: Agility requires the ability to change direction; engineering quality requires stability to complete work. Both are legitimate and necessary for a successful product.

**Resolution Strategy**: COMPROMISE

**Decision**: Fixed sprint scope (2-week sprints) with a structured change management process. Any new requirement or priority change is assessed in bi-weekly backlog refinement. Only in the case of a critical discovery (beta NPS below 15, fundamental product-market fit issue) does mid-sprint scope change occur — and only by joint agreement of PM and CTO.

**Impact on Requirements**:

- **Added**: NFR for sprint planning: sprint scope is locked at sprint planning; changes require PM and CTO joint approval; change log maintained
- **Modified**: FR requirements marked SHOULD_HAVE are first candidates for deferral if timeline pressure requires scope reduction

**Stakeholder Management**:

- **Product Manager**: Has full control over backlog priority between sprints; bi-weekly refinement gives a regular pivot window
- **Engineering Team**: Within-sprint stability guaranteed; retrospective is the venue for surfacing quality concerns from rapid changes

---

## Timeline and Milestones

### High-Level Milestones

| Milestone | Description | Target Date | Dependencies |
|-----------|-------------|-------------|--------------|
| Sprint 1-2 Complete | Architecture foundations live (IaC, CI/CD, auth, DB, observability) | 2026-04-03 | Cloud account, OAuth credentials |
| Alpha (Internal) | Core task management features usable by Quento1 team | 2026-04-30 | Sprint 1-2 complete |
| Beta Launch | 10-15 paying pilot customers onboarded; NPS measurement begins | 2026-06-30 | Alpha stable, Stripe configured, beta cohort |
| UAT Complete | Beta NPS above 20; critical bugs resolved; launch checklist passed | 2026-08-31 | Beta programme |
| Production Launch | 100 paying customers target; full public availability | 2026-09-30 | UAT complete, pen test complete |
| NPS Milestone | NPS 40+ measured in first quarterly survey | 2026-12-31 | 6 months of production operation |
| ARR Milestone | £180K ARR | 2027-03-30 | 18 months post-launch |

---

## Budget

### Build Cost Estimate

| Category | Estimated Cost | Notes |
|----------|----------------|-------|
| Engineering (6 months) | £180,000 | 4 FTE at £30K/6mo average (blended rate) |
| Cloud infrastructure (build) | £6,000 | Dev and staging environments x 6 months |
| Third-party services (build) | £2,000 | Email testing, monitoring tools, SaaS |
| Security testing (pen test) | £8,000 | External firm, pre-launch assessment |
| Tooling and licences | £3,000 | CI/CD, analytics, APM tooling |
| **Total Build Cost** | **£199,000** | |

### Ongoing Operational Costs (Monthly, at 100 Customers)

| Category | Monthly Cost | Notes |
|----------|-------------|-------|
| Cloud infrastructure | £1,500 | Production, multi-AZ; scales with usage |
| Email delivery | £50 | 10,000 emails/month estimated |
| Analytics | £100 | PostHog cloud or equivalent |
| Monitoring / APM | £200 | Datadog or equivalent |
| Third-party SaaS (misc) | £150 | Tools, integrations |
| **Total Operational** | **£2,000/month** | 13% of £15K MRR target — within BC-2 |

---

## Approval

### Requirements Review

| Reviewer | Role | Status | Date | Comments |
|----------|------|--------|------|----------|
| CEO / Founder | Business Sponsor | Pending | | |
| Product Manager | Product Owner | Pending | | |
| Jane Smith | Head of Engineering / Architect | Pending | | |
| Finance Lead | Budget Controller | Pending | | |

### Sign-Off

By approving this document, stakeholders confirm that requirements are complete, understood, and approved to proceed to design phase.

| Stakeholder | Signature | Date |
|-------------|-----------|------|
| CEO / Founder | _________ | |
| Jane Smith, Head of Engineering | _________ | |
| Product Manager | _________ | |

---

## Appendices

### Appendix A: Glossary

| Term | Definition |
|------|------------|
| ARR | Annual Recurring Revenue — annualised value of active subscriptions |
| Kanban | Visual board where tasks are represented as cards in status columns |
| MFA | Multi-Factor Authentication — requires two or more verification methods |
| MRR | Monthly Recurring Revenue |
| MVP | Minimum Viable Product — smallest feature set that delivers customer value |
| NPS | Net Promoter Score — customer loyalty metric (-100 to +100) |
| RBAC | Role-Based Access Control — permissions determined by assigned roles |
| RTO | Recovery Time Objective — maximum acceptable outage duration |
| RPO | Recovery Point Objective — maximum acceptable data loss window |
| SLA | Service Level Agreement — committed uptime or performance target |
| TOTP | Time-based One-Time Password — RFC 6238 MFA standard |
| Workspace | Quento1's top-level organisational container for a team or company |

### Appendix B: Reference Documents

- [ARC-000-PRIN-v1.0 — Quento1 Enterprise Architecture Principles](../000-global/ARC-000-PRIN-v1.0.md)
- [ARC-001-STKE-v1.0 — Task Management Portal Stakeholder Drivers & Goals Analysis](ARC-001-STKE-v1.0.md)

---

## External References

| Document | Type | Source | Key Extractions | Path |
|----------|------|--------|-----------------|------|
| ARC-000-PRIN-v1.0 | Architecture Principles | Quento1 Global | 16 architecture principles mapped to NFRs | ../000-global/ARC-000-PRIN-v1.0.md |
| ARC-001-STKE-v1.0 | Stakeholder Analysis | Quento1 Project 001 | 7 stakeholder drivers, 5 goals, 3 outcomes | ARC-001-STKE-v1.0.md |

---

**Generated by**: ArcKit `/arckit:requirements` command
**Generated on**: 2026-03-10
**ArcKit Version**: 4.1.1
**Project**: Task Management Portal (Project 001)
**AI Model**: claude-sonnet-4-6
**Generation Context**: v1.1 update — added UC-5 (sub-task management), FR-023 to FR-026, DR-003 updated (parent_task_id), DR-008a (sub-task integrity rules). Derived from ARC-001-STKE-v1.0 and ARC-000-PRIN-v1.0.
