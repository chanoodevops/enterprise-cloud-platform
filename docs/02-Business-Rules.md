# Business Rules

## 1. Document Information

**Product:** Enterprise Asset Management Platform
**Version:** 1.0
**Status:** Draft
**Related Document:** `docs/01-PRD.md`

---

## 2. Purpose

This document defines the business rules that govern asset lifecycle management, user access, approvals, maintenance, notifications, and auditability within the Enterprise Asset Management Platform.

These rules must be consistently enforced across the user interface, API, application services, database constraints, and automated workflows.

---

## 3. Rule Categories

The business rules are grouped into:

1. User and access rules
2. Asset master-data rules
3. Asset assignment rules
4. Asset transfer rules
5. Asset return rules
6. Maintenance rules
7. Approval rules
8. Notification rules
9. Audit rules
10. Reporting and data-retention rules

---

# 4. User and Access Rules

## BR-USER-001 Active User Requirement

Only active users may sign in to the platform.

### Enforcement

- Authentication service
- User status validation
- Session validation

### Expected Behaviour

- Active users may authenticate.
- Inactive users must be denied access.
- Existing sessions for deactivated users should be invalidated.

---

## BR-USER-002 Unique User Identity

Each user must have a unique email address or username.

### Enforcement

- Database unique constraint
- Application validation
- API validation

---

## BR-USER-003 Role Assignment

Every active user must have at least one assigned role.

### Exception

A newly created user may remain in a pending state until a role is assigned.

---

## BR-USER-004 Least Privilege

Users may only perform actions explicitly permitted by their assigned roles and permissions.

### Example

An employee may view assigned assets but may not create or delete asset records.

---

## BR-USER-005 Deactivated User Restriction

A deactivated user:

- Cannot sign in
- Cannot receive new asset assignments
- Cannot approve requests
- Cannot process maintenance work
- Remains visible in historical records

---

## BR-USER-006 Separation of Duties

Where practical, the requester and approver should not be the same user.

### Applies To

- Asset disposal
- High-value asset transfer
- Exceptional maintenance approval
- Administrative privilege escalation

---

# 5. Asset Master-Data Rules

## BR-ASSET-001 Unique Asset Number

Each asset must have a unique asset number.

### Enforcement

- Application validation
- Database unique constraint

---

## BR-ASSET-002 Required Asset Fields

An asset record must include:

- Asset number
- Asset name
- Category
- Current location
- Condition
- Status

Optional fields may include:

- Serial number
- Purchase date
- Purchase cost
- Warranty expiry date
- Manufacturer
- Model

---

## BR-ASSET-003 Valid Asset Status

An asset must have one valid lifecycle status.

Suggested statuses:

- Draft
- Available
- Pending Assignment
- Assigned
- Pending Transfer
- Under Maintenance
- Pending Disposal
- Disposed
- Lost
- Retired

---

## BR-ASSET-004 Asset Status Transition

Asset status changes must follow permitted transitions.

| Current Status     | Allowed Next Status                                           |
| ------------------ | ------------------------------------------------------------- |
| Draft              | Available, Retired                                            |
| Available          | Pending Assignment, Under Maintenance, Pending Disposal, Lost |
| Pending Assignment | Assigned, Available                                           |
| Assigned           | Pending Transfer, Under Maintenance, Available, Lost          |
| Pending Transfer   | Assigned, Available                                           |
| Under Maintenance  | Available, Assigned, Pending Disposal                         |
| Pending Disposal   | Disposed, Available                                           |
| Lost               | Available, Pending Disposal                                   |
| Disposed           | None                                                          |
| Retired            | None                                                          |

Direct status changes outside these transitions must be rejected.

---

## BR-ASSET-005 Asset Deletion

Asset records must not be physically deleted after business activity exists.

### Required Approach

- Mark the asset as retired, disposed, or inactive.
- Preserve assignment, maintenance, transfer, and audit history.

---

## BR-ASSET-006 Category Requirement

Every asset must belong to one active asset category.

An inactive category cannot be assigned to a new asset.

---

## BR-ASSET-007 Location Requirement

Every asset must have a current location.

Location changes must be recorded as part of transfer, assignment, return, or administrative correction.

---

## BR-ASSET-008 Serial Number

When a serial number is provided, it should be unique within the organization unless an authorized administrator approves an exception.

---

# 6. Asset Assignment Rules

## BR-ASSIGN-001 Assignment Eligibility

Only assets with status `Available` may be assigned.

---

## BR-ASSIGN-002 Single Active Assignment

An asset may have only one active assignment at a time.

### Enforcement

- Application validation
- Database constraint where practical
- Transactional processing

---

## BR-ASSIGN-003 Assignment Target

An asset may be assigned to:

- An active employee
- An active department
- An approved shared location

---

## BR-ASSIGN-004 Assignment Approval

An assignment may require approval based on:

- Asset category
- Asset value
- Department policy
- Requester role
- Assignment duration

The initial MVP may use fixed approval rules.

---

## BR-ASSIGN-005 Assignment Acknowledgement

