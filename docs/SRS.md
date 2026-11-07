# Smart College Management System — Software Requirements Specification

**Document status:** Phase 01 — Requirements analysis  
**Version:** 1.0  
**Date:** 2026-11-07  
**Repository:** [Teja453/College-Management-System](https://github.com/Teja453/College-Management-System)

## 1. Purpose and project overview

This Software Requirements Specification (SRS) defines the MVP requirements for the Smart College Management System (SCMS), a centralized web application for one college. It is intended for stakeholders, developers, testers, and deployment maintainers. It establishes expected system behavior before implementation; it is not an implementation design or a commitment to optional future capabilities.

The system will provide administrators, faculty, and students with role-appropriate access to academic and administrative workflows. The initial solution is a modular web application with a React/Bootstrap frontend, Django REST Framework backend, and MySQL database. The architecture is modular, but multi-tenant operation is outside the MVP.

## 2. Problem statement

College information and processes may be distributed across spreadsheets, paper records, and disconnected tools. This can make student records, academic schedules, attendance, examination outcomes, fees, and communications difficult to maintain consistently and access securely. SCMS will provide a centralized system of record with role-scoped access, validation, traceable administrative actions, and a consistent view of academic information.

## 3. Business objectives and success measures

- **BO-001:** Centralize student, faculty, department, course, subject, and academic-period records.
- **BO-002:** Provide controlled, role-appropriate access to information and administrative workflows.
- **BO-003:** Make attendance, examination, result, timetable, and fee information available through consistent workflows.
- **BO-004:** Support configurable academic policies rather than embedding thresholds and grading rules in application code.
- **BO-005:** Improve traceability for sensitive actions, including result publication and payment recording.
- **BO-006:** Provide a maintainable foundation that can be extended without introducing multi-institution complexity in the MVP.

**Initial success indicators:** authorized users can complete the principal workflows; access-control tests demonstrate that users cannot access out-of-scope records; academic rules can be administered and changes are traceable; published results and payment receipts can be retrieved; supported workflows have automated tests and documented operating instructions. Numerical service-level targets require stakeholder agreement before deployment.

## 4. Scope

### 4.1 MVP scope

The MVP includes authentication and role-based access; student and faculty records; departments, courses/programs, subjects, academic years and semesters; enrollment; faculty assignment; attendance; examinations, marks, configurable grading and result approval/publication; timetable; fee structures, manually recorded payments and receipts; in-app announcements and notifications; administrative reports; audit records for important actions; responsive role-based dashboards; and API and operational documentation.

### 4.2 Excluded from MVP

Multi-tenant/multi-college operation, online payment gateway processing, email/SMS delivery, native mobile applications, biometric attendance devices, automated integrations with external student information systems, advanced predictive analytics, and institution-wide single sign-on are excluded unless separately approved.

### 4.3 Future enhancements

Potential future work includes multi-institution support, payment-provider integration, email/SMS notifications, mobile applications, external identity and learning-management integrations, richer analytics, configurable approval chains, and automated timetable optimization. These are not MVP acceptance requirements.

## 5. Stakeholders, actors, and permissions

### 5.1 Actors

- **Administrator:** trusted college operator who manages users, academic structure, policies, schedules, fees, announcements, reports, and approved administrative workflows.
- **Faculty member:** instructor who can view assigned academic groups and students, record attendance, enter or submit marks, and review relevant reports.
- **Student:** authenticated learner who can view their own profile and published academic/financial information, timetable, and notifications.
- **System operator:** deploys, monitors, backs up, and maintains the application; does not receive ordinary academic access solely by virtue of operating infrastructure.

### 5.2 Authorization requirements

- **FR-001:** The system shall authenticate users and enforce authorization on every protected backend operation.
- **FR-002:** The system shall support administrator, faculty, and student roles. A user may not gain privileges by changing frontend state or request data.
- **FR-003:** Administrators shall access administrative functions; faculty shall access only assigned classes, subjects, and students; students shall access only their own permitted records and information published to them.
- **FR-004:** The system shall deny access by default where a user has no explicit permission and shall return a safe error response without disclosing protected data.
- **FR-005:** The system shall record the actor and timestamp for designated sensitive actions. Administrative role changes and result approvals/publications must be auditable.

A role/permission matrix and object-level scope checks will be maintained during design. The exact division of administrator capabilities (for example, whether one administrator can approve another administrator's configuration changes) is a stakeholder decision if separation of duties is required.

## 6. Functional requirements and module specifications

Requirements use stable identifiers. Unless a module explicitly allows public access, the actor must be authenticated and pass role and record-scope authorization. Outputs must not expose secrets or unrelated personal data.

### 6.1 Authentication and user management

**Requirements**
- **FR-010:** The system shall provide secure sign-in and sign-out and issue/refresh/revoke token-based sessions according to the selected JWT configuration.
- **FR-011:** Administrators shall create, update, activate, and deactivate user accounts and assign approved roles.
- **FR-012:** The system shall associate student and faculty profile records with user accounts without duplicating authentication credentials.
- **FR-013:** The system shall validate account identifiers, required profile fields, role assignment, and unique login identifiers.

**Workflow:** Administrator provisions an account and role; the user authenticates; the application returns only the functions and records allowed for that role. A deactivated account cannot obtain a new authorized session.

**Alternative/failure cases:** invalid credentials, expired or revoked token, inactive user, duplicate account identifier, unsupported role, and unauthorized account-management attempt must fail safely and be logged where appropriate. Password recovery and self-service registration require policy confirmation and are not assumed as MVP features.

### 6.2 Student management

**Actors:** administrator; student for self-view.

**Preconditions:** authorized account; applicable department/program and academic period exist for linked academic records.

**Main workflow:** administrator creates or updates a student profile and links it to a student account; enrollment associates the student with an approved program/period; student views their own profile and authorized academic data.

**Alternative workflows:** administrator corrects a profile, deactivates/archives a student, or reviews enrollment history. A student requests a correction outside the system if self-editing is not authorized.

**Validation:** required institutional identifier is unique; referenced academic entities must exist and be active; dates and status transitions must be valid; users cannot change another student's record.

**Outputs:** student profile, authorized roster/search results, enrollment history, and confirmation or validation errors.

**Failures:** duplicate identifier, invalid program/period, missing required data, unauthorized access, or conflicting enrollment must not partially persist changes.

### 6.3 Faculty management

**Actors:** administrator; faculty member for self-view and assigned work.

**Preconditions:** faculty account and department exist.

**Main workflow:** administrator creates/updates a faculty profile and department association, then assigns faculty to applicable subjects/classes; faculty views their assignments and scoped student lists.

**Alternative workflows:** assignment changes or faculty deactivation, retaining historical academic records.

**Validation:** institutional identifier and account link are unique; assignments reference active academic entities and valid periods; an inactive faculty member cannot receive new assignments.

**Outputs:** faculty profile, assignment list, and assigned-class rosters.

**Failures:** duplicate identity, invalid assignment, overlapping/conflicting assignment when prohibited by policy, or request outside the faculty member's scope.

### 6.4 Department and academic structure management

**Actors:** administrator; faculty/student read access only where needed.

**Preconditions:** administrator is authorized.

**Main workflow:** administrator maintains departments, programs/courses, subjects, academic years, semesters, and their relationships; records are used by enrollment, assignments, timetable, attendance, examinations, and fees.

**Alternative workflows:** deactivate or archive structures already referenced by historical records rather than deleting them.

**Validation:** codes/names follow agreed uniqueness scope; parent-child relationships are valid; academic dates are ordered and periods do not overlap where overlap is disallowed; referenced records cannot be destructively deleted.

**Outputs:** academic structure lists and detail records for downstream workflows.

**Failures:** duplicate code, invalid date range, invalid relationship, or destructive operation on referenced data is rejected with a clear validation response.

### 6.5 Academic years, semesters, courses, and subject assignment

**Actors:** administrator; faculty for assigned subject views.

**Preconditions:** department/program/subject records exist.

**Main workflow:** administrator configures academic periods and offerings, assigns subjects to programs/semesters, and assigns faculty to teach offerings.

**Alternative workflows:** amend an offering before use or archive it after its period ends. Historical offerings remain readable.

**Validation:** offering must reference valid subject/program/period; duplicate offering and invalid period combinations are rejected; faculty assignment must be active and scoped to the offering.

**Outputs:** offered-subject catalog, period configuration, and faculty assignments.

**Failures:** missing prerequisite structure, duplicate offering, invalid period, or assignment outside the user's authority.

### 6.6 Enrollment

**Actors:** administrator; student read-only self-view.

**Preconditions:** active student, program/period, and eligible subject offerings exist.

**Main workflow:** administrator enrolls a student in the relevant program/period and subjects; system validates eligibility and prevents duplicates; student views current and historical enrollments.

**Alternative workflows:** authorized correction, withdrawal, or transfer according to college policy while preserving history.

**Validation:** unique enrollment for the defined student/program/period combination; subject offerings must be open and compatible; enrollment state transitions must be permitted.

**Outputs:** enrollment confirmation and enrollment/roster data.

**Failures:** duplicate or incompatible enrollment, inactive record, missing subject offering, or unauthorized modification.

### 6.7 Attendance management

**Actors:** faculty assigned to the class; administrator; student for own attendance view.

**Preconditions:** active offering, faculty assignment, enrolled students, and a class session exist.

**Main workflow:** faculty creates or opens a session, records attendance for eligible enrolled students, validates and finalizes the roll call; students view their own attendance and resulting percentage; administrators configure attendance policy and review reports.

**Alternative workflows:** authorized correction after finalization with an audit record; cancellation of a session; reopening only under an explicitly authorized process.

**Validation and business rules:** attendance is unique per eligible student/session; only assigned faculty or authorized administrators may edit; session date and offering must match; finalized sessions become read-only except through the correction workflow. Attendance percentage is calculated from eligible sessions only. Proposed MVP eligibility rule: a session counts only after it has been held and its roll call finalized; future and cancelled sessions do not count. The initial configurable threshold is **75%**. The handling of present, absent, late, excused, and other statuses must be configurable or approved before implementation; the default status-weighting policy is an open decision.

**Outputs:** session roster, saved attendance, per-student percentage, threshold warning/alert, and reports.

**Failures:** unassigned faculty, non-enrolled student, duplicate entry, unfinalized/cancelled session in calculation, invalid status, or unauthorized correction. The system shall not silently treat missing marks as absence unless that policy is explicitly approved.

### 6.8 Examinations, marks, and results

**Actors:** administrator, assigned faculty, authorized reviewer/approver, student for published results.

**Preconditions:** offering and academic period exist; applicable grading and passing rules are configured; students are enrolled.

**Main workflow:** administrator creates an examination and maximum marks; authorized faculty enters marks for eligible students and submits a result set for review; reviewer records review outcome and requests corrections or recommends approval; an authorized administrator approves and publishes; students can view only published results.

**Workflow states:** Draft → Review → Admin-approved publication. A review rejection returns the result to draft/correction state. Published results are immutable through ordinary editing; an authorized correction/republication process must preserve prior value, reason, actor, and timestamp.

**Validation:** marks are within the assessment maximum; each eligible student/exam has at most one current result entry; grading bands are valid and non-overlapping; minimum passing marks and grade assignment use the rule version applicable to the result; draft/review results are not student-visible; approver must hold required permission.

**Outputs:** draft marks, review queue, review feedback, approval/publication record, grade/result, and student-visible published result.

**Failures:** missing configuration, invalid marks, duplicate entries, unauthorized faculty assignment, attempt to view unpublished data, unauthorized approval, or invalid state transition. Publication must be atomic so students do not see a partially published result set.

**Open policy decision:** whether reviewers and final approvers must be different users, and whether approvals are per exam, class, or result set.

### 6.9 Configurable academic rules

**Actors:** authorized administrator; relevant services use the active rule version.

**Preconditions:** administrator permission; target academic scope exists.

**Main workflow:** administrator creates or updates attendance threshold/status policy, grading bands, and minimum passing marks; system validates and versions the configuration; downstream calculations select a well-defined applicable version.

**Alternative workflow:** schedule a new rule to apply from a future academic period. Previously published outcomes retain their applied rule/version.

**Validation:** thresholds are within permitted bounds; grade ranges are non-overlapping and collectively valid per the chosen scale; pass marks are compatible with maximum marks/assessment scale; each scope has an unambiguous active configuration.

**Outputs:** rule configuration, validation feedback, and audit history.

**Failures:** overlapping bands, invalid threshold, conflicting active configuration, or attempted retroactive change without an explicit authorized correction process.

### 6.10 Timetable management

**Actors:** administrator; faculty and students read their permitted schedules.

**Preconditions:** periods, offerings, faculty assignments, and any required room records exist.

**Main workflow:** administrator creates a scheduled class with subject, group, faculty, room, date/time, and period; system detects prohibited conflicts; affected users view their timetable.

**Alternative workflows:** authorized reschedule/cancel with updated schedule and relevant in-app notification.

**Validation:** start precedes end; schedule falls within the academic period; no disallowed room, faculty, or student-group overlap; all references are active and compatible.

**Outputs:** timetable entries by student group, faculty, or period; conflict details on rejection.

**Failures:** missing assignment, conflicting schedule, invalid time/date, or unauthorized schedule access/change.

### 6.11 Fee management and manual payments

**Actors:** administrator/authorized finance operator; student for own charges and payment history.

**Preconditions:** student and applicable fee structure/charge exist; payment recorder has permission.

**Main workflow:** administrator configures fee structures and creates/applies student charges; an authorized operator records a Cash, UPI, or Bank Transfer payment with amount, date, and method details; the system updates the derived outstanding balance and creates a receipt; student views charges, balance, and payment history.

**Alternative workflows:** partial payment leaves a pending balance; overpayment is rejected unless a future approved credit/refund policy exists; erroneous payments are reversed or corrected through an audited adjustment rather than deletion.

**Validation:** amount is positive and currency/precision is valid; payment method is one of the supported methods; charge/student association is valid; recorded amount cannot exceed outstanding balance under the MVP assumption; reference fields follow method-specific rules defined during design. Receipt identifiers are unique and payment records are immutable except through audited correction.

**Outputs:** payment confirmation, receipt, transaction history, charge summary, and pending balance.

**Failures:** unauthorized payment recording, duplicate receipt/reference where uniqueness applies, invalid amount/method, stale balance/concurrent payment conflict, or receipt generation failure. A failed operation must not report a successful payment without a persisted transaction.

**Open business decisions:** currency, taxes/discounts/waivers, charge due dates and late fees, refund/overpayment handling, receipt numbering format, and whether a charge can be paid by multiple transactions. The design supports partial payment; values and policies require college confirmation.

### 6.12 Announcements and in-app notifications

**Actors:** administrator/authorized publisher; recipient users.

**Preconditions:** publisher is authorized; target audience is defined.

**Main workflow:** publisher creates and publishes an announcement or an application event produces a notification for its intended recipient(s); recipients view notifications and mark them read.

**MVP categories:** examination announcements, attendance alerts, fee reminders, result publication, and college announcements.

**Alternative workflows:** announcement is scheduled, edited before publication, or withdrawn by an authorized user; notification remains traceable according to retention policy.

**Validation:** content is non-empty and length-limited; target audience is valid; users may change read state only on their own notifications; sensitive details are not broadcast to an unauthorized audience.

**Outputs:** announcement, recipient notification list, unread count, and read status.

**Failures:** invalid audience, unauthorized publication, duplicate event processing, or delivery/record creation failure. Email and SMS are not MVP channels.

### 6.13 Reports and dashboards

**Actors:** administrator; faculty and students see role-limited views.

**Preconditions:** source records exist and user has access to the relevant scope.

**Main workflow:** dashboard/report query applies the user's role and data scope, calculates from authoritative records, and presents summaries and detail links.

**MVP outputs:** administrative summaries for students, faculty, enrollment, attendance, examinations/results, and fees; faculty summaries for assigned offerings; student summary of own attendance, published results, timetable, and fees.

**Validation:** report scope is authorized; unpublished results and other students' personal/financial data are excluded; calculations use the same rules as underlying modules.

**Failures:** invalid filters, out-of-scope query, or unavailable source data returns a safe error/empty state and does not widen access.

### 6.14 Audit records

**Actors:** system-generated; administrators and operators only as authorized for review.

**Preconditions:** designated action is performed by an authenticated actor or trusted system process.

**Main workflow:** system records actor, action, target, time, and relevant before/after or event metadata for sensitive operations; authorized users review audit history.

**MVP coverage:** user/role administration, academic-rule changes, finalized attendance corrections, result review/approval/publication/correction, fee/payment recording and adjustments, and administrative structure changes.

**Validation:** audit records must not expose passwords, tokens, or unnecessary sensitive data; ordinary users cannot alter or delete audit history.

**Failures:** critical action should not be represented as complete when its required audit record cannot be persisted; operational handling and availability policy require design-stage confirmation.

## 7. User workflows

1. **Provisioning:** administrator creates accounts and linked student/faculty profiles, configures academic structure, and assigns roles.
2. **Academic setup:** administrator defines year/semester, program/subject offerings, faculty assignment, and applicable rule versions.
3. **Enrollment:** administrator enrolls eligible students; faculty rosters derive from active assignments and enrollments.
4. **Teaching and attendance:** faculty conducts a scheduled class, records and finalizes the roll call; eligible sessions feed attendance percentage and alerts.
5. **Examination and publication:** administrator sets up exam; faculty enters marks; review occurs; an authorized administrator approves and publishes; students receive in-app notification and view published results.
6. **Fees:** administrator applies charges; authorized operator records one or more supported manual payments; system produces receipts and shows any remaining balance.
7. **Communications:** authorized publisher sends a college/audience announcement; event-driven notifications alert intended users; users view and acknowledge notifications.

## 8. Input and output specifications

All interfaces use validated structured inputs and explicit success/error responses. Exact API schemas and field-level limits are design deliverables in the API specification.

**Common inputs:** authenticated identity/token, record identifiers, required text fields, dates/times, academic period, status, and supported filters. IDs supplied by clients must be reauthorized against the authenticated user; IDs are not proof of access.

**Domain inputs:** profile and institutional identifiers; academic structure relationships; enrollment references; attendance session/status entries; exam maximum marks and student marks; timetable interval/room/group; fee charge and payment amount/method/date/reference; announcement content and target audience; rule thresholds, grade ranges, and effective period.

**Common outputs:** created/updated record or authorized list/detail, validation errors associated with fields where possible, state-transition result, report values, and safe authorization/not-found errors. Payment operations additionally return a receipt reference; result operations return workflow state and published result only when authorized. Response formats, pagination, sorting, and export formats will be defined during API design.

## 9. Non-functional requirements

- **NFR-001 Security:** use secure password hashing, protected transport in deployment, safe secret management, validated inputs, and backend-enforced role/object permissions.
- **NFR-002 Privacy:** minimize personal data returned; students and faculty receive only authorized records; protect educational and financial information from cross-user access.
- **NFR-003 Integrity:** use database transactions and constraints for related operations such as enrollment, attendance finalization, result publication, and payment recording.
- **NFR-004 Auditability:** retain trustworthy records of designated sensitive actions and rule versions applied to published outcomes.
- **NFR-005 Maintainability:** separate domain responsibilities, API representation, authorization, and business rules; keep configuration and secrets out of source control.
- **NFR-006 Usability/accessibility:** provide clear validation and error messages, keyboard-operable primary interactions, readable contrast, and responsive Bootstrap layouts for desktop and mobile browsers.
- **NFR-007 Performance:** common authenticated views should be responsive under the agreed expected college data volume. Specific latency, concurrency, and dataset targets must be established before performance acceptance.
- **NFR-008 Availability/recovery:** production backup, restoration, and operational monitoring procedures must be documented. Uptime and recovery-point/recovery-time targets require stakeholder agreement.
- **NFR-009 Compatibility:** support current stable versions of major modern browsers agreed at release; exact support matrix is a deployment decision.
- **NFR-010 Testability:** critical business rules, authorization boundaries, state transitions, and frontend workflows must have automated tests.
- **NFR-011 Data consistency:** academic calculations and reports must use the authoritative records and applicable effective rule version, not client-side-only calculations.
- **NFR-012 Configuration:** credentials, signing keys, database connection values, and deployment-specific settings must be provided through protected environment configuration, never committed as real secrets.

## 10. Business rules

- **BR-001:** MVP supports one college. Department/program records are organizational structures, not tenants.
- **BR-002:** The initial attendance threshold is 75%, stored as configurable data and scoped/versioned so policy changes are traceable.
- **BR-003:** Attendance percentage uses eligible sessions only. Proposed eligibility excludes future and cancelled sessions and requires held/finalized roll calls; status weighting requires confirmation.
- **BR-004:** Grading ranges and minimum passing marks are configurable data, validated for coherent, non-overlapping scales.
- **BR-005:** Results cannot be student-visible until approved and published by an authorized administrator.
- **BR-006:** Published results retain the academic rules used to calculate them; corrections require an authorized auditable process.
- **BR-007:** Payments are manually recorded in MVP and support Cash, UPI, and Bank Transfer; partial payments leave a derived pending balance.
- **BR-008:** Payment gateway transactions, email, and SMS are not MVP capabilities.
- **BR-009:** Users access only records permitted by role and record scope; UI hiding alone is insufficient authorization.
- **BR-010:** Historical records referenced by dependent records are archived/deactivated rather than destructively deleted.
- **BR-011:** The system shall not infer absence from a missing attendance entry unless an explicit approved policy defines that behavior.

## 11. System constraints and dependencies

- Backend: Python, Django, Django REST Framework, Django ORM, JWT authentication.
- Frontend: React, JavaScript, HTML5/CSS3, Bootstrap, Axios, and React Router.
- Database: MySQL and Django migrations.
- Testing: Django TestCase and/or pytest; React Testing Library and Vitest; API checks may use Postman.
- Documentation: Markdown and OpenAPI/Swagger.
- Version control and delivery: Git/GitHub, feature branches, code review, and environment-specific configuration.
- The application requires network access between browser, API, and database services. Production hosting, TLS termination, file storage, backups, and monitoring provider are not yet selected.
- No payment provider, email gateway, SMS provider, or external institutional system is required for MVP.

## 12. Assumptions and open decisions

### Assumptions used for this SRS

- The college supplies authoritative student/faculty identifiers and academic structure data.
- Administrators are responsible for provisioning accounts and maintaining approved academic configuration.
- Attendance sessions are associated with an offering and scheduled/held status.
- Financial balances are derived from charges and persisted payment transactions, not manually editable totals.
- In-app notifications are available to authenticated intended recipients.

### Decisions to confirm before the relevant implementation phase

1. Whether attendance statuses include late/excused and how each affects the numerator/denominator; whether missing attendance remains unmarked or becomes absent at finalization.
2. Currency, fee due dates, discounts/waivers, late fees, refund and overpayment policy, and receipt format.
3. Required student/faculty profile fields, identifier format, and account recovery/registration policy.
4. Reviewer/approver separation and result approval scope (per exam, class, or complete result set).
5. Timetable granularity, room resource rules, time zone, holidays, and recurrence requirements.
6. Data retention, archival, privacy/consent, and audit retention policy.
7. Expected users, data volume, performance targets, uptime, backup, and recovery targets.
8. Browser support matrix and whether reports need export formats in MVP.

These open decisions do not change the stated scope; they need resolution before implementation of affected details rather than being silently invented.

## 13. Acceptance checklist — requirements phase

- [x] Single-college MVP scope and multi-tenant exclusion documented.
- [x] Actors and high-level role boundaries documented; backend authorization required.
- [x] Core modules, their actors, preconditions, workflows, alternatives, validation, outputs, and failures specified.
- [x] Attendance threshold default (75%) and eligible-session calculation requirement documented; unresolved status policy called out.
- [x] Configurable grading ranges and minimum passing marks documented.
- [x] Draft, review, admin approval, and publication result workflow documented.
- [x] Manual Cash, UPI, and Bank Transfer, partial payments, balances, history, and receipts documented.
- [x] Required in-app notification categories documented; email/SMS excluded from MVP.
- [x] Functional and non-functional requirements use unique identifiers.
- [x] Input/output categories, constraints, assumptions, dependencies, and future enhancements documented.
- [x] No application code generated in Phase 01.
- [ ] Stakeholders resolve open business and deployment decisions before the affected design/implementation work.
- [ ] Stakeholder approves this SRS as the baseline for Phase 02 — System architecture.

---

**Phase 01 exit gate:** Requirements analysis is ready for stakeholder review. Phase 02 should begin only after this SRS is approved; subsequent changes should be recorded as requirement changes rather than silently altering implementation assumptions.
