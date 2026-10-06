# Staffing - Permissions Catalog

This document defines all atomic permissions for the Staffing domain.

---

## Permissions

| Category                   | Permission ID                                  | Description                                         | Applicable Scopes            | Notes                                                                           |
| -------------------------- | ---------------------------------------------- | --------------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------- |
| **Employee Roster**        | `staffing.employee-roster.view`                | View employee roster for entity                     | Entity                       | Required to see list of employees for school/district                           |
| **Employee Roster**        | `staffing.employee-roster.add-employee`        | Add new employee to roster via online entry         | Entity                       | Triggers Unique ID matching workflow with Mi-Key                                |
| **Employee Roster**        | `staffing.employee-roster.update-employment`   | Update employment status, dates, separation reason  | Entity                       | Allows termination and status changes                                           |
| **Employee Roster**        | `staffing.employee-roster.update-demographics` | Request demographic update via Mi-Key               | Entity                       | Triggers Mi-Key update workflow                                                 |
| **Employee Roster**        | `staffing.employee-roster.certify`             | Certify employee roster collection for entity       | Entity                       | Requires all pre-certification tasks complete                                   |
| **Employee Roster**        | `staffing.employee-roster.bulk-upload`         | Upload bulk file to update employee data            | Entity                       | Must include valid Unique IDs; cannot create new employees via bulk             |
| **Position Roster**        | `staffing.position-roster.view`                | View position roster and organizational chart       | Entity                       | Required to see positions for school/district                                   |
| **Position Roster**        | `staffing.position-roster.create`              | Create new position in roster                       | Entity                       | Only allowed during working window or open collection                           |
| **Position Roster**        | `staffing.position-roster.update`              | Update position details, status, approved FTE       | Entity                       | Includes status changes (Filled, Frozen, Cancelled, etc.)                       |
| **Position Roster**        | `staffing.position-roster.delete`              | Delete position during working window               | Entity                       | Only allowed before position is established/certified                           |
| **Position Roster**        | `staffing.position-roster.certify`             | Certify position roster collection for entity       | Entity                       | Requires all pre-certification tasks complete                                   |
| **Assignment**             | `staffing.assignment.view`                     | View employee assignment details                    | Entity                       | Required to see assignments for employees in entity                             |
| **Assignment**             | `staffing.assignment.create`                   | Assign employee to position                         | Entity                       | Triggers credential validation                                                  |
| **Assignment**             | `staffing.assignment.update`                   | Update assignment details (FTE, dates, course info) | Entity                       | Includes adding/removing spots from position                                    |
| **Assignment**             | `staffing.assignment.submit-justification`     | Submit credential error justification               | Entity                       | Required when assigning without appropriate credential                          |
| **Assignment**             | `staffing.assignment.certify`                  | Certify assignment details collection for entity    | Entity                       | Requires all pre-certification tasks complete                                   |
| **Education History**      | `staffing.education-history.view`              | View education history for employee                 | Entity, Self-only            | District users see for their employees; citizens see own                        |
| **Education History**      | `staffing.education-history.update`            | Add/update education history records                | Entity, Self-only            | Citizens can add/remove; districts can add, cannot remove citizen-added         |
| **Evaluation Outcome**     | `staffing.evaluation-outcome.view`             | View evaluation outcomes for employee               | Entity, Self-only            | District users see for their employees; citizens see own                        |
| **Evaluation Outcome**     | `staffing.evaluation-outcome.submit`           | Submit evaluation outcome for employee              | Entity                       | District submits for instructional staff/administrators                         |
| **Evaluation Outcome**     | `staffing.evaluation-outcome.appeal`           | Appeal evaluation outcome                           | Entity, Self-only            | District can appeal within 5-year window; citizen can appeal any                |
| **Evaluation Outcome**     | `staffing.evaluation-outcome.approve-appeal`   | Approve or deny citizen appeal request              | Entity                       | District reviews and approves/denies citizen appeals                            |
| **New Teacher Monitoring** | `staffing.new-teacher-monitoring.view`         | View new teacher monitoring data                    | Entity                       | Required to see mentor assignments and SCECH tracking                           |
| **New Teacher Monitoring** | `staffing.new-teacher-monitoring.update`       | Update new teacher monitoring data                  | Entity                       | Assign mentor, link SCECH sessions for DPPD requirement                         |
| **Collection**             | `staffing.collection.view`                     | View collection workflow status and progress        | Entity                       | See pre-certification tasks, quality review results                             |
| **Collection**             | `staffing.collection.request-exception`        | Submit collection exception request                 | Entity                       | Request extension past legislative deadline                                     |
| **Collection**             | `staffing.collection.validate`                 | Trigger quality review validation                   | Entity                       | Run data quality checks before certification                                    |
| **Audit**                  | `staffing.audit.view`                          | View audit items for ISD/RESA or SOM                | ISD/RESA Entity, System-wide | ISD Auditor sees constituent districts; SOM Auditor sees all ISDs               |
| **Audit**                  | `staffing.audit.review-submission`             | Review district data collection submission          | ISD/RESA Entity              | ISD Auditor reviews constituent district submissions                            |
| **Audit**                  | `staffing.audit.submit-finding`                | Create and submit audit finding                     | ISD/RESA Entity              | Document placement issues, request additional documentation                     |
| **Audit**                  | `staffing.audit.request-documentation`         | Request additional documentation from district      | ISD/RESA Entity              | Request master schedules, attendance records, etc.                              |
| **Audit**                  | `staffing.audit.finalize`                      | Finalize and certify audit report                   | ISD/RESA Entity              | Requires attestation and e-signature                                            |
| **Audit**                  | `staffing.audit.decertify`                     | De-certify audit report during allowable window     | ISD/RESA Entity              | Allows correction during defined de-certification period                        |
| **Admin**                  | `staffing.admin.manage-collections`            | Define and manage collection configurations         | System-wide                  | Staffing Data Admin role; create collections, set date ranges, entity types     |
| **Admin**                  | `staffing.admin.manage-validations`            | Define and manage business rule validations         | System-wide                  | Staffing Data Admin role; assign validations to collections/categories/elements |
| **Admin**                  | `staffing.admin.manage-data-elements`          | Define and manage data element definitions          | System-wide                  | Staffing Data Admin role; create/update data elements, categories, components   |
| **Admin**                  | `staffing.admin.process-exceptions`            | Approve or deny collection exception requests       | System-wide                  | Staffing Data Admin role; review and approve deadline extension requests        |
| **Admin**                  | `staffing.admin.view-file-queues`              | View and manage file upload queues                  | System-wide                  | Staffing Data Admin role; monitor bulk upload processing                        |
| **Admin**                  | `staffing.admin.reset-file-processing`         | Reset or delete files in processing queue           | System-wide                  | Staffing Data Admin role; troubleshoot stuck file uploads                       |
| **Admin**                  | `staffing.admin.manage-reports`                | Define and manage report availability and access    | System-wide                  | Staffing Data Admin role; configure report access by role                       |

**Note:** Identity matching/resolution administration (New ID, Link ID, Split ID, Retire
ID, and review of identity requests) is not a Staffing capability — it is owned
entirely by `iam` (see `iam-permissions.md`'s Identity Administrator permissions:
`iam.identity-admin.view-pending-requests`, `.resolve-request`, `.update-person-record`,
`.manage-configuration`). Staffing only triggers identity resolution and consumes its
completion event; it defines no permissions for approving, denying, or administering
identity requests.