An employee assignment is not fully completed until the assigned employee acknowledges receipt.

Suggested statuses:

- Pending Approval
- Approved
- Pending Acknowledgement
- Active
- Rejected
- Cancelled
- Completed

---

## BR-ASSIGN-006 Assignment Date

The assignment start date must not be earlier than the asset acquisition date.

---

## BR-ASSIGN-007 Assignment History

All completed, rejected, cancelled, and active assignments must remain available in history.

---

# 7. Asset Transfer Rules

## BR-TRANSFER-001 Transfer Eligibility

An asset may be transferred only when:

- It has an active assignment, or
- It is available at a valid location

---

## BR-TRANSFER-002 Transfer Types

Supported transfer types include:

- Employee-to-employee
- Department-to-department
- Location-to-location
- Employee-to-location
- Department-to-location

---

## BR-TRANSFER-003 Transfer Approval

A transfer may require approval based on:

- Source department
- Destination department
- Asset value
- Asset classification
- Policy configuration

---

## BR-TRANSFER-004 Transfer Completion

A transfer is complete only when:

- Required approval is completed
- The receiving party acknowledges receipt when required
- The asset location and assignment are updated
- The transfer history is recorded

---

## BR-TRANSFER-005 Transfer Cancellation

A pending transfer may be cancelled before completion.

A completed transfer cannot be deleted; it may only be reversed through a new transfer transaction.

---

# 8. Asset Return Rules

## BR-RETURN-001 Return Eligibility

Only an actively assigned asset may be returned.

---

## BR-RETURN-002 Condition Assessment

The asset condition must be recorded during return.

Suggested conditions:

- New
- Good
- Fair
- Damaged
- Non-functional

---

## BR-RETURN-003 Return Outcome

After return, the asset may transition to:

- Available
- Under Maintenance
- Pending Disposal
- Lost

The selected outcome depends on inspection results.

---

## BR-RETURN-004 Return Completion

A return is complete when:

- Return date is recorded
- Condition is assessed
- Active assignment is closed
- Asset status is updated
- Audit record is created

---

# 9. Maintenance Rules

## BR-MAINT-001 Maintenance Eligibility

A maintenance request may be created for any asset that is not disposed or retired.

---

## BR-MAINT-002 Maintenance Request Information

A maintenance request must include:

- Asset
- Reported issue
- Requester
- Priority
- Request date

Optional information:

- Photos
- Supporting documents
- Preferred completion date

---

## BR-MAINT-003 Maintenance Priority

Supported priorities:

- Low
- Medium
- High
- Critical

Critical requests should generate an immediate notification.

---

## BR-MAINT-004 Maintenance Status

Suggested maintenance statuses:

- Open
- Assigned
- In Progress
- Waiting for Parts
- Waiting for Approval
- Completed
- Cancelled

---

## BR-MAINT-005 Technician Assignment

Only an active user with the Technician role or equivalent permission may be assigned maintenance work.

---

## BR-MAINT-006 Maintenance Completion

A maintenance request may be completed only when:

- Diagnosis is recorded
- Work completed is recorded
- Completion date is provided
- Final asset condition is recorded
- Asset status is updated

---

## BR-MAINT-007 Maintenance History

Completed maintenance records must not be deleted by standard users.

Corrections must be made through an authorized amendment process.

---

## BR-MAINT-008 Asset Availability During Maintenance

An asset under maintenance must not be assigned or transferred until maintenance is completed or cancelled.

---

# 10. Approval Rules

## BR-APPROVAL-001 Approval Requirement

The following operations may require approval:

- Asset assignment
- Asset transfer
- Asset disposal
- High-cost maintenance
- Role or privilege change

---

## BR-APPROVAL-002 Approval Decision

An approver may:

- Approve
- Reject
- Return for clarification

---

## BR-APPROVAL-003 Approval Comments

A comment is required when:

- A request is rejected
- A request is returned for clarification
- An exceptional approval is granted

---

## BR-APPROVAL-004 Self-Approval Restriction

A requester must not approve their own request when separation of duties is required.

---

## BR-APPROVAL-005 Approval Sequence

Where multiple approval levels exist, approvals must occur in the defined order.

Example:

```text
Department Manager
        ↓
Asset Manager
        ↓
System Administrator
```

---

## BR-APPROVAL-006 Approval Expiry

A pending approval may expire after a configurable period.

The MVP may initially use a fixed period, such as seven days.

---

## BR-APPROVAL-007 Rejected Request

A rejected transaction must not update the asset assignment, transfer, disposal, or maintenance state.

---

# 11. Notification Rules

## BR-NOTIFY-001 Notification Events

Notifications should be generated for:

- New approval request
- Approval decision
- Asset assignment
- Assignment acknowledgement request
- Asset transfer
- Maintenance assignment
- Maintenance status change
- Critical maintenance issue
- Overdue approval
- Overdue return

---

## BR-NOTIFY-002 Notification Recipients

Recipients must be determined based on the event.

Examples:

- Requester
- Approver
- Asset manager
- Assigned employee
- Assigned technician
- Department manager

---

