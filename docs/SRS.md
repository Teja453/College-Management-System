# Smart College Management System — Software Requirements Specification

**Document status:** Phase 01 — Requirements analysis; subprompt 1.2 — Use cases  
**Version:** 1.1  
**Date:** 2026-11-07  
**Repository:** [Teja453/College-Management-System](https://github.com/Teja453/College-Management-System)

## 1. Purpose and project overview

This SRS defines the MVP requirements and use cases for the Smart College Management System (SCMS), a centralized application for a single college. It is intended for stakeholders, developers, testers, and operations staff. The MVP uses a React/Bootstrap frontend, Django REST Framework backend, and MySQL database. The architecture is modular, but multi-tenant operation is not in scope.

## 2. Problem statement and objectives

Academic and administrative information maintained across disconnected tools can be inconsistent, difficult to access, and hard to audit. SCMS centralizes academic records and workflows with role-scoped access and configurable academic rules.

- **BO-001:** Centralize student, faculty, department, course, subject, and academic-period records.
- **BO-002:** Provide controlled, role-appropriate access to administrative and academic information.
- **BO-003:** Digitize enrollment, attendance, examinations/results, timetables, fees, and communications.
- **BO-004:** Make academic policies configurable and traceable rather than hardcoded.
- **BO-005:** Audit sensitive actions such as rule changes, result publication, and payment recording.
- **BO-006:** Establish a maintainable foundation that can be extended without adding multi-institution complexity to the MVP.

Success means users can complete the principal workflows, backend tests demonstrate record-scope enforcement, configured rules are used consistently, published results and receipts are retrievable, and operational instructions are documented. Specific capacity, latency, and availability targets require stakeholder agreement before deployment.

## 3. Scope

### 3.1 MVP

Authentication and role-based access; student and faculty records; departments, programs/courses, subjects, academic years and semesters; enrollment and faculty assignment; attendance; examinations, marks, configurable grading and result approval/publication; timetable; fee structures, manually recorded payments and receipts; in-app announcements and notifications; dashboards, reports, and audit records; responsive UI and API/operational documentation.

### 3.2 Excluded from MVP

Multi-tenant/multi-college operation, payment gateway processing, email/SMS delivery, native mobile applications, biometric attendance devices, external SIS/LMS integrations, advanced predictive analytics, and institution-wide SSO.

### 3.3 Future enhancements

Potential future work includes multi-institution support, payment-provider integration, email/SMS, mobile apps, external identity and learning-management integrations, richer analytics, configurable approval chains, and timetable optimization. These are not MVP acceptance requirements.

## 4. Actors and authorization

- **Administrator (Admin):** manages accounts, academic structure, policies, schedules, fees, announcements, reports, and authorized approvals.
- **Faculty:** views assigned offerings and rosters, records attendance, enters/submits marks, and views assigned-class reports.
- **Student:** views their own profile, attendance, published results, timetable, fees/payment history, and notifications.
- **System operator:** operates infrastructure and backups; infrastructure access alone does not grant ordinary academic access.

- **FR-001:** Authenticate users and enforce authorization on every protected backend operation.
- **FR-002:** Support Admin, Faculty, and Student roles; frontend state or request data cannot grant privileges.
- **FR-003:** Faculty access is limited to assigned academic groups and students; students access only their own permitted records and published information.
- **FR-004:** Deny access by default and return safe errors without disclosing protected data.
- **FR-005:** Audit designated sensitive actions, including role administration and result approval/publication.

## 5. Functional requirements by module

- **FR-010 Authentication and accounts:** Secure sign-in/out and JWT session lifecycle; Admin provisions, updates, activates/deactivates accounts and assigns approved roles; unique identifiers and required profile data are validated. Student/faculty profiles link to users without duplicating credentials.
- **FR-020 Student management:** Admin creates/updates and archives student records; students view their own profile. Institutional identifiers are unique; referenced academic entities must exist and be active; historical records are retained.
- **FR-030 Faculty management:** Admin creates/updates faculty profiles and assigns teaching responsibilities; faculty view their own profile, assignments, and scoped rosters. Identifiers are unique and inactive faculty receive no new assignments.
- **FR-040 Academic structure:** Admin manages departments, programs/courses, subjects, academic years, semesters, offerings, and faculty assignments. Codes and relationships are validated; referenced records are archived rather than destructively deleted.
- **FR-050 Enrollment:** Admin enrolls eligible students in programs/periods/offerings; duplicate or incompatible enrollment is rejected; students view their own current and historical enrollments.
- **FR-060 Attendance:** Assigned faculty records attendance for enrolled students in class sessions and finalizes roll calls; authorized corrections are audited. Attendance is calculated from eligible sessions only. Proposed eligibility: session held and roll call finalized; future/cancelled sessions excluded. Initial threshold is configurable with default **75%**. Missing entries are not silently treated as absence absent an approved policy.
- **FR-070 Examinations/results:** Admin creates examinations; assigned faculty enters and submits marks; authorized reviewer reviews; Admin approves and publishes. State is Draft → Review → Admin-approved publication. Students see published results only. Marks must not exceed maximum; publication is atomic; corrections preserve history and audit.
- **FR-080 Academic rules:** Admin configures versioned attendance policies, grading ranges, and minimum passing marks, scoped to the applicable academic period. Ranges must be coherent/non-overlapping. Previously published results retain their rule version.
- **FR-090 Timetable:** Admin creates/reschedules/cancels classes; backend detects prohibited room, faculty, or student-group conflicts. Students and faculty see only their schedules.
- **FR-100 Fees/payments:** Admin/authorized finance operator configures fees, applies charges, and manually records Cash, UPI, or Bank Transfer payments. Partial payments, pending balances, history, and receipts are supported. Balances derive from charges and transactions. Payment corrections are audited; no gateway in MVP.
- **FR-110 Notifications:** Provide in-app examination announcements, attendance alerts, fee reminders, result publication notices, and college announcements. Users access only notifications addressed to them. Email/SMS are excluded.
- **FR-120 Reports/dashboards:** Provide role-scoped summaries for administration, assigned faculty work, and each student's own records. Reports use authoritative records and exclude unpublished results and unrelated personal/financial data.
- **FR-130 Audit:** Record actor, target, time, and relevant event/change data for role changes, academic-rule changes, finalized attendance corrections, result workflow, payments/adjustments, and significant structure changes. Secrets are never recorded; ordinary users cannot alter audit history.

## 6. Business rules

- **BR-001:** The MVP serves one college; organizational departments/programs are not tenants.
- **BR-002:** Attendance default threshold is 75%, stored as configurable, versioned data.
- **BR-003:** Attendance percentage includes eligible sessions only; exact status weighting and missing-attendance behavior must be confirmed before implementation.
- **BR-004:** Grading ranges and minimum passing marks are configurable and validated.
- **BR-005:** Results are visible to students only after authorized approval and publication.
- **BR-006:** Published outcomes retain the applied rule version; corrections require an authorized auditable process.
- **BR-007:** Payments are manual in MVP, use Cash/UPI/Bank Transfer, and may be partial; pending balance is derived.
- **BR-008:** No payment gateway, email, or SMS is required for MVP.
- **BR-009:** Backend role and record-scope checks are mandatory; UI restrictions alone are insufficient.
- **BR-010:** Referenced historical data is archived/deactivated, not destructively deleted.
- **BR-011:** Do not infer absence from a missing attendance entry unless an approved policy specifies it.

## 7. Non-functional requirements

- **NFR-001 Security:** secure password hashing, protected transport in deployment, safe secret management, validated inputs, and backend role/object permissions.
- **NFR-002 Privacy:** minimize returned personal data and prevent cross-user access to educational and financial records.
- **NFR-003 Integrity:** use database constraints and transactions for enrollment, attendance finalization, publication, and payments.
- **NFR-004 Auditability:** retain trustworthy sensitive-action history and the rule versions used for published outcomes.
- **NFR-005 Maintainability:** modular domain boundaries; separate API, authorization, and business logic; keep secrets out of source control.
- **NFR-006 Usability/accessibility:** clear validation, keyboard-operable primary interactions, readable contrast, responsive Bootstrap UI.
- **NFR-007 Performance:** common views should be responsive for the agreed college data volume; numerical targets remain to be set.
- **NFR-008 Availability/recovery:** document monitoring, backup, and restoration; uptime and RPO/RTO targets require agreement.
- **NFR-009 Compatibility:** support an agreed set of current major browsers.
- **NFR-010 Testability:** automate tests for business rules, permissions, state transitions, and primary frontend workflows.
- **NFR-011 Consistency:** authoritative backend records and applicable rule versions drive calculations.
- **NFR-012 Configuration:** credentials, signing keys, and deployment settings come from protected environment configuration, never committed real secrets.

## 8. Constraints, assumptions, and dependencies

**Technology constraints:** Python, Django, Django REST Framework, Django ORM, JWT; React, JavaScript, Bootstrap, Axios, React Router; MySQL; Django migrations; Django TestCase and/or pytest; React Testing Library and Vitest; OpenAPI/Swagger; Markdown; Git/GitHub.

**Assumptions:** College supplies authoritative identifiers and academic structure. Admin provisions accounts and configuration. Attendance sessions are associated with offerings and held/cancelled status. Balances are derived from charges and persisted payment transactions. Notifications are in-app and delivered to authenticated intended recipients.

**Open decisions to settle before affected implementation:** attendance statuses and weighting; missing roll-call behavior; currency, due dates, discounts/waivers, late fees, refunds/overpayment, receipt numbering; profile fields and account recovery; reviewer/approver separation and result approval scope; timetable granularity, time zone, holidays and recurrence; retention/privacy policy; expected volume, latency, availability, backup/recovery targets; browser matrix and report exports.

## 9. Use-case specification

Use-case IDs are stable references. “API interaction” is not a use case requirement here; exact endpoint paths and schemas are specified during architecture/API design. All use cases assume backend authentication and object-level authorization unless explicitly stated. Any ID supplied by a client is revalidated against the authenticated actor. Exceptions must not disclose unrelated record existence.

### UC-001 — Sign in and access role dashboard

- **Actor:** Admin, Faculty, Student.
- **Description:** Authenticate and enter the role-appropriate application area.
- **Preconditions:** User account exists and is active; application is available.
- **Main success flow:** (1) Actor submits login identifier/password. (2) System validates credentials and account state. (3) System establishes the configured JWT session. (4) Client retrieves the authenticated profile/capabilities. (5) System presents the role-appropriate dashboard.
- **Alternative flow:** Access token expires; client attempts the configured refresh flow and continues if valid. Actor signs out; client clears session and the backend revokes/blacklists refresh credentials if supported.
- **Exception flow:** Invalid credentials produce a generic failure; inactive account, invalid/revoked token, unavailable auth service, or unrecognized role prevents access and returns a safe response.
- **Postconditions:** On success, an authenticated session exists with role-derived access; on failure, no usable session is established.

### UC-002 — Admin manages user accounts and profiles

- **Actor:** Admin; Student/Faculty may view own profile.
- **Description:** Provision, update, activate/deactivate accounts and maintain linked student/faculty profiles.
- **Preconditions:** Admin is authorized; required department/program references exist for linked profiles.
- **Main success flow:** (1) Admin opens account/profile management. (2) System returns authorized records. (3) Admin enters account, role, and profile data. (4) System validates unique identifiers, required fields, role and references. (5) System saves linked records and audits sensitive changes. (6) Admin receives confirmation; affected user can view their own profile.
- **Alternative flow:** Admin corrects a profile or deactivates/archives a user while retaining academic history; user views own profile without edit permission.
- **Exception flow:** Duplicate identifier, invalid role/reference, inactive parent entity, unauthorized record, or persistence/audit failure rejects the change without partial data.
- **Postconditions:** Valid changes are persisted and sensitive operations auditable; otherwise existing records remain unchanged.

### UC-003 — Admin manages academic structure and rules

- **Actor:** Admin; Faculty/Student read relevant structures.
- **Description:** Maintain departments, programs, subjects, academic years/semesters, offerings, and versioned attendance/grading/passing rules.
- **Preconditions:** Admin is authorized; referenced parent records exist.
- **Main success flow:** (1) Admin selects structure or rule scope/effective period. (2) System displays current data. (3) Admin creates/updates entity or rule. (4) System validates uniqueness, relationships, dates, thresholds, grading range overlap, and passing-mark compatibility. (5) System saves and versions/audits the change. (6) Applicable offerings and services use the active rule version.
- **Alternative flow:** Admin schedules a future effective rule or archives referenced structures; historical results retain their prior rule version.
- **Exception flow:** Duplicate code, invalid period, overlapping ranges, conflicting active rule, invalid threshold, destructive deletion of referenced data, or unauthorized change is rejected.
- **Postconditions:** Valid versioned structure/rule is available to downstream workflows; historical records remain consistent.

### UC-004 — Admin enrolls students and assigns faculty

- **Actor:** Admin; Faculty/Student view resulting assignments/enrollment.
- **Description:** Enroll eligible students in program/period/offerings and assign faculty to offerings.
- **Preconditions:** Active student/faculty, academic period, program, and offerings exist.
- **Main success flow:** (1) Admin selects student and academic context. (2) Admin selects valid subject offerings and/or faculty assignment. (3) System validates eligibility, active status, compatibility, and uniqueness. (4) System persists enrollment/assignment and audit data where required. (5) Student and faculty see only their resulting authorized records.
- **Alternative flow:** Admin withdraws/transfers a student or changes assignment while retaining history, subject to allowed state transitions.
- **Exception flow:** Duplicate/incompatible enrollment, invalid offering, inactive person, prohibited assignment conflict, or unauthorized scope rejects the operation.
- **Postconditions:** Valid enrollment/assignment is active and visible only to authorized actors; failed changes do not partially apply.

### UC-005 — Faculty views assignments and rosters

- **Actor:** Faculty.
- **Description:** View assigned offerings, class schedules, and enrolled students.
- **Preconditions:** Faculty is authenticated and has active assignments.
- **Main success flow:** (1) Faculty opens assignments. (2) System derives faculty identity from session. (3) System returns active assigned offerings and scoped rosters. (4) Faculty selects an offering and views its details.
- **Alternative flow:** No active assignments yields an empty state; changed assignments appear on a subsequent request.
- **Exception flow:** Inactive account, stale/unassigned offering, or attempted access to another faculty member's class is denied safely.
- **Postconditions:** Faculty has viewed only current permitted assignments and student data.

### UC-006 — Faculty records and finalizes attendance

- **Actor:** Assigned Faculty; Admin for authorized correction; Student views own record.
- **Description:** Record roll call for a class session and calculate attendance from eligible sessions.
- **Preconditions:** Offering, assignment, enrolled roster, and valid class session exist.
- **Main success flow:** (1) Faculty selects an assigned session. (2) System displays eligible enrolled roster and session state. (3) Faculty records allowed statuses. (4) System validates scope, status, date, and one record per student/session. (5) Faculty saves and finalizes roll call. (6) System calculates percentage using eligible sessions and applicable attendance policy. (7) Student views their own summary; threshold alert may be generated.
- **Alternative flow:** Authorized correction/reopen after finalization is audited; a cancelled session is excluded; unfinalized roll call is saved but does not count as eligible.
- **Exception flow:** Unassigned faculty, non-enrolled student, duplicate/invalid status, future/cancelled session, conflicting update, or unauthorized correction is rejected. Missing entries are not treated as absent unless policy says so.
- **Postconditions:** Finalized attendance is authoritative and auditable; calculated percentage reflects eligible sessions only; rejected changes do not alter the record.

### UC-007 — Admin creates an examination

- **Actor:** Admin; Faculty/Student consume relevant exam information.
- **Description:** Define examination details for an offering and applicable period.
- **Preconditions:** Offering, academic period, enrolled students, and applicable grading/passing rules exist.
- **Main success flow:** (1) Admin creates exam details and maximum marks. (2) System validates references, date/period, maximum marks, and rule applicability. (3) System saves the exam in an enterable state. (4) Assigned faculty can access the exam roster.
- **Alternative flow:** Admin edits an exam before marks are submitted; after use, changes follow an authorized audited correction process.
- **Exception flow:** Missing rules, invalid maximum, duplicate exam definition, incompatible period, or unauthorized request is rejected.
- **Postconditions:** Valid exam is available to assigned faculty and eligible roster; no student result is published by this use case.

### UC-008 — Faculty enters and submits marks for review

- **Actor:** Faculty assigned to the exam; authorized reviewer/Admin receives submission.
- **Description:** Enter eligible students' marks and submit a result set for review.
- **Preconditions:** Exam is open; faculty has assignment; students are eligible; configured grading and passing rules apply.
- **Main success flow:** (1) Faculty opens assigned exam roster. (2) System returns eligible students and maximum marks. (3) Faculty enters marks and saves draft. (4) System validates mark bounds, roster, and duplicate constraints. (5) Faculty submits the result set. (6) System changes state to Review and records submission.
- **Alternative flow:** Reviewer returns the set for correction; faculty edits permitted draft values and resubmits.
- **Exception flow:** Out-of-range marks, invalid student, incomplete required roster, closed/invalid state, or unassigned faculty rejects the operation. Draft/review results remain hidden from students.
- **Postconditions:** Valid set is in Review or returned correction state with audit trail; it is not student-visible before publication.

### UC-009 — Admin reviews, approves, and publishes results

- **Actor:** Authorized reviewer and Admin approver; Students receive published results.
- **Description:** Complete review and release a validated result set.
- **Preconditions:** Submitted result set is in Review; approval permission and applicable rule version exist.
- **Main success flow:** (1) Reviewer opens the review queue. (2) System displays scoped marks, validation, and rule version. (3) Reviewer records review outcome. (4) If accepted, authorized Admin approves. (5) Admin publishes the complete set. (6) System atomically marks results published, audits actor/time/rule version, and creates student notifications. (7) Students can retrieve their own published result.
- **Alternative flow:** Reviewer returns set for correction; Admin delays publication; authorized correction/republication preserves prior values and reasons.
- **Exception flow:** Invalid marks/state, unauthorized reviewer/approver, missing approval, or persistence failure prevents publication. Students never see a partially published set.
- **Postconditions:** On success, the full approved set is published and student-visible within scope; otherwise it remains non-published.

### UC-010 — Student views profile and enrollment

- **Actor:** Student; Admin maintains the records.
- **Description:** View own profile and current/historical enrollment.
- **Preconditions:** Student is authenticated and linked to a student profile.
- **Main success flow:** (1) Student opens profile/enrollment. (2) System derives student identity from the authenticated account. (3) System returns own authorized profile and enrollment history.
- **Alternative flow:** Missing current enrollment displays historical records or a clear empty state; profile correction is requested through the college's approved process if self-editing is not enabled.
- **Exception flow:** Missing profile mapping is reported safely for administrative resolution; an ID for another student is rejected without disclosing that record.
- **Postconditions:** Student has viewed only their own permitted information; no record is changed.

### UC-011 — Student views attendance

- **Actor:** Student; Faculty/Admin create or correct attendance.
- **Description:** View session attendance, percentage, and threshold status.
- **Preconditions:** Student is authenticated and has relevant enrollment/attendance records.
- **Main success flow:** (1) Student selects period/subject. (2) System derives student identity and applicable attendance rule. (3) System returns own eligible session records, percentage, and threshold status.
- **Alternative flow:** No eligible sessions yields an explicitly defined not-yet-available state; student may submit a discrepancy request if that feature is approved.
- **Exception flow:** Other-student identifier or out-of-scope period is denied; unavailable data returns safe error without fabricating a zero percentage.
- **Postconditions:** Student sees an accurate own-record summary; attendance remains unchanged.

### UC-012 — Student views published results

- **Actor:** Student; Admin publishes; Faculty submits marks.
- **Description:** View only own approved and published examination results.
- **Preconditions:** Student is authenticated; published results may or may not exist.
- **Main success flow:** (1) Student opens results. (2) System derives student scope and filters to published results. (3) System displays authorized results and applicable grade/outcome.
- **Alternative flow:** No published results displays an empty state; result publication generates an in-app notification.
- **Exception flow:** Draft/review or another student's result is denied/not found safely; unpublished/internal review details are not disclosed.
- **Postconditions:** Student has viewed only their published results; result data is unchanged.

### UC-013 — Admin manages timetable; Faculty/Student view schedules

- **Actor:** Admin creates/updates; Faculty and Student view own schedules.
- **Description:** Maintain class schedules and provide role-scoped timetable views.
- **Preconditions:** Academic period, offering, assignment, and required room/group records exist.
- **Main success flow:** (1) Admin enters offering, room, group, faculty, and time. (2) System validates interval, period, and prohibited conflicts. (3) System saves schedule. (4) Faculty/student requests own timetable; system derives scope from assignments/enrollment and returns authorized entries.
- **Alternative flow:** Admin reschedules/cancels and may trigger an in-app notification; users see updated schedule on refresh.
- **Exception flow:** Room/faculty/group conflict, invalid interval/reference, or cross-group schedule query is rejected safely.
- **Postconditions:** Valid schedule is current and visible only to affected authorized users; rejected changes do not replace the prior schedule.

### UC-014 — Admin records manual payment and generates receipt

- **Actor:** Admin/authorized finance operator; Student views own history and receipt.
- **Description:** Record Cash, UPI, or Bank Transfer payment against a charge, including partial payment.
- **Preconditions:** Student charge exists; recorder is authorized; balance is available.
- **Main success flow:** (1) Operator opens charge and verifies current balance. (2) Operator enters positive amount, date, supported method, and relevant reference. (3) System rechecks balance and validates method/amount. (4) System transactionally records payment and unique receipt reference. (5) System derives remaining balance and displays receipt. (6) Student views own payment history/receipt.
- **Alternative flow:** Partial payment leaves pending balance; further valid payments may be recorded. An error is corrected through audited reversal/adjustment, not deletion.
- **Exception flow:** Amount exceeds balance under current MVP assumption, invalid method/reference, duplicate receipt/reference, concurrent balance change, unauthorized actor, or receipt failure prevents false success and is handled safely.
- **Postconditions:** On success, payment, receipt, audit record, and derived balance are consistent; on failure, no payment is reported as successful. Gateway processing is not used.

### UC-015 — Student views fees, payments, and receipts

- **Actor:** Student; Admin/finance operator maintains financial records.
- **Description:** View own charges, paid totals, pending balance, payment history, and receipts.
- **Preconditions:** Student is authenticated and has financial records or no records.
- **Main success flow:** (1) Student opens fees. (2) System derives student identity. (3) System returns own charges and derived balance. (4) Student opens own payment history and retrieves an associated receipt.
- **Alternative flow:** No charges or payments displays a clear empty state; partial payments show the remaining balance.
- **Exception flow:** Another student's charge/receipt is denied; unavailable receipt or data service produces a safe error.
- **Postconditions:** Student has viewed only own financial information; no payment or charge is changed.

### UC-016 — Admin publishes announcement and users manage notifications

- **Actor:** Admin/authorized publisher; Admin, Faculty, Student as notification recipients.
- **Description:** Publish college/audience announcements and provide event-based in-app notifications.
- **Preconditions:** Publisher is authorized; audience is valid; recipient accounts are eligible.
- **Main success flow:** (1) Publisher drafts content and chooses audience/category. (2) System validates content and target scope. (3) Publisher publishes. (4) System records announcement and creates recipient notifications. (5) Recipient views their notifications and marks an item read.
- **Alternative flow:** Authorized publisher edits before publication, schedules or withdraws according to policy; events generate exam, attendance, fee, or result notifications.
- **Exception flow:** Invalid audience, empty/oversized content, unauthorized publication, duplicate event processing, or attempt to change another user's read state is rejected safely.
- **Postconditions:** Published content and notifications are visible to intended users only; per-user read state is saved. Email/SMS are not sent.

### UC-017 — Users view dashboards and authorized reports

- **Actor:** Admin, Faculty, Student.
- **Description:** View role-appropriate summaries calculated from authoritative records.
- **Preconditions:** Authenticated user; source data may exist or be empty.
- **Main success flow:** (1) User opens dashboard/report and selects permitted filters. (2) System validates filters and applies role/object scope before querying. (3) System calculates summaries using current authoritative records and applicable rules. (4) System presents summaries and authorized detail links.
- **Alternative flow:** No matching data returns a clear empty state; Admin sees institution-wide permitted reports, Faculty assigned-scope reports, Student own-record summaries.
- **Exception flow:** Invalid filter or out-of-scope query returns validation/authorization error; unavailable source data does not widen scope or produce fabricated values. Draft results and unrelated student/financial data are excluded.
- **Postconditions:** User has viewed only permitted summaries; no underlying record is altered.

### UC-018 — Authorized user reviews audit history

- **Actor:** Admin/authorized operator; system creates audit records.
- **Description:** Review trace records for designated sensitive actions.
- **Preconditions:** User has audit-read permission; events exist or history is empty.
- **Main success flow:** (1) Authorized user selects permitted filters. (2) System scopes and returns audit events with actor, target, timestamp, and allowed change metadata. (3) User reviews history.
- **Alternative flow:** Empty result displays an empty state; pagination/filtering narrows authorized events.
- **Exception flow:** Unauthorized users are denied; invalid filters return validation errors; secrets/tokens are redacted; attempts to alter/delete history are denied.
- **Postconditions:** Audit history remains unchanged and accessible only to authorized reviewers.

## 10. Input and output specifications

Inputs include authenticated session, record references, required profile and academic fields, date/time ranges, status values, marks/maximum marks, fee amounts/method/reference, rule thresholds/ranges, announcement content/audience, and permitted filters. All client-supplied identifiers are checked against the actor's scope. Exact schemas, field limits, pagination, sorting, and API error contract are architecture/API-design deliverables.

Outputs include authorized records/lists, state-transition results, validation errors, scoped summaries, in-app notifications, published results, payment receipt references, and safe authentication/authorization/not-found responses. No output contains credentials, tokens, unrelated personal data, or unpublished results for students.

## 11. Acceptance checklist — requirements phase

- [x] Single-college MVP scope and multi-tenant exclusion documented.
- [x] Admin, Faculty, and Student actors and backend authorization documented.
- [x] All core modules have requirements and related use cases.
- [x] Every use case includes ID, name, actor, description, preconditions, main flow, alternatives, exceptions, and postconditions.
- [x] Attendance default of 75%, eligible-session calculation, configurable grading/passing, and result publication workflow are covered.
- [x] Manual Cash/UPI/Bank Transfer payments, partial balances, history, and receipts are covered.
- [x] In-app notification categories and role-scoped reports/audit are covered.
- [x] MVP exclusions, future enhancements, assumptions, constraints, and open policy decisions are documented.
- [x] No application code is generated as part of requirements analysis.
- [ ] Stakeholders resolve open business/deployment decisions before affected implementation.
- [ ] Stakeholder approves this SRS baseline before Phase 02 architecture work.

---

**Phase 01 exit gate:** Requirements analysis is ready for stakeholder review. Phase 02 should begin only after approval; changes should be recorded as requirement changes rather than silently altering implementation assumptions.