## BR-NOTIFY-003 Notification Failure

Notification delivery failure must not roll back the main business transaction.

Failed notifications should be logged and retried where appropriate.

---

## BR-NOTIFY-004 Notification Channels

The MVP may support:

- In-application notifications
- Email notifications

Future channels may include:

- SMS
- Microsoft Teams
- Slack
- Mobile push notifications

---

# 12. Audit Rules

## BR-AUDIT-001 Auditable Events

The platform must record audit events for:

- Authentication activity
- User and role changes
- Asset creation and updates
- Assignment
- Transfer
- Return
- Maintenance
- Approval decisions
- Administrative configuration changes

---

## BR-AUDIT-002 Audit Record Content

Each audit record should include:

- Event timestamp
- User identifier
- Action
- Entity type
- Entity identifier
- Previous value where applicable
- New value where applicable
- Correlation identifier
- Source IP where appropriate

---

## BR-AUDIT-003 Audit Immutability

Standard users must not modify or delete audit records.

---

## BR-AUDIT-004 Audit Access

Only authorized administrators and auditors may access detailed audit logs.

---

## BR-AUDIT-005 Sensitive Data

Passwords, access tokens, secrets, and other sensitive credentials must never be stored in audit records.

---

# 13. Reporting Rules

## BR-REPORT-001 Authorized Reporting

Users may only view report data they are authorized to access.

Examples:

- Employees see their assigned assets.
- Department managers see departmental assets.
- Asset managers see organization-wide asset data.
- Auditors see approved audit information.

---

## BR-REPORT-002 Report Accuracy

Reports must use committed transactional data.

Draft or cancelled transactions must not be included unless explicitly requested.

---

## BR-REPORT-003 Export

Authorized users may export selected reports to CSV or PDF.

Exports must respect the same authorization rules as on-screen reports.

---

# 14. Data Retention Rules

## BR-DATA-001 Historical Retention

Historical asset transactions must be retained for audit and reporting purposes.

---

## BR-DATA-002 Soft Deletion

Business entities with historical transactions should use soft deletion or inactive status instead of physical deletion.

---

## BR-DATA-003 Personal Data

Personal data must be limited to what is required for asset assignment and accountability.

---

## BR-DATA-004 Retention Period

The precise retention period will be configurable in a future release.

The MVP will retain all synthetic portfolio data unless manually reset.

---

# 15. Cross-Cutting Validation Rules

## BR-VALIDATION-001 Required Fields

Required fields must be validated in both the frontend and backend.

Backend validation remains authoritative.

---

## BR-VALIDATION-002 Concurrency

The platform must prevent conflicting updates to the same asset.

Optimistic concurrency control should be considered.

---

## BR-VALIDATION-003 Transaction Integrity

Multi-step business operations must be processed transactionally where appropriate.

Example:

```text
Approve assignment
        ↓
Close pending request
        ↓
Create active assignment
        ↓
Update asset status
        ↓
Create audit event
```

---

## BR-VALIDATION-004 Date Rules

- Completion dates must not precede creation dates.
- Return dates must not precede assignment dates.
- Maintenance completion dates must not precede maintenance start dates.
- Purchase dates must not be in the future unless explicitly allowed.

---

# 16. Initial Rule-to-Module Mapping

| Rule Category     | Primary Module         |
| ----------------- | ---------------------- |
| User and access   | Identity and Access    |
| Asset master data | Asset Management       |
| Assignment        | Asset Lifecycle        |
| Transfer          | Asset Lifecycle        |
| Return            | Asset Lifecycle        |
| Maintenance       | Maintenance Management |
| Approval          | Workflow               |
| Notification      | Notification           |
| Audit             | Audit and Compliance   |
| Reporting         | Reporting              |

---

# 17. Rule Enforcement Strategy

Business rules should be enforced at the correct architectural layer.

| Rule Type              | Recommended Enforcement                          |
| ---------------------- | ------------------------------------------------ |
| Required field         | API validation and UI validation                 |
| Unique identifier      | Database constraint and application validation   |
| Authorization          | API authorization policy                         |
| Status transition      | Domain model or application service              |
| Multi-step transaction | Application service and database transaction     |
| Immutable history      | Application restriction and database permissions |
| Notification retry     | Asynchronous worker                              |
| Audit creation         | Application pipeline or domain event handler     |

---

# 18. Open Decisions

The following items require future architecture decisions:

- Fixed roles versus configurable permissions
- Approval workflow configuration model
- Asset value threshold for approval
- Audit data retention period
- Optimistic concurrency approach
- Notification retry policy
- PDF generation technology
- Soft-delete implementation standard

These decisions should be recorded as Architecture Decision Records.

---

# 19. Acceptance Criteria

This document is complete when:

- Core business rules are documented.
- Each rule has a unique identifier.
- Rules align with the PRD.
- Critical asset status transitions are defined.
- Approval and separation-of-duty rules are covered.
- Audit and security-related rules are included.
- Open architectural decisions are identified.

---

# 20. Next Deliverable

```text
docs/03-Domain-Model.md
```
